<!--yml
category: 未分类
date: 2024-05-27 14:54:38
-->

# Open Table Formats Are Inevitable For Analytical Datasets | Ensemble

> 来源：[https://ensembleanalytics.io/blog/open-table-formats-inevitable](https://ensembleanalytics.io/blog/open-table-formats-inevitable)

In recent years we have seen the emergence of open source "table formats" projects such as [Apache Iceberg](https://iceberg.apache.org/), [Apache Hudi](https://hudi.apache.org/) and [Delta](https://delta.io/) by Databricks.

These formats allow you to store large collections of analytical data in static files which are well structured, fast to query and modifiable using insert, update, delete and merge operations.

Though data formats sound like a dry technical topic to anyone outside of data engineering geeks, I think this will be one of the bigger architectural changes in enterprise IT in the coming years.

I believe this pattern of storing data in open table formats on cloud object stores such as AWS S3 is so compelling that I think every organisation will trend towards it. It is going to be a a major theme in how businesses manage and work with their data.

## [](#what-are-open-source-table-formats)What Are Open Source Table Formats?

Historically, a lot of analytical data has been stored in data warehouses which save the actual data in closed proprietary formats.

Data lakes moved this forward with a more open approach, using open file formats such as CSV, JSON, and most often Parquet.

Table formats such as Iceberg, Hudi and Delta build on this and add extra capabilities:

*   A table like structure and schema, meaning that we can treat files as tables with rows and columns;

*   Inserts, updates, deletes and merge operations meaning that we can update them as we would a relational database table;

*   Transactional safety, meaning that multiple users can be inserting, updating and reading from the tables concurrently without seeing inconsistent data;

By using these table formats, we take a big step towards turning data lakes into databases by making them well structured, easier to work with and scalable to more concurrent users.

## [](#big-implications)Big Implications

The main implications of these table formats is that they are open source and standardised, meaning that any tool can access the data stored within then when both reading and writing.

As well as avoiding any lock-in, this could drive a wave of democratising access to data when it is not trapped away in proprietary datastores and instead opened up to any tool in your organisation without any integration.

If you wish to change the tools in your stack or implement specialist data stores for certain use cases, you’ll be able to do so simply by connecting your database to the files as some kind of *external table*. This means that it will be possible to innovate with data much easier than if we had to do complex data migrations or implement ETL processes to move data between stores.

Conceptually, this puts data at the heart of the architecture and turns the tools into a supporting cast.

## [](#why-is-this-inevitable)Why Is This Inevitable?

Open Table Formats stored on cloud object stores is such a useful pattern that I think it's adoption will be inevitable. The benefits include:

*   Open Standards: All data will be stored in consistent formats and based on open standards and specifications. This totally avoids lock-in and makes migration between tools and platforms significantly easier;

*   Cost: Data is stored on cloud object stores, which are notoriously cheap;

*   Reliability: Cloud object stores such as AWS S3 are known for their reliability, and as this is a significantly simplified way of working with data the entire stack is made simpler;

*   Streaming Support: Open source table formats can support both batch processing and streaming operations, making them a good fit for a future world where we need to process data as it is continuously generated;

*   Simplicity: This stack reduces so much complexity as we are effectively working with raw files.

These benefits of this [new architecture](/blog/new-architecture-cloud-native-data) are so compelling that surely every business with any data and analytics capability and scale will converge on it. This is highly disruptive for vendors who have a large part of their offering commoditised, but the benefits are significant for end users.