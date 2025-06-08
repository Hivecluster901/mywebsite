---
title: "SQL 11. Views & Data Manipulation"
date: 2025-06-08
draft: true
ShowToc: true
tags: ["sql", "data manipulation", "table", "update", "delete", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

## 1. UPDATE

In the last post, we saw how to insert a new row to an existing table using `INSERT`, but what if those rows already exist, and we want to update values?

Use `UPDATE`. The syntax is as follows:
``` sql
UPDATE <table_name>
SET <column_name> = <value>
```

The above sets the value of the specified column to the new value for all rows. Thus, we combine the above with `WHERE`.


``` sql
UPDATE <table_name>
SET <column_name>=<value>
WHERE <condition>
```

For example, if you want to change one specific row, then use the primary key in the condition. For example, 
``` sql
UPDATE songs
SET genre='Pop music'
WHERE song_id=4
```

We can make this command more dynamic. For example,
``` sql
UPDATE songs
SET price = song_id + 0.99
```

``` sql
UPDATE customer
SET email = LOWER(email)
```

## 2. DELETE

`DELETE` deletes rows from a table. The syntax is as follows:

``` sql
DELETE FROM <table_name>
WHERE condition
```

For example,

``` sql
DELETE FROM songs
WHERE song_id = 4
```

For multiple rows,

``` sql
DELETE FROM songs
WHERE song_id IN (3,4)
```

To completely delete all rows in a table, simply use `DELETE` without the `WHERE` keyword:

``` sql
DELETE FROM songs
```

To see the information in the columns of the deleted rows, one can use the `RETURNING` keyword:
``` sql
DELETE FROM songs
WHERE song_id IN (3,4)
RETURNING *
```

## 3. CREATE TABLE ... AS

We saw from the [previous post](/posts/data_eng/sql/sql10) how to create a table and insert rows. Often time, we use `CREATE TABLE ... AS` command to create a table from an existing query. The syntax is as follows:

```sql
CREATE TABLE <table_name>
AS query
```

For example, we can create a copy of an existing table:

``` sql
CREATE TABLE customer_test
AS
SELECT * FROM customer
```

Or, we can make the query more sophisticated:

``` sql
CREATE TABLE customer_anonymous
AS
SELECT customer_id, initials
FROM customer
WHERE first_name LIKE 'C%'
```

## 4. CREATE VIEW

There are two problems on `CREATE TABLE ... AS`:

1. There is additional physical storage needed. The new table is physically stored in the database, and it is still redundant as we already have all necessary data of the underlying tables.

2. Also, data in underlying table can change. However, the change of data in the underlying table will not affect the data in the new table created before the change.

Thus, we want our table to be not redundant and dynamic.

Alternative: Create view and just store the statement. From the users perspective, it looks like a real table: one can query from the view. But the big difference is that the data will not be stored physically. But, we just store the logic, i.e. the statement that we have used. Whenever we query from the view, the underlying statements will be executed behind the scenes. If the underlying statements behind the view take long time, then it takes long to query from the view.

The syntax is as follows:

``` sql
CREATE VIEW <view_name>
AS query
```

For example, 
``` sql
CREATE VIEW customer_anonymous
AS 
SELECT customer_id, initials
FROM customer
WHERE first_name LIKE `C%`
```

Then, we can just query from the view as if it were a normal table:

``` postgresql
SELECT * FROM customer_anonymous
```

Due to the time it takes to execute the statements behind the view, the main use cases for views are queries that we use very frequently but they are not too complex.

So, sometimes, views may not be ideal: Then we can decide to create a new table, but the problem is that the table will not be updated if data in the underlying tables change.

## 5. CREATE MATERIALIZED VIEW

``` postgresql
CREATE MATERIALIZED VIEW <view_name>
AS query
```
With `MATERIALIZED VIEW`, we can combine the benefits of physically available table and a view in some way. Like physical table, since data is stored physically, performance is ensured. At the same, we can update the materialized views by using the `REFRESH` command:

``` postgresql
REFRESH MATERIALIZED VEIW <view_name>
```

The data will up to date in the materialized view.


## 6. Managing Views

Same with the table, we can alter the views/materialized views and drop views/materialized views.

For the views, we can use the following commands:
- `ALTER VIEW`
- `DROP VIEW`
- `CREATE VIEW`
- `CREATE OR REPLACE VIEW`

For the materialized views, we can use the following commands:
- `ALTER MATERIALIZED VIEW`
- `DROP MATERIALIZED VIEW`
- `CREATE MATERIALIZED VIEW`

For the materialized views, one cannot replace an already existing materialized view with the `REPLACE VIEW` command.

Examples:

``` postgresql
DROP VIEW <view_name>
```

``` postgresql
DROP MATERIALIZED VIEW <view_name>
```

``` postgresql
ALTER VIEW <view_name>
RENAME TO <new_view_name>
```

``` postgresql
ALTER VIEW <view_name>
RENAME COLUMN <column_name1> TO <new_column_name1>
```

``` postgresql
CREATE OR REPLACE VIEW <view_name>
AS new_query
```

This can also be used as a creation command. Note that this is not possible for tables and materialized views. For the `CREATE OR REPLACE VIEW` command, the new query must generate the same columns that were generated by the existing view query (that is, the same column names in the same order and with the same data types).