Concourse AMI Resource
======================

A custom Concourse CI resource to find, describe, and create AWS AMIs.

Behavior
--------

### `check` - Checks for new AMIs

Given a set of filters, check for new AMIs. The AMI ID is used as the resource
version.

#### Parameters

 - [`credentials`](#credentials)
 - `region` - AWS region
 - `filters` - Map of filters, see [AWS CLI describe-images documentation on filters](aws-di-filters)

[aws-di-filters]: https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-images.html#options


### `in` - Fetch information about a specific AMI

Creates the following files in the destination:

 - `ami.json` - Full output of image description, see [AWS CLI describe-images documentation examples](aws-di-examples)
   for examples of output.
 - `id` - ID of AMI, e.g. `ami-xxxxxxxx`
 - `packer.json` - ID of AMI in Packer's `var-file` format, e.g.
   ```
   {"source_ami": "ami-xxxxxxxx"}
   ```

#### Parameters

 - [`credentials`](#credentials)
 - `region` - AWS region

[aws-di-examples]: https://docs.aws.amazon.com/cli/latest/reference/ec2/describe-images.html#examples


### `out` - Copies an AMI

#### Parameters

 - [`credentials`](#credentials)
 - `source_region` - AWS AMI source region
 - `region` - AWS AMI destination region
 - `name` - Name of AMI
 - `description` - Description for the new AMI, if not specified will use
   source AMI description
 - `kms_id` - Encrypt image with KMS, if not specified image will be
   unencrypted

### Generic Parameters

#### credentials
 - `access_key`
 - `secret_id`
 - `session_token`
 - `file`

The credentials `file` is expected to be an AWS credentials formatted file,
like:
```
[default]
aws_access_key_id = XXXX
aws_secret_access_key = XXXX
aws_session_token = XXXX
```

The credentials file will be overridden by the `access_key`, `secret_id`, and
`session_token` values. If no values are specified, the IAM profile of the
Concourse instance will be used (if available).


## Example

```
resource_types:
  - name: ami
    type: docker-image
    source:
      repository: ioben/concourse-ami-resource

resources:
  - name: amazon-linux-ami
    type: ami
    check_every: 1h
    source:
      credentials:
        file: aws/credentials
      region: ((aws-region))
      filters:
        architecture: x86_64
        ena-support: true
        owner-id: 137112412989
        is-public: true
        state: available

jobs:
  - name: AMI factory
    plan:
      - get: amazon-linux-ami
        trigger: true
```
