# MySQL Administration

- [MySQL Administration](#mysql-administration)
  - [Privileges](#privileges)
  - [MySQL Components](#mysql-components)
    - [Passwords complexity](#passwords-complexity)
  - [(Status) Variables](#status-variables)
  - [Events](#events)
  - [General optimization topics](#general-optimization-topics)
    - [Dirty pages](#dirty-pages)
  - [Instance warmup](#instance-warmup)
  - [Replication](#replication)
    - [Logging/monitors](#loggingmonitors)
    - [Non-blocking schema changes](#non-blocking-schema-changes)
  - [Client/server](#clientserver)
    - [Find configuration files used](#find-configuration-files-used)
  - [Database metadata](#database-metadata)
  - [Convenient operations](#convenient-operations)
    - [Skip the indexes on mysqldump dumps](#skip-the-indexes-on-mysqldump-dumps)
  - [RDS-specific procedures](#rds-specific-procedures)
  - [Mydumper/myloader](#mydumpermyloader)

## Privileges

The user format is `name[@'host']`

```sql
-- `PASSOWRD EXPIRE` makes the user choose the password on first connection; it still allows to connect
-- interactively, but only to change the pwd.
-- In order to allow a user to change the pwd non-interactively, they must use the option `--connect-expired-password`.
--
CREATE USER @user IDENTIFIED BY @pwd PASSWORD EXPIRE;

-- Change user properties, including pwd.
-- Can use `USER()` instead of the name literal, to make an unprivileged user change their own pwd.
--
ALTER USER @user IDENTIFIED BY @pwd;

RENAME USER @old TO @new;

GRANT SELECT ON mydb.* TO @user;

SHOW GRANTS for @user\G

-- *************************** 1. row ***************************
-- Grants for user@%: GRANT SELECT, .. ON *.* TO `user`@`1.2.3.4` WITH GRANT -- OPTION
-- *************************** 2. row ***************************
-- Grants for user@%: GRANT APPLICATION_PASSWORD_ADMIN, ... ON *.* TO `user`@`1.2.3.4` WITH GRANT OPTION

-- Must specify host pattern, if the user has it.
--
DROP USER @user;
```

## MySQL Components

Use `SELECT * FROM mysql.component` to display them.

### Passwords complexity

Reference (see subsections): https://dev.mysql.com/doc/refman/en/validate-password.html.

```sql
INSTALL COMPONENT 'file://component_validate_password';

-- Other vars available as `validate_password.%`.
--
SET GLOBAL validate_password.length = 12;
```

## (Status) Variables

```sql
SHOW GLOBAL VARIABLES [LIKE 'pattern'| WHERE expr];
SELECT @@variable_name;                             -- alternate form for selecting a global var

SHOW GLOBAL STATUS [LIKE 'pattern' | WHERE expr];
```

## Events

```sql
-- The options set are the default.

CREATE EVENT my_event
ON SCHEDULE
EVERY 15 MINUTE
-- Optional; options: DISABLE | DISABLE ON SLAVE
ENABLE
-- Optional.
COMMENT 'My event comment'
DO
CALL mydb.my_stored_procedure;
```

## General optimization topics

### Dirty pages

Dirty pages percentage query:

```sql
SELECT
  ROUND(
    100 * (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME = "Innodb_buffer_pool_pages_dirty") /
    (
      1 +
      (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME = "Innodb_buffer_pool_pages_data") +
      (SELECT VARIABLE_VALUE FROM performance_schema.global_status WHERE VARIABLE_NAME = "Innodb_buffer_pool_pages_free")
    )
  , 2) `dirty_pct`
;
```

## Instance warmup

The settings `innodb_buffer_pool_dump_at_shutdown` and `innodb_buffer_pool_load_at_startup` should always be enabled.  
Also make sure to set `innodb_buffer_pool_dump_pct` to an optimal value: the default 25% may be too low, but higher value slow down shutdown/startup and infrequently used pages may be dumped.

In cases where they can't have a valid effect (e.g. new instance), then manual warmup should be applied. Useful queries are:

```sql
SELECT COUNT(*) FROM mytable USE INDEX();          # Force a full table scan
SELECT COUNT(*) FROM mytable FORCE INDEX(myindex); # Force reading an index, also needed!
```

## Replication

```sql
-- The `SLAVE` definiton has been deprecated.
--
SHOW REPLICA STATUS\G
```

Gather the replication status via SELECT:

```sql
-- `ON`/`OFF` (enum).
--
SELECT SERVICE_STATE FROM performance_schema.replication_applier_status;

-- Another replication metadata table is `performance_schema.replication_connection_status`.
```

### Logging/monitors

```sql
-- The general log includes a lot of stuff (including, all the queries).
--
SET GLOBAL log_output = 'FILE';
SET GLOBAL general_log_file = 'general.log'; -- with FILE output, this is mandatory
SET GLOBAL general_log = 'ON';

-- Log deadlocks in the error log.
--
SET GLOBAL innodb_print_all_deadlocks = ON;
```

Monitors affect performance, so they should be enabled only for debugging:

```sql
-- Enable periodic (every 15") general monitor output.
--
SET GLOBAL innodb_status_output = ON;

-- Enable lock monitor output in the InnoDB engine status, and the periodic output (the latter requires `innodb_status_output=ON`).
--
SET GLOBAL innodb_status_output_locks = ON;
```

### Non-blocking schema changes

Gh-ost (works also on tables with triggers).

This is the configuration for running it on the master (has more overhead than running from the slave); with `--execute`, changes are applied, and the old table is kept, without it, changes are not applied:

```sh
gh-ost \
  --user=$USER --password=$PWD --host=localhost \
  --database=$DB --table=comments \
  --alter="ENGINE=InnoDB" \
  --exact-rowcount --verbose \
  --allow-on-master \
  --execute \
;
```

`pt-online-schema-change` doesn’t work on tables with triggers; crashed on production:

```sh
pt-online-schema-change --execute --alter "MODIFY section VARCHAR(40) NOT NULL DEFAULT ''" D=db_name,t=table_name,u=root,p=root_pwd
```

## Client/server

### Find configuration files used

```sh
ls -l $(mysqld --verbose --help | awk "/Default options/ { getline; gsub(\"~\", \"$HOME\", \$0); print }") 2> /dev/null
```

## Database metadata

Space occupation of InnoDB indexes (data):

```sql
# Remember that the table data is the primary index!
#
SELECT index_name, stat_value*@@innodb_page_size `size`
FROM mysql.innodb_index_stats
WHERE stat_name = 'size'
     AND (database_name, table_name) = ('my_db', 'my_table')
ORDER BY index_name;
```

Find the foreign keys for a table/column:

```sql
SELECT TABLE_NAME, COLUMN_NAME, CONSTRAINT_NAME, REFERENCED_TABLE_NAME, REFERENCED_COLUMN_NAME
FROM information_schema.KEY_COLUMN_USAGE
WHERE (REFERENCED_TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME) = ('my_db', 'my_table', 'my_column');
```

## Convenient operations

### Skip the indexes on mysqldump dumps

Replace keys with null operations; valid alternatives:

- on `ALTER TABLE`
  - `ALGORITHM=DEFAULT`
  - `LOCK=DEFAULT`
  - `ENABLE KEYS`
  - `ADD CHECK (TRUE)` : adds a check
- on `CREATE TABLE`
  - `CHECK (TRUE)` : adds a check

`ALTER TABLE` key additions is produced by the Percona distro with `--innodb-optimize-keys`.

We can't just remove the lines, because if they're at the end of the table def, they'll leave the previous statement with a terminating comma.  
The leading comma could be deleted with some text processing trickery, but this solution is simpler.

```sh
mysqldump $db | perl -pe 's/^  (UNIQUE |FULLTEXT )?KEY.+?(,)?$/  CHECK(TRUE)$2'
```

## RDS-specific procedures

See [AWS](aws.md#rds).

## Mydumper/myloader

```sh
# Using chunked load is not worth. The dump requires more threads, since many queries will return no records.
#
# - account 004/226
#   - rows 100k
#     - dump: threads=16: 56",                 threads=4: 1'30"
#     - load: threads=16: 37", threads=8: 46", threads=4: 1'29"
#   - entire tables
#     - dump: threads=16: 56",                 threads=4: 46"
#     - load: threads=16: 43",                 threads=4: 1'25"
#
# --verbose $n   : output level; 3=info (each table operation is reported)
# --compress-protocol : enable MySQL protocol compression
# --lock-all-tables : important on RDS, since standard locking (FTWRL) doesn't work; locks only briefly,
#                  at the beginning (see https://answers.launchpad.net/mydumper/+question/236273)
# --database $d  : important, otherwise, also the MySQL schemas are dumped
# --nregex $r    : exclude by regex; '.*' prefix and suffix are implied; when using delimiters, remember
#                  that the match is performed against the full name (db_name.table)
#
# --triggers     : dump triggers (off by default)
#
# --outputdir $d : default=export-$timestamp
# --no-data
# --threads $n   : default=4
# --logfile $f   : default=stdout
# --rows $n      : set chunks to N rows; by default, tables are dumped in a single query
# --chunk-filesize $n : set the chunk size to N megabytes; seems not to work as intended
#
# On verbose=3, tt's important to check the exit status, because mydumper will continue for some errors (e.g. where
# condition causing an error), and at this level, the errors are not easy to spot.
#
mydumper \
  -h $HOST -u $USER -p $PWD \
  --verbose 3 \
  --compress-protocol \
  --lock-all-tables \
  --threads 16 \
  --database $SOURCE_DB \
  --nregex '^$SOURCE_DB\.(table_prefix|table_name$)' \
  --where 'tenant_id IN (64)' \
  || echo 'Errors found!'

# --source-db $src --database $dst : restore into a different db; WATCH OUT! $dst must exist, otherwise
#                                    nothing is raised
# --socket $f    : important
# --innodb-optimize-keys : yay!
# --overwrite-tables
#
# --threads $n   : default=4
#
# There are a few issues; see:
#
# - https://github.com/maxbube/mydumper/issues/468
# - https://github.com/maxbube/mydumper/issues/469
#
# Also, as of v0.11.2, on a test db, an error exit status is returned, even if no errors are displayed.
#
myloader \
  -u $USER --socket /tmp/mysql.sock \
  --directory export-* \
  --verbose 2 \
  --source-db $SOURCE_DB --database $DEST_DB \
  --innodb-optimize-keys \
  --threads 16 \
  --overwrite-tables
```
