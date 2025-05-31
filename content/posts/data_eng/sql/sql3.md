---
title: "Basic SQL Commands 3"
date: 2025-05-31
draft: false
ShowToc: true
---

Here we go through other basic SQL commands for grouping:

## 1. Aggregate Functions
*Aggregate functions* aggregate values in multiple rows to one value. `COUNT` in [Basic SQL Commands 1](/posts/data_eng/sql/sql1/#6-count) is one of aggregate functions. Here are commonly used aggregate functions:

- SUM()
- AVG()
- MIN()
- MAX()
- COUNT()

The examples are as follows:

``` sql
SELECT 
SUM(column_name1)
FROM table_name1
```
You usually need to specify the column name inside the parentheses. `COUNT` accepts `*`. *Aggregate functions* cannot be used with other columns that are not aggregated. This is because aggregation results in only one value while other columns may have multiple values. Thus, the syntax below is not possible:
``` sql
SELECT
SUM(column_name1),
column_name2
FROM table_name1
```
On the other hand, if you *group by* `column_name2`, such is possible. `GROUP BY` is going to be introduced later in this post.
However, you can use multiple aggregations at the same time.
``` sql
SELECT
SUM(column_name1),
COUNT(*),
AVG(column_name2),
FROM table_name1
```

## 2. GROUP BY

When we aggregate, we may want to *group by* certain category. For example, instead of summing all payments made by all customer, we may want to see all payments made by each customer.

*GROUP BY* is used to group aggregations by specfic columns. The syntax is follows:
``` sql
SELECT
column_name1,
SUM(column_name2)
FROM table_name1
GROUP BY column_name1
```
If we want to filter first and order by the grouped column, we can do something like below:
``` sql
SELECT
column_name1,
SUM(column_name2)
FROM table_name1
WHERE column_name1 > n
GROUP BY column_name1
ORDER BY column_name1
```
`WHERE` should comes immediately after `FROM`. Thus, `GROUP BY` comes after either `FROM` or `WHERE`. Also note that `ORDER BY` always comes last. As mentioned in [section 1](#1-aggregate-functions), every column should be either in `GROUP BY` or *aggregate functions*.

You can also group by multiple columns. For example, we can sum over all payments made by each customer for each category of products. This is achieved by the syntax below:
``` sql
SELECT
column_name1,
column_name2,
SUM(column_name3)
FROM table_name1
GROUP BY column_name1, column_name2
```
The example code would look like
``` sql
SELECT
customer_id,
category,
SUM(amount) AS total_payment
FROM sales
GROUP BY customer_id, category
ORDER BY customer_id, total_payment DESC
```
Additioal `ORDER BY` was added to see, for each customer ordered in ascending order, which category of products was bought the most.

## 3. HAVING

*HAVING* is used to filter <u>**groupings**</u> by aggregations. *HAVING* is only used with `GROUP BY`. For example, you may want to filter the groupings where the total amount paid by the customers is bigger than 200.

``` sql
SELECT
customer_id,
SUM(amount) AS total_payment
FROM payment
GROUP BY customer_id
HAVING SUM(amount) > 200
```

Just like `WHERE`, you can use multiple conditions by using `AND` and `OR`. As noted in [Basic SQL Commands 2](/posts/data_eng/sql/sql2/#2-comments--aliases), aliases, e.g. `total_payment`, cannot be used in `HAVING`.