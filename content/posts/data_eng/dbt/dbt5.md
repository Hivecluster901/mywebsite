---
title: "DBT 5. Modern Data Stack"
date: 2025-06-04
draft: false
ShowToc: true
tags: ["dbt", "basics", "modern data stack", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

As the cost of cloud storage and compute continues to fall and internet speeds rise, data engineers are rethinking how data is stored, processed, and analyzed. This shift has led to what we now call the **modern data stack**: a flexible, scalable, cloud-native approach that replaces traditional monolithic systems.

In this post, we’ll explore key architectural changes that enabled the modern data stack, from the transition from **SMP to MPP**, to the **decoupling of storage and compute**, and the rise of **columnar databases**, **ELT workflows**, and the **tooling ecosystem** that glues it all together.

---

## 1. From SMP to MPP

### 1.1 What Is SMP?

**Symmetric Multiprocessing (SMP)** is a legacy database architecture where multiple CPU cores share a single memory space, storage, and operating system.

- **Shared resources**: Memory, I/O devices, and disk are accessed through a common bus.
- **Single-machine design**: All computation happens on one machine.
- **Scaling**: Limited to *vertical scaling*—adding more resources to a single system.
- **Performance**: Great for OLTP-style workloads but restricted by hardware limits.

### 1.2 Enter MPP

**Massively Parallel Processing (MPP)** systems scale horizontally by distributing data and computation across many machines.

- **Independent nodes**: A master node coordinates multiple compute nodes.
- **Parallelism**: Data is sharded across nodes; each node works in parallel on its portion.
- **Scaling**: Add more nodes to scale out.
- **Storage models**:
  - **Shared-nothing** (most common): Each node has local storage.
  - **Shared-storage**: All nodes access a common (often cloud-based) storage layer.

> **Note:** In cloud-native architectures, “shared storage” often refers to distributed object storage like S3, accessed independently by stateless compute nodes.

### 1.3 SMP vs. MPP Comparison
{{< table >}}

| Feature               | SMP                                      | MPP                                        |
|-----------------------|-------------------------------------------|---------------------------------------------|
| Architecture          | Single machine, shared memory            | Master + multiple independent nodes         |
| Scaling               | Vertical only                            | Horizontal (scale-out)                      |
| Storage               | Shared across cores                      | Local per node or shared via cloud storage  |
| Parallelism           | Within one machine                       | Across multiple machines                    |
| Use Case Today        | Legacy on-prem systems                   | Modern cloud warehouses                     |
{{< /table >}}

### MPP Architecture Diagram

<!-- ![MPP Diagram](/assets/data_eng/dbt/dbt3/mpp_architecture.png) -->
<img src="/assets/data_eng/dbt/dbt5/mpp_architecture.png" alt="Massively Parallel Processing (MPP) Architecture" style="max-width: 20vw; width: 100%; height: auto; margin: auto;" />

---

## 2. Decoupling Storage and Compute

Traditionally, storage and compute were tightly linked. Cloud platforms changed this with **decoupled architecture**:

- **Storage**: Durable, cheap, and separate (e.g., Amazon S3, GCS, Azure Blob).
- **Compute**: Stateless and elastic (e.g., BigQuery slots, Snowflake virtual warehouses).

### Benefits

- Scale compute and storage independently.
- Pay only for compute when in use.
- Persistent storage allows compute clusters to spin down when idle.

---

## 3. Column-Oriented Databases

### 3.1 Row vs. Column Storage

**Row-oriented databases** are optimized for OLTP (e.g., inserting or updating entire records), while **column-oriented databases** are better for OLAP (analytical queries on large datasets).

**Columnar storage** allows:

- Selective column reads (reducing I/O)
- Efficient compression (similar data grouped together)
- Vectorized computation

### 3.2 Comparison
{{< table >}}

| Feature               | Row-Oriented                        | Column-Oriented                     |
|:-----------------------:|:--------------------------------------:|:-------------------------------------:|
| Layout                | Records stored together              | Fields stored together              |
| Use Case              | OLTP (transactions)                  | OLAP (analytics, aggregations)      |
| I/O Efficiency        | Reads full rows                      | Reads only needed columns           |
| Compression           | Moderate                             | High                                |
| Examples              | PostgreSQL, MySQL                    | Snowflake, Redshift, BigQuery       |
{{< /table >}}

---

## 4. From ETL to ELT

The traditional **ETL** pattern:

1. **Extract**
2. **Transform**
3. **Load**

has largely been replaced by **ELT**:

1. **Extract**
2. **Load** raw data into the warehouse
3. **Transform** it inside the warehouse

This modern pattern simplifies pipelines and leverages cloud compute.

> **Tradeoff:** While ELT increases flexibility and auditability, it also shifts responsibility for data quality downstream—often to analytics engineers.

---

## 5. Tools in the Modern Data Stack

Each layer is served by specialized tools that integrate via APIs, JDBC/ODBC, or event systems.
{{< table >}}

| Layer              | Tools                               | Role                                                                  |
|--------------------|--------------------------------------|-----------------------------------------------------------------------|
| Extract & Load     | Fivetran, Airbyte, Stitch            | Ingest data into warehouses from APIs, DBs, and flat files            |
| Transform          | DBT, Matillion, dataiku                                  | Define SQL-based models, documentation, tests                         |
| Data Warehouse     | Snowflake, BigQuery, Redshift        | Scalable compute + storage for structured data                        |
| BI & Analytics     | Looker, Tableau, Power BI, Metabase  | Dashboards, visualization, exploration                                |
| Reverse-ETL        | Census, Hightouch                    | Sync clean data from warehouse to CRMs, ads, tools                    |
{{< /table >}}


> **JDBC** and **ODBC** are protocol standards used to connect external applications to databases.
> - **JDBC**: Java-specific
> - **ODBC**: Language-agnostic (used by Excel, Python, R, etc.)

Modern data warehouses expose these interfaces so apps can access/query data securely.

---

## 7. Metadata & Orchestration

### 7.1 Orchestration Tools

Managing dependencies, schedules, and retries across the data pipeline is critical. Popular orchestration tools:

- **Apache Airflow**: DAG-based Python workflows
- **Dagster**: Strong typing + observability
- **Prefect**: Pythonic, cloud-native orchestration with minimal config

These tools manage the “when and how” of pipeline execution.

### 7.2 Metadata & Governance

Understanding **what data exists**, **where it comes from**, and **who uses it** is essential for modern data teams.

Popular metadata tools:
{{< table >}}

| Tool       | Role                                      |
|:------------:|:-------------------------------------------:|
| DataHub    | Metadata discovery & lineage              |
| Amundsen   | Data catalog with search and context      |
| Atlan      | Collaborative metadata, governance, access |

{{< /table >}}
These tools help with **lineage, discoverability, data ownership**, and **compliance**.

---

The modern data stack is:

- **Modular**: Specialized tools connected via APIs
- **Scalable**: Independent scaling of compute/storage
- **Efficient**: Columnar, distributed, and cloud-optimized

Key enablers include:

- **MPP architectures**
- **Decoupled storage/compute**
- **Columnar formats**
- **ELT processing**
- **Orchestration and metadata tools**


<!-- ---
title: "DBT 3. Modern Data Stack"
date: 2025-06-04
draft: false
ShowToc: true
tags: ["dbt", "basics", "mode data stack", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

As the price for storage and compute start to drop and the internet becomes faster, data engineers start to think of the modern data stack.

In particular, they moved from legacy symetric multiprocessing database design (SMP) to (MPP) Cloud database design. An SMP system is one in which many processors share resources, so single instances of memory, io devices, storage and operating system are examples of these shared resources. Thses resources are linked via a shared bus. The processing occurs in parallel inside the CPU cores while other resources are shared. An SMP system consists of a single physical computer with several CPU cores. The storage and compute are tightly coupled, and thus it can be only scaled vertically. The problem is that there are limitations as to how much I can fit into one machine. Since the procesing happens in a single machine, it led to faster processing time. So this is a good design given the processing times, but it is limited by vertical scaling and consistent back up of data. Back then, we didn't have distributed routing technologies required to store data in distributed systems.

As the distributed systems and routing technologies evolved and the data transmission speeds over network got better, it opened doors to milti parallel processing (MPP) database architectures. These systems are made of a master node and many compute nodes that don't share memory. Each node is equipped with its own operating system and memory. The master node is in charge of coordinating the operations among the compute nodes and communicates with them over high speed messaging interface. Storage can be shared or not: shared storage or shared nothing MPP architecture. Shared nothing MPP is the most common MPP design. There is no difference across the systems. Every node has its own memory and disk storage, and data is spread between the nodes depending on the distribution mechanism employeed. Data from a single table might span many node and each node only processes the data rows on its related disks for it for processing. To increase more scalability, we may add more nodes which associate with associated memory and disk storage, which is known as horizontal scaling or scaling out. You would connect to a master node as a user or an application via JDBC or ODBC, and the master node would coordinate the distribution process. Cloud data warehouses all havean underlying MPP architecture, such as Azure synapse, BigQuery. The shift from SMP to MPP was a very critical step towards the modern data stack.

Another important step was the decoupling of storage and compute. As with multiple processing architectures, the control or the master node can evaluate requests from business of analytics applications and route them to a computing node for processing. Computing is only required when a request is made by the business or analytical applications. But the storage is decoupled and may acquire data continuously without involving the compute. You can basically shut down the compute nodes when you don't need them. Your data will be saved inside a low cost storage system such as Amazon S3 or Azure Blob storages. It also lets you scale your compute very dynamically. If you have a very compute intensive workload, you can just add more compute node to your cluster without having to scale to the storage with it as the storage reside outside of the worker nodes.

Another step that happended and really contributed to the modern data stack was the creation or introduction of column oriented databases. Row oriented databases, on the other hand, arrange data by record, so storing all of the data associated with the reconrd in memory. Row oriented databases are the conventional technique of arranging data, and they continue to offer some important advantages for storing data fast. Typically, these row oriented databases are designed to be efficient when it comes to writing rows, or reading rows. So you would still want to use these row oriented databases in OLTP databases like Postgres or MySQL. However, it turns out that row oriented databases are not exactly good for analytical workloads. column oriented databases were designed and introduced. Column oriented databases arrange data by field, storing all the data associated with a field in memory. Columnar databases have increased in popularity because they improve query performance, so they are optimized for efficient column reading and computation. You don't have to load the entire table for computation. It requires less input and output operations and can make the query times a lot faster. Great for OLAP databases. MPP datawarhouses today are all columnar.

Because of the MPP column oriented and decoupled cloud data warehouses, we can shift away from the ETL to ELT. Fivetran and stitch are one of the most popular extract and load tools. The transformation layer sits on top of a cloud data warehouse and it uses DBT. Looker for BI tools and Census for reverse ETL. Legacy tools had data storage, datawarehouse and visualization all vertically integrated. modern data stack is horizontally integrated. -->