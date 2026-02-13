# Rails

- [Rails](#rails)
  - [Metadata](#metadata)
  - [Tooling](#tooling)
  - [Controllers](#controllers)
    - [Filters](#filters)
  - [ActiveRecord](#activerecord)
    - [Querying](#querying)
      - [Complex AREL query examples](#complex-arel-query-examples)
      - [Batching](#batching)
      - [Metadata](#metadata-1)
    - [Updating](#updating)
    - [Manual AREL query build](#manual-arel-query-build)
    - [Migrations](#migrations)
    - [Models](#models)
      - [Associations](#associations)
      - [Callbacks](#callbacks)
      - [Metadata/connection](#metadataconnection)
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

- `action_name()`: current action name

### Filters

```rb
before_action :method
```

## ActiveRecord

### Querying

```ruby
query.ids                   # pluck the ids!
query.pluck(:column)        # select only the single column, without constructing AR instances. NOTE: also works with arrays of hashes
query.pick(:column)         # limit(1).pluck(:column)
arel.table_name             # name of the base AREL query table
arel.model                  # name of the base AREL model

# OR operator.
# If in the MyTable scope, where second where doesn't need qualification.
#
MyTable.where(cond1).or(MyTable.where(cond2))

# Strategies to chain an optional condition.
#
MyTable.where(({foo:bar} if baz))            # where(nil) → ignored
MyTable.tap { it.where!(foo: bar) if baz }   # where!     → wow!

# Ranges (don't support `>`)
#
column: 1..     # column >= 1
column: 1...    # column >= 1 (!)
column: ..7     # column <= 7
column: ...7    # column < 7
column: 1..7    # column BETWEEN 1 AND 7
column: 1...7   # 1 <= column && column < 7

Hints:

Article.joins(:user).optimizer_hints("JOIN_ORDER(articles, users)").to_sql
# => SELECT /*+ JOIN_ORDER(articles, users) */ `articles`.* FROM `articles` INNER JOIN `users` ON `users`.`id` = `articles`.`user_id`
```

Table aliases require manual AREL:

```rb
MyModel.from(Arel::Table.new(:table_name).alias("alias")).where(...)
```

Scopes:

```rb
class Event
  scope :tagged, ->(name) do
    where(tag: name)
  end
end

# Invert/negate/opposite a scope:
#
scope :without_missing_coins, -> { merge(klass).with_missing_coins.invert_where }

# In order to use scope in joins, use `merge()`:
#
Show.joins(:events).merge(Event.tagged('fun'))
```

```rb
```

Group by/having (aggregates):

```rb
customers.join(:orders).group('customers.id').having('count(*) > 10')
```

#### Complex AREL query examples

```rb
# Join with left outer joins and OR conditions
Cart.
  distinct.
  left_joins(:line_items).
  left_joins(:payments).
  where(account_id: 1025).
  where(created_at: Date.new(2024, 1, 30)..).
  merge(LineItem.where.not(id: nil).or(Payment.where.not(id: nil)))

# Using AREL subqueries with IN operator
Cart.
  where(account_id: 1025).
  where(created_at: Date.new(2024, 1, 30)..).
  where(
    # carts.id IN (SELECT cart_id FROM line_items)
    Cart.arel_table[:id].in(LineItem.arel_table.project(:cart_id)).
    or(Cart.arel_table[:id].in(Payment.arel_table.project(:cart_id)))
  )
```

#### Batching

Both `:find_each` and `:find_in_batches` have a default batch size of 1000.

#### Metadata

- `AREL#select_values()`: find out if `:select` was called

### Updating

```ruby
instance.update_columns(a: 'b')     # skips all the logic, except serialization

# Use SQL as set value of a column
#
Model.update_all(attribute: Arel.sql('other_attribute - interval 1 day'))
```

### Manual AREL query build

Example of generating an UPDATE:

```rb
table = Arel::Table.new("#{schema}.#{model.table_name}")

updater = Arel::UpdateManager.new.table(table)

# `set_values` is a hash `column_name` => `value`, whose values are going to be quoted (by AREL);
# `set_fxs` is the same, but values are not quoted.
# UpdateManager#set() can't be invoked multiple times - each invocation overwrites the previous one.
#
all_sets = set_values
  .merge(set_fxs.transform_values { |sql| Arel.sql(sql) })
  .transform_keys { |column| table[column] }
updater = updater.set(all_sets)

updater = updater.where(table[filter_column].eq(filter_value))

ApplicationRecord.connection.update(updater.to_sql)
```

### Migrations

Reference: https://apidock.com/rails/ActiveRecord/ConnectionAdapters/SchemaStatements

Table creation/table column operations:

```ruby
# On modern Rails versions, references are BIGINT, not INT anymore.
#
create_table id: false, primary_key: :column, options: 'ENGINE=MyISAM' do |t|
  t.primary_key :id, :int, auto_increment: true       # create an INT PK; also adds the column

  # :type can be a string, e.g. 'MEDIUMINT UNSIGNED NOT NULL'
  # column_options: (valid for all the column methods):
  #   :null:    true/false
  #   :limit:   int; number of chars for string, bytes otherwise
  #   :default
  #
  t.column :column_name, :type, **column_options

  t.string
  t.text :mytext, limit: 64.kilobytes, null: false    # create a MEDIUMTEXT (remove :limit for the TEXT default)
  t.integer :myint, unsigned: true, limit: 2          # the limit is in bytes (!)
  t.float
  t.decimal
  t.datetime
  t.timestamp
  t.time
  t.date
  t.binary
  t.boolean
  t.timestamps
  t.enum :myenum, ['Value', ...]

  t.index :column_names, unique: true, name: index_name # use array on multiple col. names
  t.foreign_key :ref_table [, column_name: fk_col]      # see https://api.rubyonrails.org/v7.0.4/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_foreign_key
  t.references :blah, ...                               # very confusing
end

# The create_table methods also apply here.
# :bulk perfoms all the changes in one statement (defaults to false)
#
change_table :table, bulk: true do |t|
  t.mytype, :myname, ...               # add a column

  t.change                             # changes the column definition
  t.change_default                     # changes the column default
  t.rename                             # rename a column
  t.belongs_to
  t.foreign_key                        # see :add_foreign_key

  t.remove :column_name                # remove a column
  t.remove_index :column_name{, column_name_N}  # replace also be used like `Pathname.new(__FILE__).relative_path_from(Rails.root)``name: index_name` if non-rails naming
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
add_foreign_key :articles, :authors, [to_table: ref_table, column: base_col_name]
remove_foreign_key :articles, [:authors | column: author_id | name: fk_abc123]
```

Irreversible migration error:

```ruby
raise ActiveRecord::IrreversibleMigration
```

### Models

#### Associations

`inverse_of` is required to establish bidirectional associations, *only* when they have a scope or use :through/:foreign_key (see [here](https://stackoverflow.com/a/39670478/210029)).

```rb
class Client
  has_many :client_extras, foreign_key: :instance_id, inverse_of: :client
end

class ClientExtra
  belongs_to :client, foreign_key: :instance_id, inverse_of: :client_attributes
end
```

#### Callbacks

In order to find the difference on save, use:

- `before_save` -> `changes()`
- `after_save` -> `saved_changes()`

The format is `{"field" => [before, after]}`; unchanged fields are not included.

#### Metadata/connection

```ruby
# Columns metadata. Watch out: they (obviously) depend on the DB existence; if, for example, Rake tasks
# indirectly cause this code to load while the db doesn't exist (yet), they will fail.
#
MyModel.columns_hash['my_attribute'].limit

connection.current_database

# Connection configuration for the current Rails env.
# database_configuration() returns a new hash for each call.
#
MyNamespace::Application.instance.config.database_configuration[Rails.env]

# Other way to get the current db connection; duplication is needed when changing the data, because the hash is frozen.
#
conn_cfg = ActiveRecord::Base.connection_db_config.configuration_hash.deep_dup
#
# Update and reconnect.
#
ActiveRecord::Base.establish_connection(conn_cfg.tap {|cfg| cfg[:database] = new_database})
```

### SQL queries streaming

For streaming of direct SQL, one must use the `mysql2` gem APIs - see its [notes section](ruby_libraries.md#mysql2); the low-level connection is retrieved via `connection.raw_connection`.

## Cache

```rb
Rails.cache.write(key, value, expires_in: duration) # ActiveSupport::Duration works
Rails.cache.delete(key)
```
