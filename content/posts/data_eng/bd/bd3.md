---
title: "Big Data 3. Data Warehouses, Data Lakes, and Lake Houses"
date: 2025-06-04
draft: true
ShowToc: true
tags: ["big data", "dbt", "basics", "data warehouse", "data lake", "lakehouse", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

## 1. Data Warehouses

A *Data Warehouse (DW)* is a centralized repository that aggregates data from multiple sources into a unified, consistent store optimized for analytics and business intelligence (BI) activities, such as reporting, forecasting, and data mining.

Unlike transactional databases (OLTP), data warehouses are built for **analytical workloads (OLAP)**: they support complex queries over large volumes of data, often using denormalized schemas, dimensions, and facts for performance.

Data warehouses typically use:

- **Columnar storage** for efficient scanning and aggregation.
- **Structured data formats** for optimized query performance.

> They are **not suitable for unstructured data** like images, videos, or raw documents.

Users interact with DWs using SQL, and popular BI tools connect to them to power dashboards and analytics pipelines.

---

### a) On-Premise vs Cloud-Based Data Warehouses

There are two main types of data warehouses:

1. **On-Premise DWs**  
   - Fully managed in-house: you control storage, compute, networking, and security.  
   - High upfront cost: hardware, software licenses, and maintenance.  
   - Requires dedicated teams: network engineers, DBAs, and sysadmins.  
   - **Scaling** is manual and slow: It involves purchasing and provisioning new infrastructure.

2. **Cloud-Based DWs**  
   - Fully managed by providers (e.g., Snowflake, BigQuery, Redshift).  
   - **Elastic scalability**: compute and storage scale independently and on-demand.  
   - **Predictable pricing** models (pay-as-you-go or reserved capacity).  
   - The provider handles hardware, patching, and scaling infrastructure.

Most modern enterprises prefer cloud DWs for their **flexibility, cost-efficiency**, and **ease of scaling**, especially when dealing with fast-growing or unpredictable data needs.

---

### b) Examples of Data Warehouses

#### Cloud-based Data Warehouses
- **Amazon Redshift** - Fully managed DW on AWS; supports Redshift Spectrum for external tables.
- **Google BigQuery** - Serverless DW on GCP with support for federated queries and external data sources.
- **Snowflake** - Cloud-native DW supporting multi-cloud deployments and separation of compute/storage.
- **Azure Synapse Analytics** - Microsoft's DW platform with support for serverless querying and integrated analytics.
- **Databricks SQL Warehouse** - Built on top of Delta Lake for analytics over large-scale lakehouse data.

#### On-Premise or Hybrid Data Warehouses
- **Teradata** - Scalable enterprise-grade DW, often used in financial services and telecom.
- **Oracle Exadata** - High-performance DW platform from Oracle, often deployed in enterprise data centers.
- **IBM Db2 Warehouse** - Optimized for private clouds or on-premise deployments.
- **Vertica** - Columnar DW with high-speed analytics, available on-prem and in cloud.

> Many modern warehouses also support **hybrid deployments**, where compute may reside in the cloud while data remains on-prem or in a data lake.

#### Open-Source Data Warehouse Options

While most enterprise data warehouses are managed or proprietary, there are some powerful open-source alternatives:

- **ClickHouse** - A high-performance, columnar OLAP database optimized for real-time analytics. Ideal for log data, dashboards, and metrics platforms.
- **Apache Druid** - A real-time analytics database designed for interactive queries on streaming and batch data.
- **Apache Hive** - Originally built for querying large datasets on Hadoop using SQL-like syntax. Often used with data lakes.
- **DuckDB** - Lightweight, in-process OLAP database designed for analytics on local files. Great for local development and embedded use cases.
- **Greenplum** - Massively parallel PostgreSQL-based DW, suitable for large-scale on-prem analytics.

> These open-source tools may require more setup and tuning but offer cost-effective, customizable alternatives for analytics and BI.

## 2. Data Lake

*Data lakes* are designed to store massive amounts of structured, semi-structured, and unstructured data in raw form. Unlike data warehouses, they focus on low-cost storage and flexibility, using formats like JSON, CSV, Parquet, or ORC.


### External Tables

External tables allow querying data stored in a data lake (e.g., Amazon S3) directly from a data warehouse (e.g., Redshift, BigQuery) without importing the data. This decouples compute from storage and brings cost and performance flexibility.


One challenge with traditional data warehouses — whether on-premise or cloud-based — is the tight coupling of **compute and storage**. This leads to several issues:

- You may be charged for compute even when it's idle.
- Autoscaling may not react quickly enough to handle unpredictable or peak workloads.
- Loading large volumes of raw or temporary data into the warehouse can be inefficient and costly.

To address these problems, many modern architectures adopt **external tables** to **decouple compute from storage**.

An **external table** allows the data warehouse to **query data stored outside of itself**, typically in a data lake or object storage system such as:

- Amazon S3
- Google Cloud Storage
- Azure Blob Storage
- Apache Hadoop (On-premise)

These external tables reference raw files (e.g. Parquet, ORC, CSV, JSON), but can be queried using **standard SQL**, just like internal warehouse tables.

---

##### Benefits:
- Separation of compute and storage enables independent scaling.
- No need to ingest or copy data into the warehouse — the data is queried in place.
- Lower storage costs: object storage is cheaper than warehouse-managed storage.
- Serverless infrastructure: fully managed by the cloud provider.

---

##### Query Performance Trade-offs:
- **External tables may be slower** than internal tables, especially:
  - If the external data isn't optimized (e.g. not partitioned, compressed, or columnar).
  - For complex joins, aggregations, or filters without indexes.
- Performance can often be improved by:
  - Using **columnar formats** like Parquet or ORC.
  - Applying **partitioning and bucketing**.
  - Caching frequently queried datasets.

---

##### Common Use Cases:
- Exploratory analysis on newly ingested or raw data.
- Data lakehouse architectures where the same data is shared between a lake and a warehouse.
- Staging layer before data is cleaned and loaded into structured tables.
- Cross-service querying — e.g., combining Redshift internal data with external S3 logs.

---

##### Examples of Supported Platforms:
- Amazon Redshift Spectrum
- Google BigQuery External Tables
- Snowflake External Tables
- Databricks + Unity Catalog
- Azure Synapse Serverless SQL Pools

> From the user’s perspective, querying external tables feels almost identical to querying internal ones, but the data physically resides **outside** the warehouse.

## 3. Data Lakehouses

*Data Lakehouses* emerged as a modern solution to combine the scalability and flexibility of data lakes with the data management and performance features of data warehouses.

They aim to address the shortcomings of traditional data lakes, which lack structure and reliability, and data warehouses, which can be costly and less flexible with unstructured data.

A Data Lakehouse is a unified data platform that supports:

- Structured, semi-structured, and unstructured data (like in data Lakes)
- Data engineering, BI/analytics, data science, and machine learning workloads (like in data warehouses)
- ACID transactions, schema enforcement, and metadata management (like in data warehouses)
- Low-cost storage and scalable processing (like in data lakes)

> It essentially brings the best of both worlds — the schema and governance of warehouses and the cost-efficient, large-scale storage of lakes, enabling a single platform for for analytics, BI, and machine learning.

---

### Data Lake vs. Data Warehouse vs. Data Lakehouse



{{< table >}}

| Feature                     | Data Lake            | Data Warehouse        | Data Lakehouse        |
|:-----------------------------:|:----------------------:|:------------------------:|:------------------------:|
| Data Types                  | All (unstructured, etc.) | Structured only        | All                   |
| Cost                        | Low                  | High                   | Medium                |
| Performance (analytics)     | Low (without prep)   | High                   | High                  |
| ACID Transactions           | X                   | O                     | O                    |
| Schema Enforcement          | X                   | O                     | O                    |
| ML & Data Science Friendly  | O                   | X (limited)            | O                    |

{{< /table >}}
---

### Key Features

- **Support for ACID transactions** (via technologies like Delta Lake, Apache Iceberg, Hudi)
- **Unified storage layer** — typically object storage like S3, Azure Data Lake, or GCS
- **Open format** (e.g., Parquet, ORC) with metadata layers
- **Streaming + batch** capabilities
- **Scalable** architecture for multi-persona access (data engineers, analysts, scientists)

---

### Examples of Lakehouses
- **Delta Lake**  -  Open-source storage layer developed by Databricks 
- **Apache Iceberg** - Table format for huge analytic datasets        
- **Apache Hudi** -  Incremental processing for big data lakes       
- **Databricks**  -  Lakehouse platform built on Delta Lake          
- **Snowflake** -  Expanding to include Lakehouse capabilities     
- **BigQuery**  -  Google's solution blending lake + warehouse     


> 