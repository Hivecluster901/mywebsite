---
title: "SQL 2. Filtering"
date: 2025-05-30
draft: true
ShowToc: true
tags: ["sql", "filtering", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

Here we go through other basic SQL commands for filtering:

## 1. WHERE
*WHERE* is used to filter the data in the output. It always comes after FROM. The syntax is:

``` sql
SELECT
column_name1, 
column_name2
FROM table_name1
WHERE condition
```

For example, we can filter out all columns where the amount is `10.99`.

``` sql
SELECT
*
FROM payment
WHERE amount = 10.99
```

Furthermore, we can filter out certain columns.

``` sql
SELECT
first_name,
last_name
FROM customer
WHERE first_name = 'ADAM'
```
Enclose the texts around `'`.
We can use other operators to filter the data, e.g. `>`,`<`, `<=`, `>=`, `!=` or `<>`, 

### a) IS NULL/IS NOT NULL

We can also filter out rows that have `NULL`'s for certain columns:

``` sql
SELECT
column_name1,
column_name2
FROM table_name1
WHERE column_name1 is null
```
To exclude `NULL`'s one can use `is not` command.

### b) AND/OR

You can use multiple conditions as well by using `AND` and `OR`:
``` sql
SELECT
column_name1,
column_name2
FROM table_name1
WHERE condition1
AND condition2
AND condition3
```

`AND` will always be processed before `OR`. **Thus, to correctly execute queries, sometimes you need to put parentheses.**

### c) BETWEEN ... AND ...

You can filter a range of values by using the `BETWEEN` keyword.
``` sql
SELECT
column_name1,
column_name2
FROM table_name1
WHERE column_name3 BETWEEN n AND m
```
where the range includes `n` and `m`. `n` and `m` can be numeric values and time. The opposite is possible by using `NOT BETWEEN`.

``` sql
SELECT
column_name1,
column_name2
FROM table_name1
WHERE column_name3 NOT BETWEEN n AND m
```
`n` and `m` are not included.

### d) IN

If you have a long list of values to filter, you can use the `IN` statement.

``` sql
SELECT
*
FROM table_name1
where column_name1 IN (val1, val2, ..., valn)
```

The negation of `IN` is possible by using `NOT IN`.

### e) LIKE/ILIKE
*LIKE* is used to filter by matching against a pattern. `_` is used for any single charater pattern while `%` is used for any sequence of characters. The example is shown below:

``` sql
SELECT 
*
FROM actor
WHERE first_name LIKE 'A%'
```
*LIKE* is case-senstive while *ILIKE* is case-insensitive.  

- `%A%`: A can be anywhere.
- `_A%`: The second character is A.
- `__A%`: The third character is A.

We can also do the negation of the above. For example, the below code filter everything that *doesn't* have the character A.

``` sql
SELECT 
*
FROM actor
WHERE first_name NOT LIKE '%A%'
```

## 2. Comments & Aliases
- *Comments* make code more readable and understandable. For single line comments, use `--`. For multiple line comments, use `/* */`
``` sql
-- This is a single-line comment.
/*
    This is a multiline comment.
*/
```
- *Aliases* rename columns **for this query**. In the database, the column name stays the same. This allows more descriptive name for the column. For example,
``` sql
SELECT
payment_id AS invoice_no
FROM payment
```

**Note that column alias cannot be used in a `WHERE`, or `HAVING` clauses due to the order of execution.**
