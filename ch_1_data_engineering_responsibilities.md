# Data Engineering in the Cloud: Start to Finish

NOTE: This book is currently incomplete. If you find errors or would like to fill in the gaps, read the **Contributions** section [here](https://github.com/Nunie123/data_engineering_book).

## Table of Contents
[Preface](https://github.com/Nunie123/data_engineering_book) <br>
**Chapter 1: Data Engineering Responsibilities** <br>
[Chapter 2: Accessing Data](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md)<br>
[Chapter 3: Moving Data to Your Storage](https://github.com/Nunie123/data_engineering_book/blob/master/ch_3_moving_data_to_storage.md)<br>
[Chapter 4: Building Your Data Warehouse](https://github.com/Nunie123/data_engineering_book/blob/master/ch_4_building_data_warehouse.md)<br>
[Chapter 5: Getting Data into Your Warehouse](https://github.com/Nunie123/data_engineering_book/blob/master/ch_5_getting_data_into_warehouse.md)<br>
Chapter 6: Transformations for Batch Processing<br>
Chapter 7: Orchestrating Your Batch Pipelines<br>
Chapter 8: Streaming Your Data<br>
Chapter 9: Presenting Data to Your Users<br>
Chapter 10: Infrastructure as Code<br>
Chapter 11: EXAMPLE - Building Complete Date Engineering Infrastructure in AWS<br>
Chapter 12: EXAMPLE - Building Complete Date Engineering Infrastructure in GCP

---

# Chapter 1: Data Engineering Responsibilities

I promised you a comprehensive book on setting up and maintaining Data Engineering infrastructure, so the first step is explaining what that means. This chapter explains what a Data Engineer typically does. There's a huge variety in the responsibilities for any particular Data Engineering position, so this chapter will help set the scope of what I consider Data Engineering work, and thus the scope of what is covered in this book.

If you don't need this background and want to dive into setting up cloud infrastructure, then skip ahead to [Chapter 2: Accessing Data](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md).

In short, a Data Engineer is responsible for two things:
1. Data Pipelines
2. Data Warehouses

## Data Pipelines
A data pipeline is about getting the data you want into the shape and location you want it in. Generally, this involves extracting the data from a source system, transforming the data to be more approachable for your users, then loading the data into your warehouse. This process of Extracting, Transforming, and Loading is often abbreviated as "ETL" (note that it's also common to Extract, Load, then Transform data. You won't be surprised to learn this is often abbreviates ELT). Your ETL/ELT process is your data pipeline, so let's talk a little about each, in turn.

### Extract
If the data was already in your warehouse, then your company wouldn't need you. It's more common for a company to have data that's distributed, fragmented, and siloed. When developing technology products, it's often the case that making the data used in the product available for analytics is an afterthought, if it's thought about at all. Then one day the company executives realize they're essentially throwing away one of their most valuable resources, so they hire you to make all that wonderful data available to them.

Source data will come from a wide variety or sources. Some examples include:
* Internal transactional databases
* 3rd-party web APIs
* AWS S3 buckets
* Flat files on the company intranet

In addition to a variety of sources, you will likely encounter a variety of file types, including:
* JSON blobs
* Newline delimited JSON (also called JSON Lines)
* CSV
* TSV
* Parquet

A Data Engineer's job is to automate and operationalize their data pipelines. A dependency of these pipelines is the source data being where you expect it in the format you expect it to be in. You will experience frustration when a source file is missing or malformed, and your pipeline breaks. To handle this you need to do two things:
1. To the extent possible, extract guarantees from the providers of the source files detailing how and when the files will be provided.
2. Build appropriate fault tolerance into your extract processes. Understand when it is ok for a source file is missing, and when you need to wake someone up in the middle of the night because a source file is missing. Set your alerts as needed for each pipeline.

If possible, build rapport with the people that maintain your source files. There's a good chance you'll be working together to solve a problem in the future. Many a Data Engineer has had a stressful day because the managers of an upstream system didn't think to warn them of a schema change or service outage. Establishing relationships helps reduce that problem.

### Transform
Information is organized data. It's your job to make sure that analysts can present information from the data you pulled out of a data source. The way you make sure that data is useful to them is through transformations.

As a Data Engineer you are organizing data for use in analytics. Usually the source systems generating the data have given absolutely no thought to analytics when they have their systems generate the data. Sometimes the transformations will be minor, like converting strings to integers. Sometimes you'll have to do something like convert JSON blobs into a series of normalized tables at an aggregation level different than the source schema. You need to be ready for anything.

The key here is understanding your source data, understanding what your users need, and then figuring our how to get from one to the other. Based on your user's requirements you'll build the schema of your Data Warehouse, then it is your responsibility to ensure the data neatly fits into that schema.

You'll also need to decide what to do when the source data doesn't neatly fit into your Warehouse (e.g. what do you do if you are trying to fit "08/15/985" into a date field).

### Load
The last piece of the puzzle is bringing your data into your Warehouse. If you've got your transformation right, this should be a piece of cake. The trick here is to handle the merging of new data with existing data. Think about whether you are replacing the existing data, appending to the existing data, or applying business logic to decide which records to keep and which to discard.

This step will be a little more complicated if you are doing an Extract, Load, Transform (ELT) pipeline. Because the data hasn't been transformed yet you'll have to verify that the source data will fit into your Warehouse. This can be problematic when the data is in a format not supported by your Data Warehouse solution (e.g. your Data Warehouse might not easily support nested JSON files). In this case, even if your general process is ELT, you may need to perform some minor transformations to the data to ensure it loads into your Data Warehouse solution.

### ETL vs ELT
Why would you choose ELT over ETL, or vice-versa? They are both good options, so making the decision of which to choose comes down to your specific use-case and personal preference. ETL is nice because you don't have to worry about the source file being compatible with your Data Warehouse solution. The transformations are done right away, often by a tool like Spark or Pandas, and the data is in it's final form when you start uploading it to your warehouse.

However, an ELT is nice because you can rely on the Warehouse system to process your transformations for you, instead of worrying about adding additional infrastructure (like spinning up a Spark cluster) to handle the transformations.

If most of your source files fit nicely into your Warehouse without transformation (e.g the data is coming from relational database dumps and CSV files) then ELT might be a good fit.

If your source files are heavily nested JSON files and your Warehouse does not have good options for dealing with JSON files, then ETL is likely a good choice.

Another highly important factor is: what technologies are you most familiar with. If your Data Engineering team has a lot of experience with Spark, and only a little experience with AWS Redshift, then it makes sense to leverage your Spark expertise and adopt an ETL model.

## Data Warehousing
The terms "Data Lake", "Data Warehouse", and "Data Mart" are used differently for different organizations. My definitions are as follows:
* Data Lake: a collection of all the data within an organization with no schema enforced across the various pieces of data.
* Data Warehouse: a collection of all the data within an organization that is being made available for analytics. A schema is enforced across all pieces of data.
* Data Mart: like a Data Warehouse, but focusing on a subset of the data available for analysis within an organization.

I'll being using "Data Warehouse" as a general term for a structured data store that allows Data Analysts and Data Scientists to gain insight from company data.

The most important thing to consider when designing your Data Warehouse is: what data structures will provide the most value for my users?

Answering that question should involve lots of conversations with the data analysts and data scientists within your organization. Also keep in mind that the best Data Warehouse structure will be a moving target. Your users will have evolving needs, and you will need to evolve the Data Warehouse to meet those needs.

More detail on the best practices for designing your Data Warehouse are available in: [Chapter 4: Building Your Data Warehouse](https://github.com/Nunie123/data_engineering_book/blob/master/ch_4_building_data_warehouse.md)


---

Next Chapter: [Chapter 2: Accessing Data](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md)