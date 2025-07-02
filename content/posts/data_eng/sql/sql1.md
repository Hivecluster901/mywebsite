---
title: "SQL 1. Basics"
date: 2025-05-29
draft: true
ShowToc: true
tags: ["sql", "basics", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

Here, we go through basic SQL commands:

## 1. SELECT
*SELECT* is used to select and return data.
For a single column, the syntax is as below:

``` sql
SELECT
column_name
FROM table_name1
```
For multiple columns, you just list the column names with `,`.
``` sql
SELECT
column_name1,
column_name2
FROM table_name1
```
To select all columns, use `*`.
``` sql
SELECT
*
FROM table_name1
```

Note that **the exact format doesn't matter**, e.g. the two examples are the same:
``` sql
SELECT * FROM table_name1
```
``` sql
SELECT 
* 
FROM table_name1
```

## 2. ORDER BY
*ORDER BY* is used to order results based on a column. The criterion depends on the data type of the chosen column. For example, if the column consists of texts, the table will be ordered alphabetically. If the column consists of numbers, the table will be ordered numerically. If the column consists of dates or times, the table will be ordered chronologically. *ORDER BY* can be used in conjunction with *SELECT*.

The syntax is as follows:
``` sql
SELECT
column_name1
column_name2
FROM table_name1
ORDER BY column_name1
```

You can order the table with multiple columns. Just list the column names with `,`.

## 3. DESC/ASC
*DESC* and *ASC* are used to specify the orders.

``` sql
SELECT
column_name1
column_name2
FROM table_name1
ORDER BY column_name1 DESC, column_name2 ASC
```
Numbers can be used instead of column names to be specified. For example,

``` sql
SELECT
column_name1
column_name2
FROM table_name1
ORDER BY 1 DESC, 2 ASC
```
However, such is not a recommended practice.

## 4. DISTINCT
*DISTINCT* is used to select distinct values in a table. The syntax is as follows:
``` sql
SELECT DISTINCT
column_name1
FROM table_name1
```

Multiple columns can be selected distinctly.

``` sql
SELECT DISTINCT
column_name1,
column_name2
FROM table_name1
```
Then, there can be duplicate elements for `column_name1` or `column_name2` as long as the two rows have distinct values for the other columns. 
Thus, it selects distinct rows in terms of all selected rows.

## 5. LIMIT
*LIMIT* is used to limit the number of rows in the output. 
**It is always at the very end of your query.** The syntax is:

``` sql
SELECT
column_name1,
column_name2
FROM table_name1
LIMIT n
```
It limits the output to `n` rows.

## 6. COUNT
*COUNT* is used to count the number of rows in the output. It is often used in combination with grouping and filtering.
The syntax is:
``` sql
SELECT
COUNT(*)
FROM table_name1
```
The above counts all rows in the table.
One can cout all rows of one column.
``` sql
SELECT
COUNT(column_name1)
FROM table_name1
```
The above two usuallt result in the same output. However, note that `COUNT` doesn't count `NULL`'s.
So, If there is `NULL` in `column_name1`, the results will be different.

If one wants to count the number of distinct values of a column, one can use this command:
``` sql
SELECT
COUNT(DISTINCT column_name1)
FROM table_name1
```
Be careful that `DISTINCT` is inside the parentheses.