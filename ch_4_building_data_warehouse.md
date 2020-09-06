# Data Engineering in the Cloud: Start to Finish

NOTE: This book is currently incomplete. If you find errors or would like to fill in the gaps, read the **Contributions** section [here](https://github.com/Nunie123/data_engineering_book).

## Table of Contents
[Preface](https://github.com/Nunie123/data_engineering_book) <br>
[Chapter 1: Data Engineering Responsibilities](https://github.com/Nunie123/data_engineering_book/blob/master/ch_1_data_engineering_responsibilities.md) <br>
[Chapter 2: Accessing Data](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md)<br>
[Chapter 3: Moving Data to Your Storage](https://github.com/Nunie123/data_engineering_book/blob/master/ch_3_moving_data_to_storage.md)<br>
**Chapter 4: Building Your Data Warehouse**
[Chapter 5: Getting Data into Your Warehouse](https://github.com/Nunie123/data_engineering_book/blob/master/ch_5_getting_data_into_warehouse.md)<br>
Chapter 6: Transformations for Batch Processing<br>
Chapter 7: Orchestrating Your Pipelines<br>
Chapter 8: Streaming Your Data<br>
Chapter 9: Presenting Data to Your Users<br>
Chapter 10: Infrastructure as Code<br>
Chapter 11: EXAMPLE - Building Complete Date Engineering Infrastructure in AWS<br>
Chapter 12: EXAMPLE - Building Complete Date Engineering Infrastructure in GCP

---

# Chapter 4: Building Your Data Warehouse
Before we can put data into our warehouse, we need to build it. There are many technologies you can use to build a data warehouse, but we're going to focus on AWS Redshift and Google BigQuery. But before we dive into specific implementation, let's talk about Data Warehousing in general.

Data Warehousing is about taking all or most of the data that's important to an organization and putting it in one easily accessible place. Usually, that means putting the data in an Analytical Database (as opposed to a Transactional Database). Analytical Databases are defined by their purpose (to perform analysis), their technological design (e.g. a [columnar database](https://en.wikipedia.org/wiki/Column-oriented_DBMS)), and their data model/schema (discussed below).


## Some Terminology 
We've loosely defined "Data Warehouse" above, but you may have also heard of the terms "Data Lake" and "Data Mart". "Data Lake" generally refers to an unstructured data store where data that may be useful for analytics is stored. The benefit of a Data Lake is that it provides you a central location for all your raw data. However, a Data Lake is limited because its unstructured organization (often just a collection of files) makes it difficult to perform analytics on the data stored within. In [Chapter 3](https://github.com/Nunie123/data_engineering_book/blob/master/ch_3_moving_data_to_storage.md) I discuss how to use AWS S3 and Google Cloud Storage as Data Lakes.

In contrast to a Data Lake, a Data Warehouse stores structured data to facilitate analytics. It's your job as the Data Engineer to transform the unstructured data from the Data Lake into the structured data in the Data Warehouse (discussed in more detail in Chapter 6: Transformations for Batch Processing). A Data Mart is just like a Data Warehouse, except that it usually only includes a portion of the organization's data (e.g. the Marketing Data Mart for a retail company).

In general, I avoid using the terms "Data Lake" and "Data Mart", as they are used differently in different organizations and the concepts are simple enough that I don't find much need for the extra jargon. I do use the term "Data Warehouse", but generally just as a synonym for an Analytical Database. This is just my preference when talking about these concepts, generally. Different organizations may imbue these words with a specific meaning for their particular context, which is great as long as the organization has a shared understanding of their meaning.

Another difficult word in the world of Data Engineering is "model". A "data model" refers to the fields, tables, and relationships used to describe a database schema (discussed in more detail in **Designing a Data Warehouse**, below). However, the term "data science model" is also a thing, referring to the algorithm a Data Scientist has designed to get information out of data. These terms are both widely used, so take care to understand which "model" someone is talking about.

"Schema" is another word that can lead to some confusion. The "schema" of a database generally refers to it's architecture or design. So under this definition an ERD is a visualization of a database's schema. This definition can also be applied to specific tables, where that table's schema refers to the particular fields and other structural implementation details for that table. However, "schema" is also frequently used as the term for a namespace and container under which tables are specified (e.g. `select * from schema_name.table_name`). Redshift, along with many other databases, uses this second definition of "schema", while BigQuery uses the term "dataset" to refer to the same concept.

Finally, we get to "normalization", which is a single word for two distinct tasks that Data Engineers regularly undertake. The first use is to describe database design. The general principle is that tables should have a single purpose and duplicating data across multiple tables should be avoided. We'll talk more about why that's useful, below. The second usage refers to data cleaning. If you have a field called `length` and values of `2 in.`, `2"` and `two inches`, you might normalize the data so that the values all read as `2 in.`.


## Designing a Data Warehouse
Many entire books have been written about designing a Data Warehouse (I can recommend [this one](https://www.google.com/books/edition/The_Data_Warehouse_Toolkit/4rFXzk8wAB8C?hl=en&gbpv=0)). This is also an evolving area for Data Engineers as many of the current generation of data warehousing tools (like Redshift and BigQuery, discussed below) are more powerful than their predecessors and don't rely on Data Engineers to manage previously important details (like indexing and primary keys). Which is to say that the best practices on older systems will not necessarily cary forward to newer systems.

### Goals
We build Data Warehouses because we want our users to be able to pull valuable information out of the data we've [collected](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md) and [transformed](https://github.com/Nunie123/data_engineering_book/blob/master/ch_6_transformations_for_batch_processing.md). While it is paramount that the data be presented in a way that is convenient to the user, it is the Data Engineer's responsibility to impose structure on the data that improves functionality, accuracy, and flexibility.

### Challenges
Growing organizations will generally prioritize in-house analytics before they hire a Data Engineer (which can often be sensible on a tight budget). But that also means that many Data Engineers at younger and smaller organizations are brought on to clean up the analytical databases set up by their analyst predecessors. Even larger and more established companies can have serious data problems, particularly if the different parts of the organization handle their data separately, or the organization's existing Data Warehouse hasn't been properly maintained.

#### Removing Data Silos
"Data Silos" refer to parts of an organization's data that are separated from other parts. For example, an organization might have marketing department that stores data on the campaigns it runs in a Customer Relationship Management system. The organization might also have a Point of Sale vendor that tracks sales data. If the organization wants to answer the question "Did this marketing campaign result in increased sales?" then they'll have to tear down those silos and put their data in a central Warehouse. This doesn't mean that the organization needs to abandon their CRM or POS systems. It just means they need to develop a system to efficiently extract the data from those systems.

Fortunately, most software and service vendors understand their customers need to be able to extract their data from the system. When an organization is evaluating whether to purchase a software product, its the Data Engineer's responsibility to evaluate whether the data from the software product can be reasonably integrated into the Data Warehouse. 

Going back to our goals, removing data silos greatly improves functionality. Analysts will be able to connect previously separate datasets, gaining new insight from existing organizational data.

#### Maintaining a Data Dictionary and ERD
As you are imposing structure on your organization's data you are in a great position to generate and maintain a Data Dictionary and an Entity Relationship Diagram (ERD). These items compose essential documentation for users to get the most value out of your Warehouse. Maintaining this documentation isn't the sexiest task for a Data Engineer, but your users will be glad you took the effort.

Your Warehouse has tables and those tables have fields, and the Data Dictionary defines what data is in each of those fields. You might now the intimate details of each and every field in your warehouse, but your users won't. And if you don't have a Data Dictionary there's a good chance your users will just guess what a field means, potentially leading to inaccurate analyses. As a simple example: suppose you have a `sessions` table describing web sessions on your website which includes the field `customer` to indicate the person who is visiting the site. You also have a `sales` table with the field `customer`. Your marketing team and finance team could each hand a report to the CEO on the same day showing wildly different numbers for "unique customers in the last month" based off the different tables. Neither team would be wrong, but the devil is in the details, and the details live in the Data Dictionary.

While a Data Dictionary defines what the fields mean, an ERD is useful for describing the relationship between the tables in your warehouse. Providing your users with an ERD lets them understand which datasets can be compared and how they are connected. Suppose, for example, you wanted to find the ship date for a particular sale. You can use the ERD to see that the `transactions` tables has the `ship_date` field and there is a many-to-one relationship between the `transactions` table and the `orders` table. A quick glance at the ERD just saved you what could have been a ton of work.

A Data Dictionary can play a huge role in improving the accuracy of your users' reports. An ERD improves functionality by enabling users to discover datasets that can be joined together, while also improving accuracy by showing how tables should be joined.

#### Adapting to Business Needs
A Data Warehouse is not a piece of art that is finely crafted, then hung on a wall to be admired. As new data becomes available, and the needs of your users change, you'll be tasked with adding new tables, modifying existing tables, and deprecating disused tables in your Warehouse. By setting up a thoughtful Data Warehouse architecture you'll be able to easily make modifications with little or no negative impact to your users.

For example, a `sales` table has customer date within the table (e.g. `customer_address`). You create a new `marketing` table that also includes customer data, including `customer address`. Now suppose the marketing team updates the customer address for a particular customer within their CRM, then your amazing data pipeline propagates that update to the `marketing` table. Now you have a single customer with multiple addresses in your system and you'll be lucky to untangle the mess. A better design would be to have a separate `customers` table that is referenced by the `sales` and `marketing` tables. That way the `customer address` field is updated in one place, with no opportunity for conflicting information. This is the concept of [database normalization](https://en.wikipedia.org/wiki/Database_normalization).

By adhering to concepts database [normalization](https://en.wikipedia.org/wiki/Database_normalization) and [dimensional modeling](https://en.wikipedia.org/wiki/Dimensional_modeling) you'll develop a Warehouse that is flexible enough to meet your users' changing needs.

### Deciding on a Design
Phew, that was a lot of background, but now we're ready to design the dang thing. Sketching out your design on a whiteboard, ERD tool ([draw.io](https://app.diagrams.net/), [Lucidchart](https://www.lucidchart.com/pages/)), or a piece of paper is often a good starting point. 

Let's take a simplified example of a grocery store chain. We first brainstorm what tables we need by thinking about people and things the business needs to function:
* Grocery stores sell _food_, so we'll need a `products` table.
* The food is on shelves, so we'll need an `inventory` table.
* They need _physical stores_ to sell the products in, so we'll need a `locations` table.
* _Employees_ stock the shelves: `employees` table.
* _Customers_ buy the food: `customers` table.
* _Vendors_ are selling the food to the grocery stores: `suppliers` table.

Next, let's think about the important actions associated with this business:
* Grocery stores _sell_ food to customers: `sales` table.
* They _purchase_ food from suppliers: `supplier_purchases` table.

Another way to think about it is that these tables answer the question: "What does a grocery store chain do?" 

Answer: A grocery store chain purchases (`supplier_purchases`) food (`products`) from vendors (`suppliers`) and has the employees (`employees`) stock the shelves (`inventory`) of their stores (`locations`) so that customers (`customers`) can buy (`sales`) the food.

This probably won't get us all the tables we need, but it's a great start. We're just using this as an example because people are generally familiar with grocery stores. Your business will likely not be as approachable, so you'll have to work harder to understand the important processes and entities in your organization. Interview your colleagues and be thoughtful. This is not something you want to mess up.

The next step is to go through each table, define what fields will be in the table and how the tables connect to each other. In other words, it's time to make your ERD:

![Grocery Store ERD v1](ch_4_erd_1.png)

By making this diagram it allows us to notice a problem. The products and sales tables don't connect. More concretely, we don't have a way of recording which products were part of which sales to the customer. We should create `sale_items` table that links the `products` table to the `sales` table.

![Grocery Store ERD v2](ch_4_erd_2.png)

Some of you may be thinking: this is all well-and-good, but you've just designed a transactional database. I thought we were going to be designing an analytical database? 

Which brings us to our next step: now that we have tables for the major pieces of the organization mapped out, it's time to start adding tables customized to your users' needs. Of course, the best way to find out what your users need is to talk to them. Consider:
* Do they have any specific suggestions for tables that would be helpful to them? 
* Are there any queries they run on a regular basis that they wish they could easily track historical results from? 
* Are there queries they run that take a long time to run that maybe you could set up as a table or view?
* Are there any initiatives in the company coming up that would benefit from improved data (e.g. a large marketing campaign coming up)?
* Are there any parts of the business that your users wish they had more insight into?

These analytical tables are going to be much harder to pin down than our initial set of ables, and they're going to be much more dependant on your specific organization and the needs or the people in it. For our hypothetical grocery store, let's suppose organizational leadership has decided our focus should be to increase customer retention and repeat business. As a Data Engineer, you might support this initiative by creating a table like this:

![Customer Metrics Table Diagram](ch_4_customer_metrics.png)

The value of this table is very much dependant upon the organization. If this data is only needed infrequently, and the people who need it are SQL whizzes that could write a query to get this from scratch in 5 minutes, then there's not much point having Data Engineers maintain this table in the Warehouse. On the other hand, if this table is used frequently and by users that access the table through drag-and-drop analytics tools, this table could provide real value to a lot of people. 

Another important consideration is consistency. If different analysts use different methods to calculate, say, `most_purchased_product` for a given customer, you could end up with conflicting reports. By having Data Engineering maintain a table with that field the organization now has a single understanding of what the term "most purchased product" means in regards to a customer.

Finally, a table like this could be valuable in describing a customer over time. If this table is updated regularly (daily, weekly, etc.) and the historical data is also maintained, then this table enables time-series analysis on how frequently customers come in and how much they spend.

This was a (very) brief introduction to analytical database design. As mentioned above, you should definitely read up on [database normalization](https://en.wikipedia.org/wiki/Database_normalization) and [dimensional modeling](https://en.wikipedia.org/wiki/Dimensional_modeling) when designing a warehouse for your organization.

## Making Tables in AWS Redshift
Now that you've got your Data Warehouse design figured out it's time to buckle down and actually build the thing. 

### Setting Up Your Cluster
The first thing you'll need to do is set up a Redshift cluster. A Redshift cluster has one or more virtual machines (called "nodes") that handle data processing and some storage. AWS recommends using either their DC2 nodes, or their RA3 nodes. DC2 nodes are a bit cheaper, but the space on each node is limited, so if you need a lot of storage you'll need to buy another node, even if you don't need the additional compute power. RA3 nodes have unlimited storage (it automatically offloads data to S3 once the SSDs fille up), but are also more expensive. AWS recommends using DC2 nodes if you expect working with less than a TB of data, otherwise use the RA3 nodes.

AWS charges Redshift users by the node, making it easy to scale out your infrastructure by adding additional nodes to your redshift cluster. The cheapest Redshift cluster you can run right now is a single node dc2.large, which will cost you around $0.25/hour (~$180/month). The cheapest RA3 node is the ra3.4xlarge, which will cost you around $3.26/hour (~$2,347.20/month). People using RA3 will additionally billed for any S# storage they use at $0.024 per GB per month. You can get reduced rates on your cluster costs if you're willing to commit to a year or more at a particular minimum cluster size. You can see more details on pricing [here](https://aws.amazon.com/redshift/pricing/).

Finally, before you can set up a cluster you'll need to make sure you have [permissions configured](https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-create-an-iam-role.html) to allow you to do so.

Below I'll discuss how to create a cluster in the `aws` CLI and Python. In **Chapter 10** I'll discuss how manage your AWS infrastructure in your code using tools like [Terraform](https://www.terraform.io/).

#### `aws` Command Line Tool
As discussed in [Chapter 2](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md), to use the `aws` command line tool you must have it [downloaded](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) and [configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html).

Previously we used the `aws s3` command to interact with S3, so it should be no surprise we'll be using `aws redshift` to interact with Redshift. The `create-cluster` command has [lots of options](https://docs.aws.amazon.com/cli/latest/reference/redshift/create-cluster.html), but the required options are:
* node-type: designate what type of DC2 or RA3 node you would like to use (discussed above).
* master-username: username for the master user account for the cluster.
* master-user-password: password for the master user account.
* cluster-identifier: a unique name for the cluster. This will be used to identify the cluster for other commands.

```bash
> aws redshift create-cluster --node-type dc2.large --number-of-nodes 2 --master-username my_username --master-user-password my_password --cluster-identifier my_cluster
```

#### Python
You must first [install](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#installation) and [configure](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#configuration) `boto3`, as discussed in more detail in [Chapter 2](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md).

``` python
import boto3

client = boto3.client('redshift')
options = {
    NodeType: 'dc2.large',
    NumberOfNodes: 2,
    MasterUsername: 'my_username',
    MasterUserPassword: 'my_password',
    ClusterIdentifier: 'my_cluster'
}
client.create_cluster(options)
```

### Building Your Tables
You've got your Redshift cluster, now let's fill it up with some databases, schemas, and tables. Your Redshift cluster can have multiple databases on it, but you cannot query across the databases. One potential use-case for having multiple databases is to create a development and production environment on the same cluster. Having these environments on the same cluster can be cheaper and a little more convenient, but comes with a serious drawback: your production and development systems will be sharing the same compute power. By sharing the compute power you have the potential to cripple the performance of your production environment because you are testing code in your development environment.vThis may be acceptable to you when you're first starting to build out your infrastructure, but sharing infrastructure between dev and prod is done at your own risk.

"Schemas" are essentially name-space sub-division of databases for organizing your tables. Unlike databases in Redshift, you can query across schemas. One strategy is to create schemas based on topics, with each schema containing related tables. For example, the `products` schema might include the tables `product_details`, `product_prices`, and `product_inventory`. Another option is to use schemas to help organize how data is ingested, transformed, and presented within the database. For example, the `landing` schema could be where all raw data from the various sources is dumped, the `operational_data_store` schema could be where data goes after it is cleaned and/or deduped, and the `public` schema could be where tables intended for use by your users are maintained.

#### `aws` Command Line Tool
The `aws` CLI does not include commands for creating tables. You can create tables using the AWS console, or through the use of a SQL client. The next section discusses connecting to Redshift through Python's `psycopg2` client.

#### `psycopg2` and `sqlalchemy-redshift` Python Library
Redshift is based off of the PostgreSQL database. One of the benefits of that legacy is that we can use a PostgreSQL driver to connect to Redshift: `psycopg2`. In [Chapter 2](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md) we discussed how to use the SQLAlchemy python library to connect to a PostgreSQL database. Because Redshift is not natively supported by SQLAlchemy, instead we will be using [sqlalchemy-redshift](https://pypi.org/project/sqlalchemy-redshift/), which will allow us to use the familiar syntax of SQLAlchemy to connect with the Redshift cluster.

Just like when connecting to a PostgreSQL database, you'll need to know:
* username
* password
* host
* port (usually 5439)
* database name ("dev" by default)

You can look up these connection details in the AWS console. The master username and password you established when creating the cluster should not be the credentials used as part of you data pipeline. The account used to create tables (or query tables, etc.) should only have enough permissions to do the required tasks, and no more. To handle this you should [create a user](https://docs.aws.amazon.com/redshift/latest/dg/r_CREATE_USER.html) and then [grant](https://docs.aws.amazon.com/redshift/latest/dg/r_GRANT.html) the user the required permissions. 

You'll first need to install `psycopg2` and `sqlalchemy-redshift` into your environment:
``` bash
> pip install psycopg2
> pip install sqlalchemy-redshift
```

Now you can import `sqlalchemy` like normal:
``` python
import json
from sqlalchemy import create_engine

def execute_raw_sql(connection_string: str, raw_sql: str) -> None:
    engine = create_engine(connection_string)
    with engine.connect() as connection:    # using `with` here lets us not worry about closing the connection (i.e. `connection.close()`)
        connection.execute(raw_sql)


connection_string = 'redshift+psycopg2://my_username:my_password@abc123.amazonaws.com:5439/my_database'

# create a table specifying column names, their data types, and other options
raw_sql_1 = ('CREATE TABLE users ( '
            'user_id    INT         IDENTITY '
            'username   VARCHAR     NOT NULL '
            'email      VARCHAR     NULL'
            )
execute_raw_sql(connection_string, raw_sql_1)

# create table with structure and data generated from a query
raw_sql_2 = ('CREATE TABLE AS users_with_email '
            'select username, email '
            'from users '
            'where email is not null'
            )
execute_raw_sql(connection_string, raw_sql_2)

# create a table with the same structure as an existing table
raw_sql_3 = 'CREATE TABLE users2 LIKE users'
execute_raw_sql(connection_string, raw_sql_3)
```

More details on the syntax for creating tables is available [here](https://docs.aws.amazon.com/redshift/latest/dg/r_CREATE_TABLE_NEW.html).

## Making Tables in Google BigQuery

### Setting Up Your BigQuery Instance
Unlike Redshift, starting to use BigQuery doesn't involve provisioning specific compute resources. BigQuery is a fully managed service, and all you need to do to start using it is to [create a project](https://cloud.google.com/resource-manager/docs/creating-managing-projects) (which is required for using any GCP services).

The [pricing model for BigQuery](https://cloud.google.com/bigquery/pricing) is also completely different than Redshift. BigQuery automatically scales, providing whatever storage and processing power is needed to accomplish the tasks you send to it. So instead of paying per provisioned machine, you pay per byte stored and byte processed. You'll pay around $0.02 per GB per month for storage, and $5.00 per TB per month for processing queries. Keep in mind that while operations like loading and copying data is free, all queries performed against that data are charged.

### Building Your Tables
BigQuery has the concept of "datasets", which are the equivalent of "schemas" in Redshift. Essentially they're name-spaces that are convenient for organizing your tables, but there is no impediment to querying tables across datasets.

#### `bq` Command Line Tool
While the `gsutil` command line tool is used for interacting with Google Cloud Storage, the `bq` command line tool is used for interacting with BigQuery. And unlike Redshift, whose command line tool has limited capability, the `bq` command is a powerful and fully featured tool for interacting with BigQuery.

In [Chapter 2](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md) I explained how to set up the Google Cloud SDK on your machine in order to use the `gsutil` tool. If you completed that then you're already set to use the `bq` tool. If not, then take a look at [Chapter 2](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md) or review the instructions [here](https://cloud.google.com/sdk/docs).

Now that `bq` is working on your machine, it's time to [create some "datasets"](https://cloud.google.com/bigquery/docs/datasets). As mentioned above, in BigQuery "datasets" are namespaces in which tables reside, similar the "schemas" in Redshift.

Here we make a dataset called "my_dataset" inside the project "my_project". Note that you can query tables cross-dataset, but tables in different projects are isolated from each other. In **Chapter 10** we'll discuss how to manage datasets through a deployment pipeline.
``` bash
> bq mk --dataset my_project:my_dataset
```

There's one more piece we need before we can build our table: defining our table's schema. This can be accomplished by defining the schema in-line or by reference to a JSON file. For any tables that have more than a couple fields you'll probably want to make a file.

__my_schema.json__:
``` json
[
    {
        "description": "The Unique ID for the user.",
        "mode": "REQUIRED",
        "name": "user_name",
        "type": "STRING"
    },
    {
        "description": "The count of of all items purchased by this user.",
        "mode": "NULLABLE",
        "name": "items_purchased",
        "type": "INT64"
    },
    {
        "description": "The sum of all payments made to the organization by the user in USD.",
        "mode": "NULLABLE",
        "name": "dollars_spent",
        "type": "FLOAT64"
    }
]
```

Now we are finally ready to make our table:
``` bash
> bq mk --table my_project:my_dataset.my_table my_schema.json
```

Defining the schema in-line:
``` bash
> bq mk --table my_project:my_dataset.my_table user_name:STRING,items_purchased:INT64,dollars_spent:FLOAT64
```

You can also create a table based on a query from another table:
``` bash
> bq query --destination_table my_project:my_dataset.my_table \
--use_legacy_sql=false \
'select user_name, items_purchased, dollars_spent from user_dataset.user_activity_table'
```
By creating a table from a query the new table will have the field definitions of the fields in the existing table, so there's no need to define a schema.

Finally, you can create a table using the `load` command:
``` bash
bq --headless load \
--source_format=NEWLINE_DELIMITED_JSON \
--replace \
my_project:my_dataset.my_table \
gs://path/to/blob/in/bucket/file.json \\
my_schema.json
```

We'll be talking more about the `load` command in **Chapter 5**. You can read the documentation for creating tables [here](https://cloud.google.com/bigquery/docs/tables#bq).

#### `google-cloud-bigquery` Python Library
Let's start by adding the `google-cloud-bigquery` library to our python library:
``` bash
pip install google-cloud-bigquery
```

Now we can create our datasets and tables:
``` python
from google.cloud import bigquery

def create_dataset(dataset_name: str, project_name: str) -> None:
    dataset_id = f'{project_name}.{dataset_name}'
    dataset_obj = bigquery.Dataset(dataset_id)
    client = bigquery.Client()
    dataset = client.create_dataset(dataset)

def create_table(table_name: str, dataset_name: str, project_name: str, schema: list) -> None:
    table_id = f'{project_name}.{dataset_name}.{table_name}'
    table_obj = bigquery.Table(table_id, schema)
    client = bigquery.Client()
    table = client.create_table(table)

project_name = 'my_project'
dataset_name = 'my_dataset'
table_name = 'my_table'
schema = [
    bigquery.SchemaField('user_name', 'STRING', mode='REQUIRED'),
    bigquery.SchemaField('items_purchased', 'INT64', mode='REQUIRED'),
    bigquery.SchemaField('dollars_spent', 'FLOAT64', mode='REQUIRED')
]

create_dataset(dataset_name, project_name)
create_table(table_name, dataset_name, project_name)
```

We can also create a table from a query:
``` python
from google.cloud import bigquery

def create_table_from_query(table_name: str, dataset_name: str, project_name: str, raw_sql: str) -> None:
    table_id = f'{project_name}.{dataset_name}.{table_name}'
    job_config = bigquery.QueryJobConfig(destination=table_id)
    query_job = client.query(raw_sql, job_config=job_config)
    query_job.result()

project_name = 'my_project'
dataset_name = 'my_dataset'
table_name = 'my_table'
raw_sql = 'select user_name, items_purchased, dollars_spent from user_dataset.user_activity_table'
create_table_from_query(table_name, dataset_name, project_name, raw_sql)
```
You can find the fool documentation for the Python API [here](https://googleapis.dev/python/bigquery/latest/index.html).

---

Next Chapter: [Chapter 5: Getting Data into Your Warehouse](https://github.com/Nunie123/data_engineering_book/blob/master/ch_5_getting_data_into_warehouse.md)

