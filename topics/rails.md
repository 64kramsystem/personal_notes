# Rails

- [Rails](#rails)
  - [Metadata](#metadata)
  - [Tooling](#tooling)
  - [Controllers](#controllers)
  - [ActiveRecord](#activerecord)
    - [Querying](#querying)
    - [Batching](#batching)
    - [Updating](#updating)
    - [Migrations](#migrations)
    - [Callbacks](#callbacks)
    - [Metadata](#metadata-1)
    - [SQL queries streaming](#sql-queries-streaming)
  - [Cache](#cache)

## Metadata

```rb
Rails.gem_version                 # Rails version in Gem::Version format
Rails.root                        # Rails root path (as Pathname; can also be used like `Pathname.new(__FILE__).relative_path_from(Rails.root)`
```

## Tooling

```sh
rails runner $code_or_filename   # also filename is valid
```

## Controllers

Predefined variables:

- `action_name`: current action name

## ActiveRecord

Query hints (6.0+):

```ruby
Article.joins(:user).optimizer_hints("JOIN_ORDER(articles, users)").to_sql
# => SELECT /*+ JOIN_ORDER(articles, users) */ `articles`.* FROM `articles` INNER JOIN `users` ON `users`.`id` = `articles`.`user_id`
```

Table aliases require manual AREL:

```rb
MyModel.from(Arel::Table.new(:table_name).alias("alias")).where(...)
```

### Querying

```ruby
query.ids                   # pluck the ids!
query.pluck(:column)        # select only the single column, without constructing AR instances
query.pick(:column)         # limit(1).pluck(:column)
arel.table                  # name of the main AREL query table

# OR operator.
# If in the MyTable scope, where second where doesn't need qualification.
#
MyTable.where(cond1).or(MyTable.where(cond2))

# Ranges (don't support `>`)
#
column: 1..     # column >= 1
column: 1...    # column >= 1 (!)
column: ...7    # column < 7
column: 1..7    # column BETWEEN 1 AND 7
```

Scopes:

```rb
class Event
  scope :tagged, ->(name) do
    where(tag: name)
  end
end
```

In order to use scope in joins, use `merge()`:

```rb
Show.joins(:events).merge(Event.tagged('fun'))
```

Group by/having:

```rb
customers.join(:orders).group('customers.id').having('count(*) > 10')
```

### Batching

Both `:find_each` and `:find_in_batches` have a default batch size of 1000.

### Updating

```ruby
instance.update_columns(a: 'b')     # skips all the logic, except serialization

# Use SQL as set value of a column
#
Model.update_all(attribute: Arel.sql('other_attribute - interval 1 day'))
```

### Migrations

Reference: https://apidock.com/rails/ActiveRecord/ConnectionAdapters/SchemaStatements

Table creation/table column operations:

```ruby
create_table id: false, primary_key: :column, options: 'ENGINE=MyISAM' do |t|
  # :type: can be a string, e.g. 'MEDIUMINT UNSIGNED NOT NULL'
  #
  # :column_options (valid for all the column methods):
  #   :null:    true/false
  #   :limit:   int; number of chars for string, bytes otherwise
  #   :default
  #
  t.column :column_name, :type, **column_options

  t.string
  t.text :info, limit: 64.kilobytes, null: false    # create a mediumtext (<64k is text)
  t.integer
  t.float
  t.decimal
  t.datetime
  t.timestamp
  t.time
  t.date
  t.binary
  t.boolean
  t.timestamps

  t.index :column_name{, column_name_N}, unique: true, name: index_name
end

# The create_table methods also apply here.
# :bulk perfoms all the changes in one statement (defaults to false)
#
change_table :table, bulk: true do |t|
  t.change                             # changes the column definition
  t.change_default                     # changes the column default
  t.rename                             # rename a column
  t.references
  t.belongs_to

  t.remove :column_name                # remove a column
  t.remove_index :column_name{, column_name_N}  # replace also be used like `Pathname.new(__FILE__).relative_path_from(Rails.root)``name: index_name` if non-rails naming
  t.remove_references
  t.remove_belongs_to
  t.remove_timestamps
end
```

Direct operations:

```ruby
rename_table :old_table_name, :new_table_name
drop_table :table_name
change_column :table, ... # see options in :create_table
change_column_default :table, :column, :default
remove_index :table, ...  # see options in :change_table

# Foreign keys
#
add_foreign_key :articles, :authors
remove_foreign_key :articles, [:authors | column: author_id | name: fk_abc123]
```

Irreversible migration error:

```ruby
raise ActiveRecord::IrreversibleMigration
```

### Callbacks

In order to find the difference on save, use:

- `before_save` -> `changes()`
- `after_save` -> `saved_changes()`

The format is `{"field" => [before, after]}`; unchanged fields are not included.

### Metadata

```ruby
# Columns metadata. Watch out: they (obviously) depend on the DB existence; if, for example, Rake tasks
# indirectly cause this code to load while the db doesn't exist (yet), they will fail.
#
MyModel.columns_hash['my_attribute'].limit
```

### SQL queries streaming

For streaming of direct SQL, one must use the `mysql2` gem APIs - see its [notes section](ruby_libraries.md#mysql2); the low-level connection is retrieved via `connection.raw_connection`.

## Cache

```rb
Rails.cache.write(key, value, expires_in: duration) # ActiveSupport::Duration works
Rails.cache.delete(key)
```
