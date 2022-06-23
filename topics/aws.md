# AWS

- [AWS](#aws)
  - [IAM](#iam)
    - [Permissions](#permissions)
  - [S3](#s3)
    - [Fileshare/Permissions](#filesharepermissions)
  - [RDS](#rds)
    - [Support stored procedures](#support-stored-procedures)
  - [DynamoDB](#dynamodb)

## IAM

### Permissions

Permissions can use variables, for example:

```json
// If associated to a EC2 profile, this gives the permission to the caller instance.
// See: https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html.
//
Resource = "arn:aws:ec2:eu-west-1:01234567:instance/${ec2-instance-id}"
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
-- | rds_set_configuration             |
-- | rds_set_fk_checks_off             |
-- | rds_set_fk_checks_on              |
-- | rds_show_configuration            |
-- | rds_skip_repl_error               | # Skip slave replication errors
-- | rds_start_replication             |
-- | rds_stop_replication              |
-- | ...                               |
-- +-----------------------------------+
```

## DynamoDB

In order to list the content of a table, perform a "scan".
