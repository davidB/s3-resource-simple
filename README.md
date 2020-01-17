# Simple S3 Resource for [Concourse CI](http://concourse.ci)

Resource to upload files to S3. Unlike the [the official S3 Resource](https://github.com/concourse/s3-resource), this Resource can upload or download multiple files.

## Usage

Include the following in your Pipeline YAML file, replacing the values in the angle brackets (`< >`):

```yaml
resource_types:
- name: s3-sync
  type: docker-image
  source:
    repository: davidb31/s3-resource-simple
resources:
- name: <resource name>
  type: <resource type name>
  source:
    access_key_id: ((aws-access-key))
    secret_access_key: ((aws-secret-key))
    bucket: ((aws-bucket))
    path: [<optional>, use to sync to a specific path of the bucket instead of root of bucket]
    options: [<optional, see note below>]
    region: <optional, see note below>
jobs:
- name: <job name>
  plan:
  - <some Resource or Task that outputs files>
  - put: <resource name>
    params:
      options: [<optional, see note below>]
      dir: <optional, see note below>
```

### AWS Credentials

The `access_key_id` and `secret_access_key` are optional and if not provided the EC2 Metadata service will be queried for role based credentials.

### `options`

The `options` parameter is synonymous with the options that `aws cli` accepts for `sync`. Please see [S3 Sync Options](http://docs.aws.amazon.com/cli/latest/reference/s3/sync.html#options) and pay special attention to the [Use of Exclude and Include Filters](http://docs.aws.amazon.com/cli/latest/reference/s3/index.html#use-of-exclude-and-include-filters).
Final `options` is the concatenation of the one defined into "resources/source" and into "put/params".

### `dir`

Use to upload from a subfolder of the working directory.

Given the following directory `test`:

```txt
test
├── results
│   ├── 1.json
│   └── 2.json
└── scripts
    └── bad.sh
```

we can upload _only_ the `results` subdirectory by using the following `dir` in our put configuration:

```yaml
      - put: "results-bucket"
        params:
          dir: 'results'
```

### `region`

Interacting with some AWS regions (like London) requires AWS Signature Version
4. This options allows you to explicitly specify region where your bucket is
located (if this is set, AWS_DEFAULT_REGION env variable will be set accordingly).

```yaml
region: eu-west-2
```
