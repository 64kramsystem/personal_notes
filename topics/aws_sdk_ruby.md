# AWS SDK Ruby v3

- [AWS SDK Ruby v3](#aws-sdk-ruby-v3)
  - [EC2](#ec2)
  - [Target groups](#target-groups)
  - [Waiters](#waiters)
  - [DynamoDB](#dynamodb)

## EC2

Find the instance id:

```rb
require 'aws-sdk-core'
Aws::EC2Metadata.new.get("/latest/meta-data/instance-id")
```

## Target groups

Find all the target group arns:

```rb
target_groups_data, next_page = @lb_client.describe_target_groups

raise "Next page found while calling :describe_target_groups" if next_page

target_group_arns = target_groups_data.target_groups.map(&:target_group_arn)
```

Find ther target healths for each target group, and search the matching instance id:

```rb
target_group_arns.map do |target_group_arn|
  target_group_health_states = @lb_client.describe_target_health(target_group_arn: target_group_arn).target_health_descriptions

  return target_group_arn if target_group_health_states.keys.include?(instance_id)
end
```

De/register a target:

```rb
targets_data = {
  target_group_arn: target_group_arn,
  targets: [{id: instance_id}],
}

# Use #register to register.
@lb_client.deregister_targets(targets_data)
```

## Waiters

Waiters take a name, and a parameters hash with a structure equal to the related resource describe API (see example link).

```rb
# See https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/ElasticLoadBalancingV2/Waiters.html

targets_data = {
  target_group_arn: registration_data.target_group_arn,
  targets: [{id: registration_data.instance_id}],
}

lb_client.wait_until(:target_deregistered, targets_data) do |waiter|
  waiter.delay = WAIT_INTERVAL                    # interval between attempts
  waiter.max_attempts = WAIT_TIME / waiter.delay
end
```

## DynamoDB

Reference: https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/DynamoDB/Client.html#update_item-instance_method.

List all items ("scan"):

```rb
client.scan(table_name: "Music")
```

Read an item:

```rb
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
```

Upsert an item:

```rb
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
