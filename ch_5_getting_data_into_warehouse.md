# Data Engineering in the Cloud: Start to Finish

NOTE: This book is currently incomplete. If you find errors or would like to fill in the gaps, read the **Contributions** section [here](https://github.com/Nunie123/data_engineering_book).

## Table of Contents
[Preface](https://github.com/Nunie123/data_engineering_book) <br>
[Chapter 1: Data Engineering Responsibilities](https://github.com/Nunie123/data_engineering_book/blob/master/ch_1_data_engineering_responsibilities.md) <br>
[Chapter 2: Accessing Data](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md)<br>
[Chapter 3: Moving Data to Your Storage](https://github.com/Nunie123/data_engineering_book/blob/master/ch_3_moving_data_to_storage.md)<br>
[Chapter 4: Building Your Data Warehouse](https://github.com/Nunie123/data_engineering_book/blob/master/ch_4_building_data_warehouse.md)<br>
**Chapter 5: Getting Data into Your Warehouse**<br>
[Chapter 6: Transforming Your Batch Data](https://github.com/Nunie123/data_engineering_book/blob/master/ch_6_transforming_batch_data.md)<br>
Chapter 7: Orchestrating Your Pipelines<br>
Chapter 8: Streaming Your Data<br>
Chapter 9: Presenting Data to Your Users<br>
Chapter 10: Infrastructure as Code<br>
Chapter 11: EXAMPLE - Building Complete Date Engineering Infrastructure in AWS<br>
Chapter 12: EXAMPLE - Building Complete Date Engineering Infrastructure in GCP

---

# Chapter 5: Getting Data into Your Warehouse
So now we've got our Warehouse built, but it's not worth much until we get our data in. 

When setting up your data pipelines there are two main strategies, ELT and ETL, referring to the "Extract", "Load", and "Transform" phases of the data pipeline. In [Chapter 2](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md) we discussed the "Extract" part, and this chapter will discuss the "Load" part. In Chapter 6 we'll discuss the "Transform" part. Choosing between ELT and ETL will have a major impact on the technologies you use for transformation, but loading the data into your warehouse is only minimally impacted by whether you choose to do transformations before or after the load. In both ETL and ELT you're likely to be loading data from your storage solution (S3 or GCS). The major difference is in your Warehouse schema and the permissions you set on the destination table. Tables containing untransformed data will likely not by exposed to your users, whereas tables containing transformed data might be exposed to users.

## Differences Between AWS Redshift and Google BigQuery
Let's start by discussing some differences between Redshift and BigQuery: 
1. BigQuery behaves as a "sever-less" solution abstracting away considerations like how much compute power to dedicate to supporting your Warehouse infrastructure. Redshift, on the other hand, expects you to pick the amount of processing power in your cluster, giving you the flexibility and the responsibility of handling your Warehouse's performance and scaling. These different levels of abstraction require different pricing strategies, with BigQuery focused on the amount of data processed while using the service, while Redshift prices are based on the amount of compute power dedicated to your cluster. Both services charge you for the amount of storage you need.
2. Redshift expects you to interact with you database (creating tables, loading data, querying, etc.) as if you were connecting to a standard relational database. In fact, as you'll see below, when connecting through Python you'll be using an unmodified PostgreSQL driver. BigQuery, on the other hand, provides you with custom tooling in the form of a dedicated programming libraries and a command line tool. 
3. BigQuery allows nested fields, while Redshift does not. As a workaround, if you want to query nested JSON or PARQUET files in Redshift, you can leave the files in S3 and use [Redshift Spectrum](https://docs.aws.amazon.com/redshift/latest/dg/c-using-spectrum.html) to query S3 directly. BigQuery allows nested data to be loaded directly into tables, giving the tables a nested schema (which you will see how to define, below). This BigQuery feature allows you flexibility in your data structures and allows you to create tables that more closely match the source data. There are still limitations on this functionality, such as only allowing a nested depth of 15 levels and not allowing array-of-array fields. However, the array-of-objects structure you commonly see in JSON data is fully supported (though in BigQuery the terminology is an "ARRAY of STRUCTs"). A potential drawback to using this feature is that querying tables with nested fields may require your users to use unfamiliar SQL syntax. Be sure to work with your users if you intend to expose to them a table that includes nested fields.
4. BigQuery supports table partitioning, while Redshift only supports partitioning external tables. This is a nice feature that Redshift is conspicuously lacking, but in Redshift's defence it has [sort keys](https://docs.aws.amazon.com/redshift/latest/dg/t_Sorting_data.html) and [distribution keys](https://docs.aws.amazon.com/redshift/latest/dg/t_Distributing_data.html) that you can modify to improve table performance.


## Loading Data into AWS Redshift

### `aws` Command Line Tool
As mentioned in [Chapter 4](https://github.com/Nunie123/data_engineering_book/blob/master/ch_4_building_data_warehouse.md), the `aws redshift` commands are limited, mostly focusing on creating and maintaining Redshift clusters. Since we can't load out data using the command line, we'll focus on the `psycopg2` and `sqlalchemy-redshift` Python libraries, explained below.

### `psycopg2` and `sqlalchemy-redshift` Python Library
I went over connecting to Redshift using `psycopg2` and `sqlalchemy-redshift` in [Chapter 4](https://github.com/Nunie123/data_engineering_book/blob/master/ch_4_building_data_warehouse.md#user-content-psycopg2-and-sqlalchemy-redshift-python-library), so you should check that section first.

Now that you've got your connection set up, let's write some python code to load in your data.
``` python
from sqlalchemy import create_engine

def execute_raw_sql(connection_string: str, raw_sql: str) -> None:
    engine = create_engine(connection_string)
    with engine.connect() as connection:    # using `with` here lets us not worry about closing the connection (i.e. `connection.close()`)
        connection.execute(raw_sql)

connection_string = 'redshift+psycopg2://my_username:my_password@abc123.amazonaws.com:5439/my_database'
source_file_location = 's3://my_bucket/path/to/file.csv'
destination_table_name = 'my_table'
aws_access_key_id=os.getenv('AWS_ACCESS_KEY_ID'),
aws_secret_access_key=os.getenv('AWS_SECRET_ACCESS_KEY_ID')
raw_sql = (
    f' copy {destination_table_name} from {source_file_location} '
    f' credentials "aws_access_key_id={aws_access_key_id};aws_secret_access_key={aws_secret_access_key}" '
     ' format as csv; '
    )

execute_raw_sql(connection_string, raw_sql)
```
The above example uses access keys to authenticate, but you can also [authenticate using IAM roles](https://docs.aws.amazon.com/redshift/latest/dg/copy-parameters-authorization.html). 

The table loaded into must already exist and you should be careful the columns in the file and the table are in a matching order.

The above example loads a CSV file, but the `copy` command supports several [file formats](https://docs.aws.amazon.com/redshift/latest/dg/copy-parameters-data-format.html), including custom delimiters (e.g. TSV) and columnar file types (e.g. parquet). Also note that Redshift natively supports copying JSON files, though the file may need some processing before it can be loaded. You can read more about ingesting JSON files [here](https://docs.aws.amazon.com/redshift/latest/dg/copy-usage_notes-copy-from-json.html).

In addition to loading data directly into Redshift from S3, you can create what are called "external tables". External tables can be queried like regular tables, but the data is never loaded into Redshift. Under the hood, the data remains in S3 and some powerful tooling allows you to quickly query the S3 files, but all of that is abstracted away from you. External tables are nice because you can save money on storage costs and they allow you to query nested data in JSON and Parquet files, the drawback is that they are not as fast to query as native tables.
``` python
from sqlalchemy import create_engine

def execute_raw_sql(connection_string: str, raw_sql: str) -> None:
    engine = create_engine(connection_string)
    with engine.connect() as connection:    # using `with` here lets us not worry about closing the connection (i.e. `connection.close()`)
        connection.execute(raw_sql)

connection_string = 'redshift+psycopg2://my_username:my_password@abc123.amazonaws.com:5439/my_database'
source_files_location = 's3://my_bucket/path/to/files/'
source_file_type = 'parquet'
destination_table_name = 'my_external_table'
database_name = 'my_database'
schema_name = 'my_external_schema'
create_schema_sql = (
    f' create external schema {schema_name} '
     ' from data catalog '
    f' database {database_name} '
     ' iam_role "arn:aws:iam::123456789012:role/MyRedshiftRole" '
    )
create_table_sql = (
    f' create external table {database_name}.{schema_name}.{table_name} '
     ' (transaction_id INT, transaction_type VARCHAR, transaction_date DATE) '
    f' stored as {source_file_type} '
    f' location {source_files_location} '
    )

execute_raw_sql(connection_string, create_schema_sql)
execute_raw_sql(connection_string, create_table_sql)
```
External tables can't be put inside a regular schema, which is why I created an external schema, above. Note that we needed to provide an IAM ARN string for authentication when creating the schema, which you can read more about [here](https://docs.aws.amazon.com/redshift/latest/dg/c-spectrum-iam-policies.html). For the source location we provided a directory, which means that all of the parquet files in that directory (and any sub-directories) will be included in the table. Alternatively, we could have specified a single file to be the source of the external table.

Querying external tables uses the AWS service [Redshift Spectrum](https://docs.aws.amazon.com/redshift/latest/dg/c-using-spectrum.html), which charges $5.00 per terabyte of data scanned, on top of any other Redshift costs.

## Loading Data into Google BigQuery

### `bq` Command Line Tool
Unlike Redshift, the command line tool for BigQuery is a good option for loading data into your warehouse. In [Chapter 2](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md#user-content-gsutil-command-line-tool) I explained how to set up the Google Cloud SDK on your machine in order to use the `gsutil` tool. If you completed that then you're already set to use the `bq` tool. If not, then take a look at [Chapter 2](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md#user-content-gsutil-command-line-tool) or review the instructions [here](https://cloud.google.com/sdk/docs).


We talked briefly about the `bq load` command in [Chapter 4](https://github.com/Nunie123/data_engineering_book/blob/master/ch_4_building_data_warehouse.md#user-content-bq-command-line-tool), but here we'll go into more depth. 


The major pieces of information you'll need to know for your load operation are:
* The destination project, dataset, and table
* The source file location
* The source file format ([Avro](https://cloud.google.com/bigquery/docs/loading-data-cloud-storage-avro), [Parquet](https://cloud.google.com/bigquery/docs/loading-data-cloud-storage-parquet), [ORC](https://cloud.google.com/bigquery/docs/loading-data-cloud-storage-orc), [CSV](https://cloud.google.com/bigquery/docs/loading-data-cloud-storage-csv), [JSON](https://cloud.google.com/bigquery/docs/loading-data-cloud-storage-json))
* The schema of the destination table
* Whether you are replacing or appending the current table (`--replace` and `--noreplace` flags)
* How you are choosing to partition the table

Let's first start by defining our schema. As mentioned in [Chapter 4](https://github.com/Nunie123/data_engineering_book/blob/master/ch_4_building_data_warehouse.md#user-content-bq-command-line-tool), the load command will create a table if one does not exist, so providing the schema here will define your table. The `--autodetect` flag can be used to have BigQuery infer your schema from the data, but that should generally be avoided outside of development and testing. Let's look at a schema definition file:
``` json
[
    {
        "name": "product_id",
        "type": "INT64",
        "mode": "REQUIRED",
        "description": "The unique ID for the product."
    },{
        "name": "product_name",
        "type": "STRING",
        "mode": "NULLABLE",
        "description": "The name of the product."
    },{
        "name": "product_tags",
        "type": "RECORD",
        "mode": "REPEATED",
        "description": "All tags associated with the product.",
        "fields": [{
            "name": "tag_id",
            "type": "INT64",
            "mode": "NULLABLE",
            "description": "The unique ID for the tag."
        }, {
            "name": "tag_name",
            "type": "STRING",
            "mode": "NULLABLE",
            "description": "The name of the tag."
        }]
    },{
        "name": "created_on",
        "type": "DATE",
        "mode": "REQUIRED",
        "description": "The date the product was added to inventory."
    }
]
```
You'll notice that this schema defines a nested field, called a "REPEATED" field in BigQuery. Fields can be nested up to 15 layers deep. The schema above shows a field of REPEATED RECORDS, which is the equivalent of and array or objects in JSON. BigQuery also supports structures like an array of integers, or an array of dates, but it does not support an array of arrays.

Now that we have our schema file, let's load our data.
``` bash
> bq load \
--source_format=NEWLINE_DELIMITED_JSON \
--noreplace \
--time_partitioning_type=DAY \
--time_partitioning_field created_on \
my_project:my_dataset.my_table \
gs://path/to/blob/in/bucket/file.json \
./my_schema.json
```
The above command loaded `file.json` into the table `my_table` using a schema saved locally in `my_schema.json` and partitioned using the `created_on` field. There's a few things to note:
* The file type is "NEWLINE_DELIMITED_JSON". This means that the JSON source file needs to have a separate record on each line, with each record containing the fields for the table. This is not standard JSON format, and your text editor may yell at you for that if you were to open it up locally. Nonetheless, this is the required JSON format for BigQuery (and for Redshift).
* I specified that the table is partitioned on the `created_on` field. This means that each distinct date in that field will be treated as a distinct partition, improving performance when filtering on that field. Partitions are always optional, but are useful when you know you will be filtering on a particular field (e.g. querying for all products created in the last seven days). If I included `--time_partitioning_type=DAY` but did not provide a field to partition on, BigQuery would have automatically assigned a partition date of the date the record was ingested into BigQuery. We can filter on this automatically generated partition date by using BigQuery's [pseudo-columns](https://cloud.google.com/bigquery/docs/querying-partitioned-tables#limiting_partitions_queried_using_pseudo_columns): `PARTITIONDATE`, and `PARTITIONTIME`.

### `google-cloud-bigquery` Python Library

Let's start by adding the `google-cloud-bigquery` library to our python environment:
``` bash
> pip install google-cloud-bigquery
```

Now let's load our data:
``` python
from google.cloud import bigquery

def load_json_data(table_id: str, source_file_location: str, schema: list) -> None:
    client = bigquery.Client()
    job_config = bigquery.LoadJobConfig(
        schema=schema,
        source_format=bigquery.SourceFormat.NEWLINE_DELIMITED_JSON,
    )
    load_job = client.load_table_from_uri(
        source_file_location,
        table_id,
        job_config=job_config,
    )
    load_job.result()

table_id = 'my_project.my_dataset.my_table'
source_file_location = 'gs://path/to/blob/in/bucket/file.json'
schema = [
    bigquery.SchemaField('product_id', 'INT64', mode='REQUIRED'),
    bigquery.SchemaField('product_name', 'STRING', mode='NULLABLE'),
    bigquery.SchemaField('product_tags'
                        , 'RECORD'
                        , mode='REPEATED'
                        , fields=[
                            bigquery.SchemaField('tag_id', 'INT64', mode='REQUIRED'),
                            bigquery.SchemaField('tag_name', 'STRING', mode='NULLABLE'),
                        ]),
    bigquery.SchemaField('created_on', 'DATE', mode='REQUIRED')
]
load_json_data(table_id, source_file_location, schema)

```
Just like above, we're loading data with a nested field. Unlike the `bq load` command, if you wish to use the BigQuery's Python API to load data into a partitioned table you must create the table with the specified partition first, then load the data.

---

Next Chapter: [Chapter 6: Transforming Your Batch Data](https://github.com/Nunie123/data_engineering_book/blob/master/ch_6_transforming_batch_data.md)<br>