# Data Engineering in the Cloud: Start to Finish

NOTE: This book is currently incomplete. If you find errors or would like to fill in the gaps, read the **Contributions** section [here](https://github.com/Nunie123/data_engineering_book).

## Table of Contents
[Preface](https://github.com/Nunie123/data_engineering_book) <br>
[Chapter 1: Data Engineering Responsibilities](https://github.com/Nunie123/data_engineering_book/blob/master/ch_1_data_engineering_responsibilities.md) <br>
[Chapter 2: Accessing Data](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md)<br>
[Chapter 3: Moving Data to Your Storage](https://github.com/Nunie123/data_engineering_book/blob/master/ch_3_moving_data_to_storage.md)<br>
[Chapter 4: Building Your Data Warehouse](https://github.com/Nunie123/data_engineering_book/blob/master/ch_4_building_data_warehouse.md)<br>
**Chapter 5: Getting Data into Your Warehouse**<br>
Chapter 6: Transformations for Batch Processing<br>
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

## Loading Data into AWS Redshift

### `aws` Command Line Tool
As mentioned in [Chapter 4](https://github.com/Nunie123/data_engineering_book/blob/master/ch_4_building_data_warehouse.md), the `aws redshift` commands are limited, mostly focusing on creating and maintaining Redshift clusters. Since we can't load out data using the command line, we'll focus on the `psycopg2` and `sqlalchemy-redshift` Python libraries, explained below.

### `psycopg2` and `sqlalchemy-redshift` Python Library


## Loading Data into Google BigQuery

### `bq` Command Line Tool

### `google-cloud-bigquery` Python Library