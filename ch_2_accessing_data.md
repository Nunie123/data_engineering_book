# Online Data Engineering: Start to Finish

NOTE: This book is currently incomplete. If you find errors or would like to fill in the gaps, read the **Contributions** section below.

## Table of Contents
[Preface](https://github.com/Nunie123/data_engineering_book)
[Chapter 1: Data Engineering Responsibilities](https://github.com/Nunie123/data_engineering_book/blob/master/ch_1_data_engineering_responsibilities.md) <br>
**Chapter 2: Accessing Data**<br>
Chapter 3: Moving Data to Your Storage<br>
Chapter 4: Building Your Data Warehouse<br>
Chapter 5: Getting Data into Your Warehouse<br>
Chapter 6: Transformations for Batch Processing<br>
Chapter 7: Orchestrating Your Pipelines<br>
Chapter 8: Streaming Your Data<br>
Chapter 9: Presenting Data to Your Users<br>
Chapter 10: Wrapping up

---

## Chapter 2: Accessing Data

### Overview

A big part of most Data Engineers' jobs is to take data from one place and putting it somewhere else. This chapter is going to talk about how you are going to be taking the data.

### Cloud Storage Services
One of the most common places for data to be stored is on cloud storage service. The most common varieties include AWS Simple Storage Solution (S3), Google Cloud Storage (GCS), and Azure Storage. 

While there is a good chance you won't be building your Data Engineering infrastructure using multiple cloud providers, there's still a good chance you'll have to get data out of more than one cloud storage service. That's the thing about accessing data, usually someone else has control of it and you've got to go to them, wherever and however they're storing it. 

The nice things about getting data out of cloud storage providers are:
* Good documentation
* Good tooling to support automation
* You generally don't have to worry about format or schema to move the data.

While all of these storage services include a web console to perform tasks, we are not going to discuss it here. As a Data Engineer you are focused on automating tasks, setting up an alert to let you know if it breaks, then forgetting about it. That's why we'll be discussing the CLI (Command Line Interface) tools and Python libraries supported by these storage services.

#### AWS S3
S3 allows you to create repositories of files. The repositories are called "buckets" and the files are called "objects". You can create sub-folders within a bucket, though they behave a little differently than directories in a file system. We could got a lot deeper into s3, but this is enough to get us going.

_CLI_

AWS provides CLI tool for download called "aws". The `aws` CLI is available to download from your favorite package manager ([Homebrew](https://brew.sh/), [APT](https://help.ubuntu.com/community/AptGet/Howto), etc.), or you can follow the instructions from AWS [here](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

Once you install the `aws` CLI you will also need to [configure](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) it so that it has access to your (hopefully) secured S3 bucket. For simple set-ups just run `aws configure`.

The CLI has many commands for many different services, but since this about getting data out we will focus on the [copy](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-s3-commands.html#using-s3-commands-managing-objects-copy), [move](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-s3-commands.html#using-s3-commands-managing-objects-move), and [sync](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-s3-commands.html#using-s3-commands-managing-objects-sync) commands for S3.

Fortunately, "cp" and "mv" behave just as you would expect. You can copy or move files from S3 to local, or between buckets (make sure your credentials allow you to access both buckets). 

Copy everything from "my_folder" in the "my-bucket" bucket into the current working directory.
`aws s3 cp s3://my-bucket/my_folder ./`

Move the file "iceCream.txt" from "cool-bucket" to "cooler-bucket".
`aws s3 mv s3://cool-bucket/iceCream.txt s3://cooler-bucket/`

The `sync` command you may not be as familiar with, but it's nonetheless pretty intuitive. By syncing an S3 path to a local path any changes to the files in the S3 path will also applied to the local path. For example, you can ensure that any files added to an S3 bucket are also added to your local directory.

Sync files from bucket "coolest-bucket" with current directory.
`aws s3 sync s3://coolest-bucket .`

There are a huge number of options and capabilities for these tools, so it's definitely worth reading the [documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-services-s3-commands.html).

_PYTHON_

AWS also provides a Python library, [boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html) which can be used for accessing S# (among many other functions). It can be installed with `pip`, `pip3`, or however you install you python packages. You can configure by running `aws configure`, or you can provide you credentials from within your code, as shown below.

This example shows establishing a boto3 s3 client using credentials stored as environment variables.
```Python
import boto3

s3_client = boto3.client(
        's3',
        aws_access_key_id=os.getenv('AWS_ACCESS_KEY_ID'),
        aws_secret_access_key=os.getenv('AWS_SECRET_ACCESS_KEY_ID')
    )
```

The S3 client from boto3 provides [many methods](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html), but the ones we can focus on for accessing data are [download_file](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Client.download_file) and [copy](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html#S3.Client.copy)

```Python
import boto3

def copy_s3_file_to_local(bucket_name: str, file_name: str, destination_path: str) -> None:
    """
    This function downloads a file in an S3 bucket to a local directory. Note that the "file_name" parameter should
    also include sub-directories, if applicable (e.g. "my/folder/file.txt"), but not the name of the bucket.
    """
    s3_client = boto3.client('s3')
    s3_client.download_file(bucket_name, file_name, destination)

def copy_file_between_s3_buckets(source_bucket_name: str, source_file_name: str, dest_bucket_name: str, dest_file_name: str) -> None:
    """
    This function copies a file from one S3 location to another. Note that the "*_file_name" parameters should
    also include sub-directories, if applicable (e.g. "my/folder/file.txt"), but not the name of the bucket.
    """
    s3_client = boto3.client('s3')
    source_file = {
        'Bucket': source_bucket_name,
        'Key': source_file_name
    }
    s3_client.copy(source_file, dest_bucket_name, dest_file_name)

copy_file_between_s3_buckets('my-first-bucket', 'doc1.txt', 'my-second-bucket', 'docs/document.txt')
copy_s3_file_to_local('my-second-bucket', 'docs/document.txt', '~/Documents/')

```


#### Google Cloud Storage

#### Azure Storage

### Web APIs

### Database Connections

### Web Scraping