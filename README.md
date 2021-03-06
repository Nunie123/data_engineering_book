# Data Engineering in the Cloud: Start to Finish
The completely free E-Book for setting up and running a Data Engineering stack on a major cloud platform.

NOTE: This book is currently incomplete. If you find errors or would like to fill in the gaps, read the **Contributions** section below.

## Table of Contents
**Preface** <br>
[Chapter 1: Data Engineering Responsibilities](https://github.com/Nunie123/data_engineering_book/blob/master/ch_1_data_engineering_responsibilities.md) <br>
[Chapter 2: Accessing Data](https://github.com/Nunie123/data_engineering_book/blob/master/ch_2_accessing_data.md)<br>
[Chapter 3: Moving Data to Your Storage](https://github.com/Nunie123/data_engineering_book/blob/master/ch_3_moving_data_to_storage.md)<br>
[Chapter 4: Building Your Data Warehouse](https://github.com/Nunie123/data_engineering_book/blob/master/ch_4_building_data_warehouse.md)<br>
[Chapter 5: Getting Data into Your Warehouse](https://github.com/Nunie123/data_engineering_book/blob/master/ch_5_getting_data_into_warehouse.md)<br>
[Chapter 6: Transforming Your Batch Data](https://github.com/Nunie123/data_engineering_book/blob/master/ch_6_transforming_batch_data.md)<br>
Chapter 7: Orchestrating Your Batch Pipelines<br>
Chapter 8: Streaming Your Data<br>
Chapter 9: Presenting Data to Your Users<br>
Chapter 10: Infrastructure as Code<br>
Chapter 11: EXAMPLE - Building Complete Date Engineering Infrastructure in AWS<br>
Chapter 12: EXAMPLE - Building Complete Date Engineering Infrastructure in GCP

---

# Preface
This is a book designed to teach you how to set up a basic data engineering stack on [Google Cloud Platform](https://cloud.google.com/) and [Amazon Web Services](https://aws.amazon.com/). Each chapter will take a general concept in Data Engineering, discuss the concept generally, then go into implementation details for each cloud provider.

## Who This Book Is For
This book is for people with coding familiarity that are interested in setting up professional data pipelines and data warehouses using a modern cloud platform. I expect the readers to include:
* Data Engineers looking to learn a new cloud platform or learn more about a platform they are already familiar with.
* Junior Data Engineers looking to learn best practices for building and working with data engineering infrastructure.
* Software Engineers, DevOps Engineers, Data Scientists, Data Analysts, or anyone else that is tasked with performing Data Engineering functions to help them with their other work.

This books assumes familiarity with SQL and Python (if you're not familiar with Python, you should be able to muddle through with general programming experience). If you do not have experience with these languages (particularly SQL) it is recommended you learn these languages and then return to this book.

This book covers a lot of ground. Many of the subjects we'll cover in just part of a chapter will have entire books written about them. I will provide references for further research. Just know that while I aim for this book to be comprehensive in the sense that it provides all the information you need to get a stack up and running, I do not attempt the impossible task of compiling all the data engineering information you will ever need.

Finally, there are a vast array of data engineering tools that are in use. I cover many popular tools for data engineering, but many more have been left out of this book due to brevity and my lack of experience with them. If you feel I left off something important, please read the **Contributions** section below.

## How to Read This Book
This book is divided into chapters discussing major Data Engineering concepts and functions. Each chapter is then divided into four parts: an overview and then a separate section for GCP and AWS. 

You can read this book front-to-back if you're just looking to get a general familiarity with common tools on the major cloud platforms. Another good option is to pick a cloud platform and just read that section (plus the introduction) for each chapter. Finally, if you know exactly what topic you want to learn more about, you'll have no trouble skipping to the chapter that has your interest.

I provide code samples in each chapter and you are encouraged to play around with the tools discussed in each chapter. Where applicable, I provide information about pricing both so you can price out your production infrastructure, and so you know how much it will cost to play around with it while learning.

## Contributions

You may have noticed: this book is hosted on GitHub. This results in three great things:
1. The book is hosted online and freely available.
2. You can make pull requests.
3. You can create issues.

If you think the book is wrong, missing information, or otherwise needs to be edited, there are two options:
1. Make a pull request (the preferred option). If you think something needs to be changed, fork this repo, make the change yourself, then send me a pull request. I'll review it, discuss it with you, if needed, then add it in. Easy peasy. If you're not very familiar with GitHub, instructions for doing this are [here](https://gist.github.com/Chaser324/ce0505fbed06b947d962). If your PR is merged it will be considered a donation of your work to the project. You agree to grant a [Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) license for your work. You will be added the the **Contributors** section on this page once your PR is merged.
2. Make an issue. Go to the issue tab above, click to create a new issue, then tell me what you think is wrong, preferably including references to specific line numbers.

I look forward to working with you all.

## Contributors
Ed Nunes. Ed lives in Chicago and works as a Data Engineer for [Zoro](https://www.zoro.com). Feel free to reach out to him on [LinkedIn](https://www.linkedin.com/in/ed-nunes-b0409b14/).


## License
This book is licensed under the [Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0)](https://creativecommons.org/licenses/by-nc-nd/4.0/) license.

---

Next Chapter: [Chapter 1: Data Engineering Responsibilities](https://github.com/Nunie123/data_engineering_book/blob/master/ch_1_data_engineering_responsibilities.md)