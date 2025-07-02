---
title: "SQL 8. Joins"
date: 2025-06-02
draft: true
ShowToc: true
tags: ["sql", "join", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

`JOIN` is used to combine information from multiple tables in one query. There are four types of `JOIN`s: `INNER JOIN`, `OUTER JOIN`, `LEFT JOIN`, and `RIGHT JOIN`. When combining two tables, you need one common column, so called reference column or join column.

## 1. INNER JOIN

*INNER JOIN* combines two tables only for the rows where the value in the join column occurs in both tables.

If there are `m` matching rows in the first table and `n` matching rows in the second table for the join column value, the result will contain `m × n` rows for that value.


### a) Basic Syntax
Let's look at the syntax.
``` sql
SELECT * FROM table_name1
INNER JOIN table_name2
ON table_name1.join_column1 = table_name2.join_column2
```

For `INNER JOIN`:
- we can swap `table_name1` and `table_name2`. 
- If `join_column1` and `join_column2` are unique to each table, you can omit the table names after `ON`, but it is NOT recommended.
- The columns of the first table, i.e. `table_name1`, appear before the columns of the second table, i.e. `table_name2`.

### b) Using Table Aliases
When the table names are too long, we can give aliases for the tables that are more descriptive and shorter:
``` sql
SELECT * FROM table_name1 AS A
INNER JOIN table_name2 AS B
ON A.join_column1 = B.join_column2
```

Note that you can give aliases to tables without `AS`. Furthermore, once you define aliases, you should only refer to the table using aliases everywhere in the query.

### c) Selecting Columns with Name Overlap

If we want to select the column that appears both in the table, we need to specify the corresponding table name.

``` sql
SELECT A.join_column, column_name2 FROM table_name1 AS A
INNER JOIN table_name2 AS B
ON A.join_column = B.join_column
```

### d) Filtering with `ON` vs `WHERE`

You can apply filtering for the joined table by two methods, either using `ON` or using `WHERE`. Let's look at the examples below:

``` sql
SELECT * FROM table_name1 AS A
INNER JOIN table_name2 AS B
ON A.join_column = B.join_column 
AND A.column_name1 = B.column_name2
```

``` sql
SELECT * FROM table_name1 AS A
INNER JOIN table_name2 AS B
ON A.join_column = B.join_column 
WHERE A.column_name1 = B.column_name2
```

These two both filter the joined table. While the above two examples output the same result for `INNER JOIN`, they output different results for `LEFT JOIN`. The subtleties will be explain further later. Note that filtering with `ON` is more cost-efficient than filtering with `WHERE`.

### e) Joining multiple tables

Finally, you can join multiple tables. For example, if you are joining three tables, use the following syntax:
``` sql
SELECT * FROM table_name1 tb1
INNER JOIN table_name2 tb2
ON tb1.column_name1 = tb2.column_name2
INNER JOIN table_name3 tb3
ON tb3.column_name3 = tb1.column_name4 AND tb1.column_name5 != tb2.column_name1
```
This can be pretty complicated. Use clear aliases and indentation to stay organized.

## 2. FULL OUTER JOIN

*FULL OUTER JOIN* combines **all rows** from both tables. In contrast to `INNER JOIN`, which only includes matching rows, `FULL OUTER JOIN` includes **matching + non-matching** rows.

The basic syntax is as follows:

```sql
SELECT * FROM table_name1
FULL OUTER JOIN table_name2
ON table_name1.column_name1 = table_name2.column_name2
```

- Similar to `INNER JOIN`, `FULL OUTER JOIN` is symmetric, so `table_name1` and `table_name2` can be swapped.
- Rows that **don’t have matches** will have `NULL`s in place of missing values.

### a) Filter: Only in One Table

A common use case of `FULL OUTER JOIN` is to filter out rows that are only present in one of the tables. For example, if you want row only in `table_name1`, use the following code:

``` sql
SELECT * FROM table_name1 t1
FULL OUTER JOIN table_name2 t2
ON t1.column_name1 = t2.column_name2
WHERE t2.any_column IS null
```

You can use this pattern to **simulate a "diff"** between two datasets.

Some databases, e.g. MySQL, doesn't support `FULL OUTER JOIN` natively. In those cases, you can perform `FULL OUTER JOIN` by using `UNION`, `LEFT JOIN`, and `RIGHT JOIN`. We will cover `UNION` in detail in a [SQL 9. UNION and Subqueries](/posts/data_eng/sql/sql9).

## 3. LEFT OUTER JOIN (LEFT JOIN)
*LEFT JOIN* covers all rows from first table and **matching** rows from the second table.

The syntax is as follows:
``` sql
SELECT * FROM table_name1 tb1
LEFT JOIN table_name2 tb2
ON tb1.column_name1 = tb2.column_name2
```
Here, `table_name1` is the first table and `table_name2` is the second table.

As mentioned in the [section 1.d](#d-filtering-with-on-vs-where), filtering behaviours of `ON` and `WHERE` will be different if we you `LEFT JOIN`. Let's look at some examples.

``` sql
-- Option 1: Filter in ON
SELECT *
FROM orders
LEFT JOIN customers
ON orders.customer_id = customers.id AND customers.country = 'USA';

-- Option 2: Filter in WHERE
SELECT *
FROM orders
LEFT JOIN customers
ON orders.customer_id = customers.id
WHERE customers.country = 'USA';
```
- Option 1 will join only USA customers and still keep orders **with no matching customers**.
- Option 2 will **NOT** keep orders **with no matching customers** because, after the `LEFT JOIN`, `courtry` column of orders with no matching customers will be `NULL` and they are filtered by the `WHERE` clause.

For visualization, suppose we have two following tables, i.e. `orders`, `customers`:

{{< table >}}

| order_id | customer_id |
|:----------:|:-------------:|
| 1        | 101         |
| 2        | 102         |
| 3        | 99    |
| 4        | 103         |
{{< /table >}}

Above: `orders`.

{{< table >}}

| id  | name  | country |
|:-----:|:-------:|:---------:|
| 101 | Alice | USA     |
| 102 | Bob   | Canada  |
| 103 | Carol | USA     |
{{< /table >}}

Above: `customers`.

The results of two filtering, Option 1(`ON`) and Option 2 (`WHERE`) are as follows:

{{< table >}}
| order_id | customer_id | id  | name  | country |
|:----------:|:-------------:|:-----:|:-------:|:---------:|
| 1        | 101         | 101 | Alice | USA     |
| 2        | 102         | NULL | NULL  | NULL    |
| 3        | 99        | NULL| NULL  | NULL    |
| 4        | 103         | 103 | Carol | USA     |
{{< /table >}}

Above: Result of filtering in the `ON` clause (Option 1).

---
{{< table >}}
| order_id | customer_id | id  | name  | country |
|:----------:|:-------------:|:-----:|:-------:|:---------:|
| 1        | 101         | 101 | Alice | USA     |
| 4        | 103         | 103 | Carol | USA     |
{{< /table >}}


Above: Result of filtering in the `WHERE` clause (Option 2).


## 4. RIGHT OUTER JOIN (RIGHT JOIN)
*RIGHT JOIN* covers all rows from second table and **matching** rows from the first table.

The syntax is as follows:
``` sql
SELECT * FROM table_name1 tb1
RIGHT JOIN table_name2 tb2
ON tb1.column_name1 = tb2.column_name2
```
Here, `table_name1` is the first table and `table_name2` is the second table.

The above is equivalent to
``` sql
SELECT * FROM table_name2 tb2
LEFT JOIN table_name1 tb1
ON tb1.column_name1 = tb2.column_name2
```
up to the change of ordering of output columns.


