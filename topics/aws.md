# AWS

- [AWS](#aws)
  - [EC2](#ec2)
  - [IAM](#iam)
    - [Permissions](#permissions)
  - [Cloudwatch](#cloudwatch)
    - [Agent (config)](#agent-config)
  - [S3](#s3)
    - [Fileshare/Permissions](#filesharepermissions)
  - [RDS](#rds)
    - [Support stored procedures](#support-stored-procedures)
    - ["Create Read Replica" greyed out](#create-read-replica-greyed-out)
  - [DynamoDB](#dynamodb)
    - [Permissions](#permissions-1)
    - [TTL](#ttl)
  - [WAF](#waf)
  - [Lambda](#lambda)

## EC2

vCPU threads/cores info: https://docs.aws.amazon.com/ec2/latest/instancetypes/gp.html#gp_hardware

## IAM

### Permissions

WATCH OUT!! Permissions don't always support specific/scoped resources - verify them in the console!

Permissions can use variables, for example:

```json
// If associated to a EC2 profile, this gives the permission to the caller (self) instance.
// See: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html.
//
"Resource": "arn:aws:ec2:eu-west-1:01234567:instance/${ec2-instance-id}"
```

## Cloudwatch

### Agent (config)

```js
"cpu": {
  // Either specify `resources` (reports per-CPU) or `totalcpu` (aggregate).
  "resources": ["*"],
  "totalcpu": true
},
"disk": {
  "measurement": ["used_percent"],
  // Use the mountpoint for this metric
  "resources": ["/"]
},
"diskio": {
  "measurement": ["io_time"],
  // Use the device for this metric
  "resources": ["/dev/nvme0n1p1"]
},
```

## S3

Bucket public URL format: https://mybucket.s3.amazonaws.com

When versioning is enabled, uploaded files have `VersionId` set; it's null otherwise.

When the storage class of an object is changed, a new version is created (even if versioning is disabled), with updated `LastModified`, `IsLatest` = true (and according `VersionId`).

### Fileshare/Permissions

If one needs to exchange files, it's possible to use private bucket/objects, with [presigned URLs](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ShareObjectPreSignedURL.html); they temporarily grant the URL client the permissions of the entity that created it. Ruby implementation:

```rb
s3_object = @s3_client.bucket(BUCKET).object(object_key)
s3_object.put(body: file_io, acl: 'private')
s3_object.presigned_url(:get, expires_in: URL_EXPIRY, secure: true).to_s
```

## RDS

It's not possible to explain queries run by other users, even with the root user, since the `SUPER` privilege is required.

### Support stored procedures

RDS provides some SPs to perform jobs not allowed to non-root users:

```sql
SELECT routine_name
FROM information_schema.routines
WHERE routine_type = 'PROCEDURE'
      -- AND routine_schema = @database_name
      AND routine_name LIKE 'rds\_%';

-- +-----------------------------------+
-- | rds_innodb_buffer_pool_dump_now   |
-- | rds_innodb_buffer_pool_load_abort |
-- | rds_innodb_buffer_pool_load_now   |
-- | rds_kill                          | # Kill a thread: `CALL mysql.rds_kill(@tid);`
-- | rds_kill_query                    |
-- | rds_next_master_log               |
-- | rds_rotate_general_log            |
-- | rds_rotate_global_status_history  |
-- | rds_rotate_slow_log               |
-- | rds_set_configuration             | # Set RDS config value; get via `rds_show_configuration()`
-- | rds_set_fk_checks_off             |
-- | rds_set_fk_checks_on              |
-- | rds_show_configuration            |
-- | rds_skip_repl_error               | # Skip slave replication error and restart the replication (see `sql_slave_skip_counter`)
-- | rds_start_replication             |
-- | rds_stop_replication              |
-- | ...                               |
-- +-----------------------------------+
```

### "Create Read Replica" greyed out

Enable backups.

## DynamoDB

In order to list the content of a table, perform a "scan".

Writes are performed via `put` and `update`, however WATCH OUT!!:

- `update` doesn't add new columns where it performs an insert
- `put` does!

Supported data types:

- `S`: strings
- `N`: numeric -> use for timestamps (unix time)
- `B`: binary
- `BOOL`: boolean
- (...)

References:

- https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/time-to-live-ttl-before-you-start.html
- https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBMapper.DataTypes.html
- https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/HowItWorks.NamingRulesDataTypes.html

### Permissions

For basic CRUD, give:

- `dynamodb:GetItem`
- `dynamodb:PutItem`
- `dynamodb:UpdateItem`
- `dynamodb:Scan`

They're granular, so `Scan` doesn't allow getting an item.

### TTL

Item expiry needs to be activated in order to work; the column must be Number data type, and the value in epoch format.

WATCH OUT!:

- it can take 1 hour for expiry to be actually activated
- it can take up to 48 hours for an expired item to be deleted; in the meanwhile, searches must explicitly add a condition

There is a TTL preview in the console, but didn't yield any value when tried it.

## WAF

The simplest way to configure a "log mode", is to allow non-matching rules, and enable the Cloudfront logging, then, cloud in Log Insights:

    stats count() as requestCount by httpRequest.uri
    | filter terminatingRuleId = "Default_Action"
    | sort requestCount desc

## Lambda

Actions below raise an error if the Lambda name is not correct.

```sh
# Manually invoke a Lambda, then follow its logs.
#
aws lambda invoke --function-name my_lambda --payload '{}' /dev/stdout &!
aws logs tail /aws/lambda/my_lambda --follow
```
