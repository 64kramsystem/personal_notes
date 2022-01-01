# AWS SDK Ruby v3

- [AWS SDK Ruby v3](#aws-sdk-ruby-v3)
  - [DynamoDB](#dynamodb)

## DynamoDB

Reference: https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/DynamoDB/Client.html#update_item-instance_method.

```rb
# Read an item
# Raises an error if the table doesn't exist.
# Integers can be returned as BigDecimal.
#
response = client.get_item(
  table_name: "Music",
  key: {
    "Artist" => "Acme Band",
    "SongTitle" => "Happy Day",
  },
)

reponse.to_h == {
  item: {
    "AlbumTitle" => "Songs About Life",
    "Artist" => "Acme Band",
    "SongTitle" => "Happy Day",
  },
}

# Update an item
#
response = client.update_item(
  table_name: "Music",
  expression_attribute_names: {
    "#AT" => "AlbumTitle",
    "#Y" => "Year",
  },
  expression_attribute_values: {
    ":t" => "Louder Than Ever",
    ":y" => "2015",
  },
  key: {
    "Artist" => "Acme Band",
    "SongTitle" => "Happy Day",
  },
  return_values: "ALL_NEW",  # Optional
  update_expression: "SET #Y = :y, #AT = :t",
)
```
