[![Imports: isort](https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336)](https://pycqa.github.io/isort/)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

# dbt-athena

* Supports dbt version `1.3.*`
* Supports [Seeds][seeds]
* Correctly detects views and their columns
* Support [incremental models][incremental]
  * Support two incremental update strategies: `insert_overwrite` and `append`
  * Does **not** support the use of `unique_key`
* **Only** supports Athena engine 2
  * [Changing Athena Engine Versions][engine-change]
* Does not support [Python models][python-models]

[seeds]: https://docs.getdbt.com/docs/building-a-dbt-project/seeds
[incremental]: https://docs.getdbt.com/docs/building-a-dbt-project/building-models/configuring-incremental-models
[engine-change]: https://docs.aws.amazon.com/athena/latest/ug/engine-versions-changing.html
[python-models]: https://docs.getdbt.com/docs/build/python-models#configuring-python-models

### Installation

* `pip install dbt-athena-community`
* Or `pip install git+https://github.com/dbt-athena/dbt-athena.git`

### Prerequisites

To start, you will need an S3 bucket, for instance `my-staging-bucket` and an Athena database:

```sql
CREATE DATABASE IF NOT EXISTS analytics_dev
COMMENT 'Analytics models generated by dbt (development)'
LOCATION 's3://my-staging-bucket/'
WITH DBPROPERTIES ('creator'='Foo Bar', 'email'='foo@bar.com');
```

Notes:
- Take note of your AWS region code (e.g. `us-west-2` or `eu-west-2`, etc.).
- You can also use [AWS Glue](https://docs.aws.amazon.com/athena/latest/ug/glue-athena.html) to create and manage Athena databases.

### Credentials

This plugin does not accept any credentials directly. Instead, [credentials are determined automatically](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html) based on `aws cli`/`boto3` conventions and
stored login info. You can configure the AWS profile name to use via `aws_profile_name`. Checkout DBT profile configuration below for details.

### Configuring your profile

A dbt profile can be configured to run against AWS Athena using the following configuration:

| Option          | Description                                                                     | Required?  | Example             |
|---------------- |-------------------------------------------------------------------------------- |----------- |-------------------- |
| s3_staging_dir  | S3 location to store Athena query results and metadata                          | Required   | `s3://bucket/dbt/`  |
| region_name     | AWS region of your Athena instance                                              | Required   | `eu-west-1`         |
| schema          | Specify the schema (Athena database) to build models into (lowercase **only**)  | Required   | `dbt`               |
| database        | Specify the database (Data catalog) to build models into (lowercase **only**)   | Required   | `awsdatacatalog`    |
| poll_interval   | Interval in seconds to use for polling the status of query results in Athena    | Optional   | `5`                 |
| aws_profile_name| Profile to use from your AWS shared credentials file.                           | Optional   | `my-profile`        |
| work_group| Identifier of Athena workgroup   | Optional   | `my-custom-workgroup`        |
| num_retries| Number of times to retry a failing query | Optional  | `3`  | `5`

**Example profiles.yml entry:**
```yaml
athena:
  target: dev
  outputs:
    dev:
      type: athena
      s3_staging_dir: s3://athena-query-results/dbt/
      region_name: eu-west-1
      schema: dbt
      database: awsdatacatalog
      aws_profile_name: my-profile
      work_group: my-workgroup
```

_Additional information_
* `threads` is supported
* `database` and `catalog` can be used interchangeably

### Usage notes

### Models

#### Table Configuration

* `external_location` (`default=none`)
  * The location where Athena saves your table in Amazon S3
  * If `none` then it will default to `{s3_staging_dir}/tables`
  * If you are using a static value, when your table/partition is recreated underlying data will be cleaned up and overwritten by new data
* `partitioned_by` (`default=none`)
  * An array list of columns by which the table will be partitioned
  * Limited to creation of 100 partitions (_currently_)
* `bucketed_by` (`default=none`)
  * An array list of columns to bucket data
* `bucket_count` (`default=none`)
  * The number of buckets for bucketing your data
* `format` (`default='parquet'`)
  * The data format for the table
  * Supports `ORC`, `PARQUET`, `AVRO`, `JSON`, or `TEXTFILE`
* `write_compression` (`default=none`)
  * The compression type to use for any storage format that allows compression to be specified. To see which options are available, check out [CREATE TABLE AS][create-table-as]
* `field_delimiter` (`default=none`)
  * Custom field delimiter, for when format is set to `TEXTFILE`

More information: [CREATE TABLE AS][create-table-as]

[run_started_at]: https://docs.getdbt.com/reference/dbt-jinja-functions/run_started_at
[invocation_id]: https://docs.getdbt.com/reference/dbt-jinja-functions/invocation_id
[create-table-as]: https://docs.aws.amazon.com/athena/latest/ug/create-table-as.html

#### Supported functionality

Support for incremental models:
* Support two incremental update strategies with partitioned tables: `insert_overwrite` and `append`
* Does **not** support the use of `unique_key`

Due to the nature of AWS Athena, not all core dbt functionality is supported.
The following features of dbt are not implemented on Athena:
* Snapshots

#### Known issues

* Quoting is not currently supported
  * If you need to quote your sources, escape the quote characters in your source definitions:

  ```yaml
  version: 2

  sources:
    - name: my_source
      tables:
        - name: first_table
          identifier: "first table"       # Not like that
        - name: second_table
          identifier: "\"second table\""  # Like this
  ```

* Tables, schemas and database should only be lowercase
* **Only** supports Athena engine 2
  * [Changing Athena Engine Versions][engine-change]

### Contributing

This connector works with Python from 3.7 to 3.10.

#### Getting started
In order to start developing on this adapter clone the repo and run this make command (see [Makefile](Makefile)) :

```bash
make setup
```

It will :
1. Install all dependencies.
2. Install pre-commit hooks.

Next, configure the environment variables in [dev.env](dev.env) to match your Athena development environment.

#### Running tests
You can run the tests using `make`:

```bash
make run_tests
```

### Community

* [fishtown-analytics/dbt][fishtown-analytics/dbt]
* [fishtown-analytics/dbt-presto][fishtown-analytics/dbt-presto]
* [Dandandan/dbt-athena][Dandandan/dbt-athena]
* [laughingman7743/PyAthena][laughingman7743/PyAthena]

[fishtown-analytics/dbt]: https://github.com/fishtown-analytics/dbt
[fishtown-analytics/dbt-presto]: https://github.com/fishtown-analytics/dbt-presto
[Dandandan/dbt-athena]: https://github.com/Dandandan/dbt-athena
[laughingman7743/PyAthena]: https://github.com/laughingman7743/PyAthena
