# AWS SDK Ruby v3

- [AWS SDK Ruby v3](#aws-sdk-ruby-v3)
  - [Responses paging](#responses-paging)
  - [EC2](#ec2)
  - [Target groups](#target-groups)
  - [Waiters](#waiters)
  - [DynamoDB](#dynamodb)
  - [S3](#s3)
  - [Samples](#samples)
    - [Informations EC2/Load balancing](#informations-ec2load-balancing)

## Responses paging

Many APIs return a [PageableResponse](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/PageableResponse.html); the options to iterate are:

```rb
# Automatic (typically, flat_map(&:collection) is used).
# WATCH OUT! Old gem versions may not implement paginations, causing this to return only the first
# page's content.

response.flat_map(&:items).filter_map { it["MyField"] }

# Manual

result = []
while response.next_page? do
  response = response.next_page
  result << response.key
end
```

## EC2

Find the instance id (from the instance):

```rb
require 'aws-sdk-core'
Aws::EC2Metadata.new.get("/latest/meta-data/instance-id")
```

Find instances:

```rb
# Basic filtering.
#
resp = @ec2_client.describe_instances(
  filters:[
    { name: 'tag:mytag',           values: %w[tag1] },
    { name: 'instance-state-name', values: %w[stopped] },
    # ...
  ],
)

resp == {
  reservations: [
    instances: [
      {
        instance_id: "InstanceId",
      }
    ],
  ],
  # ...
  next_token: "String",
}
```

Start instances:

```rb
resp = client.start_instances({
  instance_ids: ["i-1234567890abcdef0"],
})
```

## Target groups

Find all the target group arns:

```rb
# See https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/ElasticLoadBalancingV2/Client.html#describe_target_groups-instance_method
# As of Jan/2024, filtering options are very limited (LB/TG ARN, TG name)

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

Numeric items returned are of BigDecimal type.

List all items ("scan"):

```rb
resp = client.scan(
  table_name: "Music"
  expression_attribute_values: {    # optional
    ":e" => Time.now.to_i
  },
  filter_expression: "Expiry > :e", # ^^optional
)

resp.items == [
  [{"Key"=>"foo", "Val"=>0.1656062109e10}]
]
```

Read an item:

```rb
# Raises an error if the table doesn't exist.
# #item is nil if the item is not found.
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

WATCH OUT! See [aws section](aws.md#dynamodb) for differences between put and update (they're both upserts).

Put an item

```rb
response = client.put_item(
  table_name: "Music",
  item: {
    "Artist" => "Acme Band",
    "SongTitle" => "Happy Day",
  },
)
```

Update an item:

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

## S3

Upload a file:

```rb
# Higher level (e.g. large files handling); use this.
#
# :source:  String, IO and others
# :options: other options, including progress callback and threading
#
Aws::S3::Object.new.upload_file(source, options={})

# Use this only if lower-level control is required.
#
Aws::S3::Object.new.put(tons_of_options={})
```

Collect the keys matching a pattern:

```rb
loop.inject([nil, []]) do |(continuation_token, log_keys)|
  response = @client.list_objects_v2(bucket: BUCKET, prefix: keys_prefix, continuation_token:)

  response.contents.each do |object|
    log_keys << object.key if File.fnmatch(keys_pattern, object.key)
  end

  if response.is_truncated
    [response.next_continuation_token, log_keys]
  else
    break log_keys
  end
end
```

## Samples

### Informations EC2/Load balancing

Find the host taget statuses (quite a pain):

```rb
def find_hosts_target_status(hostnames)
  target_group_arns = @lb_client
    .describe_target_groups
    .flat_map(&:target_groups)
    .map(&:target_group_arn)

  # {id => state}
  target_states = target_group_arns
    .map { |arn| @lb_client.describe_target_health(target_group_arn: arn) }
    .flat_map(&:target_health_descriptions)
    .each_with_object({}) { |target, result| result[target.target.id] = target.target_health.state }

  # {hostname => state}
  @ec2_client
    .describe_instances(filters: [{name: 'tag:Hostname', values: hostnames}])
    .flat_map(&:reservations)
    .flat_map(&:instances)
    .select { |instance| instance.state.name != 'terminated' }
    .each_with_object({}) do |instance, result|
      hostname = instance.tags.find { |tag| tag.key == 'Hostname' }.value
      result[hostname] = target_states[instance.instance_id] || 'unattached'
    end
end
```
