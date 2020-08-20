# Data Engineering in the Cloud: Start to Finish

NOTE: This book is currently incomplete. If you find errors or would like to fill in the gaps, read the **Contributions** section below.

## Table of Contents
[Preface](https://github.com/Nunie123/data_engineering_book) <br>
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

Copy everything from "my_folder" in the "my-bucket" bucket into the current working directory:

`> aws s3 cp s3://my-bucket/my_folder ./`

Move the file "iceCream.txt" from "cool-bucket" to "cooler-bucket":

`> aws s3 mv s3://cool-bucket/iceCream.txt s3://cooler-bucket/`

The `sync` command you may not be as familiar with, but it's nonetheless pretty intuitive. By syncing an S3 path to a local path any changes to the files in the S3 path will also applied to the local path. For example, you can ensure that any files added to an S3 bucket are also added to your local directory.

Sync files from bucket "coolest-bucket" with current directory.

`> aws s3 sync s3://coolest-bucket .`

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
    This function downloads a file in an S3 bucket to a local directory. Note that the "file_name" parameter should also include sub-directories, if applicable (e.g. "my/folder/file.txt"), but not the name of the bucket.
    """
    s3_client = boto3.client('s3')
    s3_client.download_file(bucket_name, file_name, destination)

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

copy_file_between_s3_buckets('my-first-bucket', 'doc1.txt', 'my-second-bucket', 'docs/document.txt')
copy_s3_file_to_local('my-second-bucket', 'docs/document.txt', '~/Documents/')

```


#### Google Cloud Storage
GCS is organized much like AWS's S3, with the ability to create buckets, put objects (called "blobs") into those buckets, and organize blobs inside sub-folders. Indeed, Google has made it as painless as possible to switch from S3 to GCS, going so far as to provide a command line tool that allows copying directly from S3 to GCS (a nice feature).

_CLI_

Whereas AWS has a single command line tool, Google Cloud Platform offers several. The tool we'll need to interact with GCS is [gsutil](https://cloud.google.com/storage/docs/gsutil). To get gsutil you need to install Google Cloud SDK, which can be done through your favorite package manager, curl (`curl https://sdk.cloud.google.com | bash`), or by downloading an installer. More details are [here](https://cloud.google.com/storage/docs/gsutil_install).

Once installed, you should configure gsutil by running `gsutil init`. If you intend to use gsutil to access S3 buckets, then you should also set up a [Boto configuration file](https://cloud.google.com/storage/docs/boto-gsutil).

The gsutil offers lots of commands for monitoring and manipulating GCS, but in this section we'll focus on the primary commands you'll use to get data out of GCS: [cp](https://cloud.google.com/storage/docs/gsutil/commands/cp), [mv](https://cloud.google.com/storage/docs/gsutil/commands/mv), and [rsync](https://cloud.google.com/storage/docs/gsutil/commands/rsync).

Copy the contents of "my-bucket" bucket to the "files" directory:

`> gsutil cp gs://my-bucket/* ./files`

Move all .csv file from "my-bucket" bucket inside the "files" sub-folder to the current working directory.

`> gsutil cp gs://my-bucket/files/*.csv .`

The rsync command is analogous to the `aws s3 sync` command. It allows you to synchronize a local directory (or GCS bucket or S3 bucket) with a GCS bucket. Files added or altered in the GCS bucket will be added or altered in the rsync directory. It is also possible to have delete files that have been removed from the GCS bucket, but this is not the default behavior.

Synchronize contents of "my-bucket" with current working directory.

`> gsutil rsync gs://my-bucket .`

We've just the scratched the service of these three commands (and completely ignored a couple dozen other commands), so it's worth your time to review the gsutil [documentation](https://cloud.google.com/storage/docs/gsutil).

_PYTHON_

To access GCS from Python you'll need the google-cloud-storage package (`pip install google-cloud-package`). You'll also need to set up authentication. Unlike AWS, Google requires you to set up a service account, generate a key as a JSON file, then point the `GOOGLE_APPLICATION_CREDENTIALS environment variable at this JSON file. More details are [here](https://cloud.google.com/storage/docs/reference/libraries#client-libraries-install-python.

```Python
from google.cloud import storage

def save_blob_to_file(blob_uri: str, destination_path: str) -> None:
    """
    This function downloads a specified file from GCS to a local file. The blob URI should be in the format "gs://bucket_name/path/to/blob/file_name"
    """
    client = storage.Client()
    with open(destination_path) as f:
        client.download_blob_to_file(blob_uri, destination_path)

def copy_blob_to_new_bucket(source_bucket_name: str, blob_name: str, dest_bucket_name: str) -> None:
    """
    This function copies a file from 
    """
    client = storage.Client()
    source_bucket = client.bucket(source_bucket_name)
    source_blob = source_bucket.blob(source_blob_name)
    destination_bucket = client.bucket(dest_bucket_name)

    source_bucket.copy_blob(source_blob, destination_bucket)

copy_blob_to_new_bucket('dev_bucket', 'my_file.txt', 'prod_bucket')
save_blob_to_file('gs://prod_bucket/my_file.txt', '~/Documents/')

```
Documentation for the Python storage library is [here](https://googleapis.dev/python/storage/latest/client.html).


#### Azure Storage

### Web APIs
When dealing with data from 3rd party data sources you'll likely have to navigate their web API to get the data out. Each web API is going to be different, but fortunately the tools to get the data out are often simple to implement. Below is some Python code getting data out of the Wikipedia API. In Chapter 7: Orchestrating Your Pipelines we'll discuss setting up a cloud environment for your code to run in.

```Python
"""
These functions get data from the Wikipedia web API. 
Documentation here: https://www.mediawiki.org/wiki/API:Main_page
"""
import requests
import urllib

def get_first_10_results(search_term: str) -> list:
    """
    This function returns the first 10 results in a request against the Wikipedia search endpoint. It is common for web APIs
    to limit the number of items in a response. For Wikipedia, the default number of results returned is 10.
    """
    encoded_search_term = urllib.parse.quote(search_term)  # this encodes the search term so it can be part of a URL
    url = f'https://en.wikipedia.org/w/api.php?format=json&action=query&list=search&srsearch={encoded_search_term}'
    response = requests.get(url)
    response.raise_for_status()
    data = response.json()
    search_results = data['query']['search']
    return search_results


def get_all_results(search_term: str) -> list:
    """
    To get all of the results from a web API request you will generally have to implement pagination. Each "page" is a new request using a token from the previous request. Depending on the number of requests you need to make, this can take awhile.
    """
    encoded_search_term = urllib.parse.quote(search_term)  # this encodes the search term so it can be part of a URL
    base_url = f'https://en.wikipedia.org/w/api.php?format=json&action=query&list=search&srlimit=500&srsearch={encoded_search_term}'
    get_next_page = True
    search_results = []
    url = None
    while get_next_page:
        url = url or base_url
        response = requests.get(url)
        response.raise_for_status()
        data = response.json()
        search_results.extend(data['query']['search'])
        get_next_page = data.get('continue')
        if get_next_page:
            sroffset = data.get('continue').get('sroffset')
            continue_string = data.get('continue').get('continue')
            url = base_url + f'&sroffset={sroffset}&continue={continue_string}'
    return search_results

```

### Database Connections
There's a good chance that another part of your organization will be storing data in a database somewhere, and it'll be your job to get that data out. There is a huge assortment of database systems out there, but we'll focus on PostgreSQL and MongoDB in this section. 

These systems could be on-premises or in the cloud. Where the database is hosted will impact authentication, but the process for getting the data out should be the same. We'll look at exporting data through command line tools, using driver libraries for Python, and using SQLAlchemy, a popular library for interacting with databases in Python.

### Web Scraping