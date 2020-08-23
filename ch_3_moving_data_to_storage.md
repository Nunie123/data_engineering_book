# Data Engineering in the Cloud: Start to Finish

NOTE: This book is currently incomplete. If you find errors or would like to fill in the gaps, read the **Contributions** section [here](https://github.com/Nunie123/data_engineering_book).

## Table of Contents
[Preface](https://github.com/Nunie123/data_engineering_book) <br>
[Chapter 1: Data Engineering Responsibilities](https://github.com/Nunie123/data_engineering_book/blob/master/ch_1_data_engineering_responsibilities.md) <br>
[Chapter 2: Accessing Data](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md)<br>
**Chapter 3: Moving Data to Your Storage**<br>
[Chapter 4: Building Your Data Warehouse](https://github.com/Nunie123/data_engineering_book/blob/master/ch_4_building_data_warehouse.md)<br>
Chapter 5: Getting Data into Your Warehouse<br>
Chapter 6: Transformations for Batch Processing<br>
Chapter 7: Orchestrating Your Pipelines<br>
Chapter 8: Streaming Your Data<br>
Chapter 9: Presenting Data to Your Users<br>
Chapter 10: Wrapping up

---

## Chapter 3: Moving Data to Your Storage
So you've got your data from your various sources, but where are your going to put it? Ultimately you're going to want it available for your users in your data warehouse. But by the time it's in the warehouse you've probably run some transformations on it. But what happens if there's a problem with one of your transformations? What if you've been rounding a `tax` field to two decimal places for a year and your users suddenly announce they need it rounded to four decimal places for all historical data? It's for those sorts of situations where it's good to have all of your source files saved in their original format.

Fortunately, AWS and GCP provide some cheap storage solutions that allow you to save your source files without modification. Once your source files are in your cloud storage you can worry about bringing the data into your warehouse.

### AWS
In [Chapter 2](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md) we talked about moving data out of S3. Here we're going to be using the same tools to put data in: the `aws` command line tool and the `boto3` Python library.

When using S3, AWS will charge you based on the amount of storage you are using and the amount of data be transferred out of S3. As we'll discuss in Chapter 5, we'll be transferring data from S3 to Redshift, which is free. Storage costs are tiered and vary by region, but for example, storage in us-east-1 costs between $0.21 and $0.23 per GB per month right now. Storing 20 TB of data in S3 would cost you about $471/month in storage fees. AWS offers cheaper storage options for data that is accessed infrequently and/or you are ok with the data being slow to retrieve. You can visit AWS's S3 pricing page [here](https://aws.amazon.com/s3/pricing/) and their cost calculator [here](https://calculator.aws/#/createCalculator).

#### `aws` Command Line Tool
As discussed in [Chapter 2](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md), to use the `aws` command line tool you must have it [downloaded](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) and [configured](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html).

So now you are looking to put your raw files in S3, but you still need to designate a home for them. This home is called a "bucket" which you can think of as a top-level directory for S3. You'll first need to create a bucket, then you can use the `cp`, `mv`, and `sync` commands to move your file to your designated bucket. 

Note that while you can create sub-directories within a bucket to organize your files, you cannot grant permissions at the sub-directory level, only at the bucket and object level. If you have a group of files that you want to have more restrictive access for (e.g. because they contain Personally Identifiable Information), it's generally best practice to put these files in a separate bucket. It is a reasonable strategy to create a separate bucket for each data source you are getting files from. AWS S3 charges you for storing objects and transferring objects in and out, but does not charge extra for creating or using multiple buckets.

To create a bucket you must provide the bucket name and the [region](https://docs.aws.amazon.com/general/latest/gr/s3.html):

``` bash
> aws s3api create-bucket --bucket my-bucket --region us-west-1 --create-bucket-configuration LocationConstraint=us-west-1
```

[Copy](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/cp.html) the file "my_file.txt" to "my-bucket":

``` bash
> aws s3 cp my_file.txt s3://my-bucket
```

[Move](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/mv.html) the contents of the "images" directory to the "static" sub-directory in "my-bucket":

``` bash
> aws s3 mv ~/images/ s3://my-bucket/static/
```

We saw in [Chapter 2](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md) that we can [sync](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3/sync.html) a local file to match a bucket in S3. The reverse is also possible, having an S3 bucket that automatically matches the files in a local directory:

``` bash
> aws s3 sync ~/scans/ s3://my-bucket/
```

#### `boto3` Python Library
You must first [install](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#installation) and [configure](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/quickstart.html#configuration) `boto3`, as discussed in more detail in [Chapter 2](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md).

`boto3` offers an array of commands to interact with S3, but we'll be focusing on [create_bucket()](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Client.create_bucket), [upload_file()](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Client.upload_file), and [copy](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Client.copy).

``` python
import boto3

def create_bucket(bucket_name: str, region: str) -> None:
    """
    This function creates a bucket with the specified name in the specified region. The bucket owner will be set by the default credentials.
    """
    s3_client = boto3.client('s3')
    s3_client.create_bucket(
        Bucket=bucket_name,
        CreateBucketConfiguration={
            'LocationConstraint': region
        }
    )

def upload_local_file_to_s3(source_path: str, bucket_name: str, key: str) -> None:
    """
    This function uploads a local file to an S3 bucket. Note that the "key" parameter is the destination file name and should also include sub-directories, if applicable (e.g. "my/folder/file.txt"), but not the name of the bucket. So to upload a file to s3://my-bucket/images/beach.jpg the "key" would be "images/beach.jpg".
    """
    s3_client = boto3.client('s3')
    s3_client.download_file(source_path, bucket_name, key)

def copy_file_between_s3_buckets(source_bucket_name: str, source_file_name: str, dest_bucket_name: str, dest_file_name: str) -> None:
    """
    This function copies a file from one S3 location to another. Note that the "*_file_name" parameters should also include sub-directories, if applicable (e.g. "my/folder/file.txt"), but not the name of the bucket.
    """
    s3_client = boto3.client('s3')
    source_file = {
        'Bucket': source_bucket_name,
        'Key': source_file_name
    }
    s3_client.copy(source_file, dest_bucket_name, dest_file_name)

create_bucket('my-first-bucket', 'us-east-2')
upload_local_file_to_s3('~/Documents/memo.md', 'my-first-bucket', 'memo1.md')
create_bucket('my-second-bucket', 'us-east-2')
copy_file_between_s3_buckets('my-first-bucket', 'memo1.md', 'my-second-bucket', 'docs/memo.txt')

```

### GCP
Below I go through using the `gsutil` command line tool and `google.cloud.storage` Python library to save data in GCS. If you're familiar with AWS S3 you'll notice the functionality is quite similar. 

#### `gsutil` Command Line Tool
As discussed in [Chapter 2: Accessing Data](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md), you'll have to [install and configure](https://cloud.google.com/storage/docs/gsutil_install) `gsutil` to start.

We'll focus on the four commands you'll most commonly be using for storing data in GCS: [mb](https://cloud.google.com/storage/docs/gsutil/commands/mb), [cp](https://cloud.google.com/storage/docs/gsutil/commands/cp), [mv](https://cloud.google.com/storage/docs/gsutil/commands/mv), and [rsync](https://cloud.google.com/storage/docs/gsutil/commands/rsync).

Create "my-bucket":

``` bash
mb gs://my-bucket
```

Copy the contents of the "files" directory to "my-bucket" bucket:

``` bash
> gsutil cp ./files gs://my-bucket/*
```

Move all .csv files from the current working directory to "my-bucket" bucket inside the "files" sub-folder.

``` bash
> gsutil cp ./*.csv gs://my-bucket/files/
```

The rsync command is analogous to the `aws s3 sync` command, discussed above. Use `rsync` to keep a bucket in sync with a local directory:


``` bash
> gsutil rsync . gs://my-bucket
```
Check out the `gsutil` [documentation](https://cloud.google.com/storage/docs/gsutil) for more detail.

#### `google.cloud.storage` Python Library
First things first: make sure you have `google-cloud-package` [installed and configured](https://cloud.google.com/storage/docs/reference/libraries#client-libraries-install-python).

In this section we'll look at [create_bucket()](https://googleapis.dev/python/storage/latest/client.html#google.cloud.storage.client.Client.create_bucket) method from the Client object, the [copy_blob()](https://googleapis.dev/python/storage/latest/buckets.html#google.cloud.storage.bucket.Bucket.copy_blob) method from the Bucket object, and the [upload_from_filename()](https://googleapis.dev/python/storage/latest/blobs.html#google.cloud.storage.blob.Blob.upload_from_filename) method from the Blob object.

``` python
from google.cloud import storage

def create_bucket(bucket_name: str) -> None:
    """
    This function creates a bucket with the provided bucket name. The project and location are set to default.
    """
    client = storage.Client()
    client.create_bucket(bucket_name)

def upload_file_to_bucket(source_filepath: str, bucket_name: str, blob_name: str) -> None:
    """
    This function uploads a file to a bucket. The blob_name will be the name of the file in GCS.
    """
    client = storage.Client()
    bucket = client.bucket(bucket_name)
    blob = bucket.blob(blob_name)
    
    blob.upload_from_filename(source_filename)

def copy_blob_to_new_bucket(source_bucket_name: str, blob_name: str, dest_bucket_name: str) -> None:
    """
    This function copies a file from one bucket to another.
    """
    client = storage.Client()
    source_bucket = client.bucket(source_bucket_name)
    source_blob = source_bucket.blob(source_blob_name)
    destination_bucket = client.bucket(dest_bucket_name)

    source_bucket.copy_blob(source_blob, destination_bucket)

create_bucket('test_bucket')
upload_file_to_bucket('~/files/my_file.md', 'test_bucket', 'my_file.txt')
copy_blob_to_new_bucket('test_bucket', 'my_file.txt', 'prod_bucket')

```
Documentation for the Python storage library is [here](https://googleapis.dev/python/storage/latest/client.html).

---

Next Chapter: [Chapter 4: Building Your Data Warehouse](https://github.com/Nunie123/data_engineering_book/blob/master/ch_4_building_data_warehouse.md)