## Automation of Building a Transactional Data Lake
1. [Introduction](https://github.com/vrund2/Transactional-Data-Lake-Pipline-/blob/main/README.md#introduction)
2. [Guide Overview](https://github.com/vrund2/Transactional-Data-Lake-Pipline-/blob/main/README.md#guide-overview)
<br>

## Introduction
This guide provides a simple and deployable use case for automating the process of building a transactional data lake on the AWS environment. With this guide, you will configure the parameters of AWS Glue, including choosing an open table format among Apache Hudi, Apache Iceberg, and Delta Lake. Once the parameters are configured, you will deploy this guide on your AWS environment and initiate the automated process of building a transactional data lake using the chosen open table format.
<br>
<br>
## Guide Overview
#### Background
Many customers are interested in building a transactional data lakses with open table formats such as Apache Hudi, Apache Iceberg, Delta Lake on the AWS environment. This approach helps overcome the limitations of a traditional data lakes. However, when building a transactional data lake on AWS environment, customers may face the following challenges:

- Difficulty in choose an open table format that fits their functional requirements and supports integrations with other AWS services
- Significant learning curve and development overhead associated with building a transactional data lake

This guide aims to mitigate the above challenges that you may encounter. We encourage you to use this guide as a high-level guidance which can help you to understand how a transactional data lake is built on AWS environment with each open table formats, and eventually to identify what open table format is likely a good fit for your use case. 

#### Proposed Guide
<!--[Image] Architecture Diagram-->
![Architecture Diagram](./images/aws-smaples.jpg)

The focus area of the guide is to help customers understand how to build a transactional data lake with their preferred open table format. To simplify the guide and save costs, we make the following assumptions:

* We have one database named ```game``` and four tables named ```item_data```, ```play_data```, ```purchase_data``` and ```user_data``` under this database in Amazon RDS.
* The tables stored in Amazon RDS were migrated into Amazon S3 (Raw Zone)‘s ```initial-load``` folder by the initial (full-load) replication task of Amazon DMS.
* Ongoing changes in tables stored in Amazon RDS, after initial (full-load) replication task, have been captured and migrated into Amazon S3 (Raw Zone)‘s ```cdc-load``` folder by the ongoing replication task of Amazon DMS.

The folder structure of Amazon S3 (Raw Zone) will be as follows:
```
.
├── cdc-load
│   └── game
│       ├── item_data
│       ├── play_data
│       ├── purchase_data
│       └── user_data
└── initial-load
    └── game
        ├── item_data
        ├── play_data
        ├── purchase_data
        └── user_data
```

With the above assumptions, the guide will only provision the cloud resources enclosed with AWS CDK box on the above architecture diagram image as default.
However, if you choose Delta Lake as the open table format, more cloud resources need to be provisioned compared to when you choose Apache Hudi and Apache Iceberg, for the following reasons:

- **AWS Glue Data Catalog Update**: AWS Glue currently does not support Data Catalog update option with Delta Lake, which means that a crawler has to be additionally created.
- **Integration with Amazon Redshift**: Amazon Redshift currently requires additional metastore storing  symlink-based manifest table definitions to query Delta Lake tables.

This guide will set up the following cloud resources when you choose **Apache Hudi** or **Apache Iceberg**:
|Resource|Description|
|--|--|
|S3 Bucket(1)|Object Storage for Raw Data Lake|
|S3 Bucket(2)|Object Storage for Transactional Data Lake|
|Glue ETL Job(1)|PySpark Job for creating tables in Glue Data Catalog and S3 Bucket(2) with initial data|
|Glue ETL Job(2)|PySpark Job for updating tables in S3 Bucket(2) with CDC data|
|Glue Data Catalog|Metastore for table definition|

This guide will set up the following resources when you choose **Delta Lake**:
|Resource|Description|
|--|--|
|S3 Bucket(1)|Object Storage for Raw Data Lake|
|S3 Bucket(2)|Object Storage for Transactional Data Lake|
|Glue ETL Job(1)|PySpark Job for creating tables in S3 Bucket(2) with initial data|
|Glue ETL Job(2)|PySpark Job for updating tables in S3 Bucket(2) with CDC data|
|Glue Data Catalog(1)|Metastore for table definition|
|Glue Data Catalog(2)|Metastore for symlink-based manifest table definition|
|Glue Crawler(1)|Classifier for creating table definition in Glue Data Catalog(1) after Glue ETL Job(1) is complete|
|Glue Crawler(2)|Classifier for creating symlink-based manifest table definition in Glue Data Catalog(1) after Glue ETL Job(1) is complete|
<br>


