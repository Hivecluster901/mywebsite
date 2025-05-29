---
title: "Basic SQL Commands 1"
date: 2025-05-29
draft: false
ShowToc: true
---

Here, we go through basic SQL commands:

## 1. SELECT
*SELECT* is used to select and return data.
For a single column, the syntax is as below:

``` sql
SELECT
column_name
FROM table_name
```
For multiple columns, you just list the column names with `,`.
``` sql
SELECT
column_name1,
column_name2
FROM table_name
```
To select all columns, use `*`.
``` sql
SELECT
*
FROM table_name
```

Note that **the exact format doesn't matter**, e.g. the two examples are the same:
``` sql
SELECT * FROM table_name
```
``` sql
SELECT 
* 
FROM table_name
```

## 2. ORDER BY
*ORDER BY* is used to order results based on a column. The criterion depends on the data type of the chosen column. For example, if the column consists of texts, the table will be ordered *alphabetically*. If the column consists of numbers, the table will be ordered *numerically*. If the column consists of dates or times, the table will be ordered *chronologically*. *ORDER BY* can be used in conjunction with *SELECT*.

The syntax is as follows:
``` sql
SELECT
column_name1
column_name2
FROM table_name
ORDER BY column_name1
```

You can order the table with multiple columns. Just list the column names with `,`.

## 3. DESC/ASC
*DESC* and *ASC* are used to specify the orders.

``` sql
SELECT
column_name1
column_name2
FROM table_name
ORDER BY column_name1 DESC, column_name2 ASC
```
Numbers can be used instead of column names to be specified. For example,

``` sql
SELECT
column_name1
column_name2
FROM table_name
ORDER BY 1 DESC, 2 ASC
```
However, such is not a recommended practice.


