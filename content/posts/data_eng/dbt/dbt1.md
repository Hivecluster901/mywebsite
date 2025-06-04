---
title: "DBT 1. Data Maturity Model"
date: 2025-06-03
draft: false
ShowToc: true
tags: ["dbt", "basics", "data maturity model", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

## 1. Intro

The Maslow's hierarchy of needs models the basic needs of humans as a hierarchy of 5 levels from the bottom to higher-level needs at the top. In this model, one cannot proceed to the next level unless the basic needs of the current level is met. 

Silmiarly, one can think of *data hierarchy of needs*. The 5 levels are **Data Collection, Data Wrangling, Data Integration, BI and Analytics, and Artificail Intellgence.**

Raw data should be collected, transformed and mapped to a useful format for analytics. Then, clean data should be loaded to data lake or data warehouse so that the later stages, such as Business Analytics and Machine Learning can be performed.

Here, we focus on the first three stages of data evolution.

Data collection is extraction of data from difference sources systems. Data can come in different format and different ways. *Three V's of big data* are *variety*, *velocity*, and *volume*. When designing our data architecture, it is important to think about the *three V's* to build reliable, scalable and maintainable systems.

Data wrangling, or data transform stage, is cleaning of raw data and improvement of data quality. For example, duplicate data, missing data, typos, and errors often create inconsistencies and mistakes. The goal is to convert data into a data warehouse format.

Data integration is about writing the transformed data from the staging area to a target database, where the analytics workloads will be performed. Loading can be done by two ways: refresh (complete rewrite) and update.

## 2. ETL vs. ELT

The above explains ETL, which stands for Extract, Transform, and Load. When the storage and compute resource were expensive, it was important to consider what data you want to persist in the database. Thus, transformation was performed outside of the database in the staging area. Thus, ETL was wide spread and traditional approach.

There are a few problems associated with ETL:

- Schema changes in source systems can break ETL pipelines, leading to data processing errors or incorrect results. Maintaining schema compatibility requires version control, validation, and robust schema evolution strategies.
- The rate at which data is generated or ingested (data velocity) can fluctuate, affecting the performance and reliability of ETL pipelines.
- Scaling ETL pipelines to accommodate growing data volumes and complexity can be challenging.
- Integrating a new data source into an existing ETL pipeline can be time-consuming and complex.
- Testing and debugging ETL processes is often difficult due to the complexity, volume, and asynchronous nature of data pipelines.

Nowadays, the storage and computational resource are so cheap that there is no harm in loading raw data from the source system directly to the destination. Data warehouses, e.g. snowflake, redshift, and BigQuery are extremely scalable and performant, and it is natural to change from ETL to ELT, i.e. the transformations are performed inside the database rather than an external processing layer. 

Doing the transformation inside the database facillitates handling issues with upstream schemas and downstream data models. Tools like Fivetran and stitch are used to connect to source systems and load data into our destination, i.e. Extract and Load.


