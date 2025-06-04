---
title: "DBT 4. Denormalized Schema in Data Warehouses"
date: 2025-06-04
draft: false
ShowToc: true
tags: ["dbt", "basics", "modern data stack", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

#### 1. Denormalized Schema

A **denormalized schema** is a database design that reduces the number of joins by **combining related data into fewer tables**, often at the expense of some redundancy. 

- **Optimized for read-heavy analytical workloads**
- Not ideal for OLTP systems (which benefit from normalization)

In a typical data warehouse:
- Data is modeled to **maximize query performance** and **simplify analytics**.
- This contrasts with normalized schemas used in transactional databases, where the goal is data integrity and avoiding redundancy.

---

#### 2. Star and Snowflake Schemas

Two common denormalized designs are:

- **Star Schema**: A central **fact table** connected to multiple **dimension tables**. Dimension tables are usually denormalized.
- **Snowflake Schema**: Similar, but dimension tables are **normalized**, creating additional sub-dimensions. This reduces redundancy but adds joins.

---

#### 3. Fact Tables

**Fact tables** hold the **quantitative data** for analysis — they store **measurable facts** like sales, revenue, clicks, or transactions.

- Often very large
- Contains **foreign keys** to dimensions
- Examples:
- `total_sales`
- `units_sold`
- `order_amount`

Each row in a fact table typically corresponds to a **business event**.

---

#### 4. Dimension Tables

**Dimension tables** provide the **context** for facts — they store **descriptive attributes** about business entities like time, products, customers, or locations.

- Often smaller in size
- Examples:
- `Product Name`, `Category`, `Brand`
- `Customer Name`, `Segment`, `Region`
- `Date`, `Month`, `Year`

Dimension tables allow users to **slice and dice** data — e.g., "Total Sales by Region", "Orders by Product Category".

---

#### 5.Summary

{{<table>}}
| Concept           | Purpose                                | Example                    |
|:-------------------:|:----------------------------------------:|:----------------------------:|
| Denormalized Schema | Speed up reads & simplify queries      | Star schema                |
| Fact Table        | Stores numeric/measurable data         | Revenue, Sales Count       |
| Dimension Table   | Stores descriptive/contextual data     | Product, Customer, Date    |
{{</table>}}
> Denormalization is a conscious trade-off in data warehouses: **we favor performance and simplicity over strict normalization**.
