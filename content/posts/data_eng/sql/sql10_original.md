---
title: "SQL 10. Managing Tables & Databases"
date: 2025-06-06
draft: true
ShowToc: true
tags: ["sql", "table", "create", "drop", "foreign key", "primary key", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

So far, we only looked at querying from the tables. In this post, we are going to look at how we can manage the tables. In this post, we are going to cover **data definition**: dealing with the properties of the tables and database objects, i.e. dealing with data structures. The corresponding operations are how to create, alter, drop objects, such as database, table, and schemas.

On the other hand, there is also data manipulation: dealing with data itself. The corresponding operations are How to insert, update, and delete rows in a table. These will be covered in later posts.

Aside from these, there are also some other important concepts in table management: Data Types, Constraints, Primary & Foreign Keys, and Views. The first three will be covered in this post and views are going to be covered in later posts.

## 1. Data Types

It is important to understand the data types when creating the table. We are going to look at storage size, allowed values, possible operations for each data type. And, we are also going to look at which one to use for some examples. 

There are many data types, but notably Numeric, Strings, Date/Time, and others. Please refer to the [official documentation](https://www.postgresql.org/docs/current/datatype.html).

### 1. Numeric Data Types
{{<table>}}
|Type|Storage Size|Range|Notes|
|:----:|:----:|:----:|:-----:|
|INT|4 bytes|`-2^31` to `2^31-1`| Typical Choice|
|SMALLINT| 2bytes|`-2^15` to `2^15-1`| Small integers|
|BIGINT|8 bytes|`-2^63` to `2^63-1`|Large integers|
|NUMERIC/ DECIMAL| variable | up to `131072` digits <br> before the decimal point; <br> up to `16383` digits <br> after the decimal point| user-defined <br> precision|
|SERIAL| variable | `1` to `2^31`| autoincrementing <br> integer|
{{</table>}}

We use the data type `NUMERIC`/`DECIMAL` when we have decimal places. 

``` sql
NUMERIC(precision, scale)
--precision: total count of digits
--scale: count of decimal places
-- e.g. 24.99 -> numeric(4, 2)
```

`SERIAL` is often used for primary key.

### 2. String Data Types

{{<table>}}
|Type|Storage Size|Example|Notes|
|:----:|:----:|:----:|:-----:|
|character varying(n), <br> varchar(n)|variable-length with limit|Any text, <br> "Hello"| |
|character(n), <br> char(n)| fixed-length, blank padded|"M" or "F"|Space-padded|
|text|variable unlimited length|Any text, "Hello"||
{{</table>}}

`character(n)` is not better in performance due to space paddings, and it is only used if there is only one character to be stored. Thus, in most cases, people use either `VARCHAR(n)` or `text`, and `text` is a little bit more flexible than `VARCHAR(n)`.

> How should we store ZIP codes or phone numbers, e.g. ZIP code: 0142, phone: 0049-234422?
Since these values don't have numerical meanings, we should store them as strings!

### 3. Date/time Data Types

{{<table>}}
|Type|Description|Example|
|:----:|:----:|:----:|
|date|Just date without time|'2022-11-28'| 
|time (with/without time zone)| Just time without date|'01:02:03.678'|
|timestamp (with/without time zone)|Date and time|'2022-11-28 01:02:03.678+02'|
|intervals|Time interval|'3 days 01:02:03.678'|
{{</table>}}

### 4. Other Data Types

{{<table>}}
|Type|Description|Example|Range|
|:----:|:----:|:----:|:-----:|
|boolean|state of true or false|is_in_stock| `TRUE`, `FALSE`, `NULL`|
|enum| A value of a list of ordered values|movie_rating|User-defined|
|array|Stores a list of values|text[] or int[]| Depending on type|
{{</table>}}

The `enum` data type only allows certain values:
``` sql
CREATE TYPE mppa_rating AS ENUM('G', 'PG', [...])
```
Also, when we order a table by the specific column of type enum, the column will be ordered by the order in the definition. For example, if we order by `movie_rating` of type `mppa_rating`, the order will be 'G' 'PG', and so on.

Array column stores multiple values. For example, a person can have multiple phone numbers. Then, one can get all numbers of all people as 
``` sql
SELECT name, phone FROM customers
```

If you only want to get the first value of the array for all people, you can do

``` sql
SELECT name, phone[1] FROM customers
```
When you do the filtering, you can do achieve that using the `ANY` keyword. For example, if you want to get the name and the first phone number of a person who has a phone number, `+44-66764453`, you can run the following command:
``` sql
SELECT name, phone[1] FROM customers
WHERE '+44-66764453' = ANY(phones)
```

## 2. CONSTRAINTS
When you define a table, you need to define the column names, data types of the columns, and constraints. Constraints are defined when a table is created and used to define rules for the data in a table. This will prevent insert of invalid data. You can have constraints either in the columns or in the table.

Examples of column constraints are:
- `NOT NULL`: Ensures that a column cannot have a `NULL` value. 
- `UNIQUE`: Ensures that all values in a column are different.
- `DEFAULT`: Sets a default value for a column if no value is specified.
- `PRIMARY KEY`: A combination of a `NOT NULL` and `UNIQUE`. Uniquely identifies each row in a table.
- `REFERENCES`: Ensures referential integrity (Allow only values that are already in the referenced column in another tabel). If we try to insert something that violates the `REFERENCES` constraint, we get an error, saying violation of foreign key constraint.
- `CHECK`: Ensures that the values in a column satisfies a specific condition. For example, we can set a condition that only values from 10 to 100 can be inserted.

We can define the above constraints on a table level. For example, if we want to define a primary key on multiple columns,

``` sql
PRIMARY KEY (column [, ...])
UNIQUE (column [, ...])
CHECK(search_condition)
```

## 3. CHECK

`CHECK` limits the value range that can be placed in a column.

``` sql
CREATE TABLE <table_name> (
    <column_name> TYPE1 CHECK(condition)
)
```

For example, we want the `name` column in `director` table to have length bigger than 1.
``` sql
CREATE TABLE director (
    name TEXT CHECK (length(name) > 1)
)
```

We can add the name of the constraint by adding `CONSTRAINT` keyword:
``` sql
CREATE TABLE director (
    name TEXT CONSTRAINT name_length CHECK(length(name) > 1)
)
```

However, for every constraint we introduce, there is a default name assigned. In this case, it will be `<table>_<column>_check`, i.e. `director_name_check`. Note that the column name comes from the name of column used in the condition. For example,

``` sql
CREATE TABLE director (
    name TEXT,
    date_of_birth DATE,
    start_date DATE,
    end_date DATE CHECK(start_date > '01-01-2000')
)
```
Furthermore, if we have two different columns in the `CHECK` constraint, the default becomes just `<table>_check`. E.g. 

``` sql
CREATE TABLE director (
    name TEXT,
    date_of_birth DATE,
    start_date DATE,
    end_date DATE CHECK(end_date > start_date)
)
```
`director_check`.

Then, the default name of the `CHECK` constraint will be `director_start_date_check`. You can use any columns in the `CHECK` constraint, and this means that `CHECK` constraints are table constraints but not column constraints.

You can add `CHECK` constraints as follows:

``` sql
ALTER TABLE director
ADD CONSTRAINT date_check CHECK(start_date > date_of_birth)
```

To drop the constraint,

``` sql
ALTER TABLE director
DROP CONSTRAINT date_check
```

Dropping first and adding a new one is a standard way to alter the `CHECK` constraint. We can rename the constarint:

``` sql
ALTER TABLE director
RENAME CONSTRAINT date_check TO data_constraint
```


## 4. Primary Key & Foreign Key

A primary key is one or multiple columns that uniquely identify each row in a table. To uniquely identify a row in a table, the primary key should satisfy two constraints: `UNIQUE`, `NOT NULL`.

A foreign key is one or multiple columns that refers to the primary in another table. We refer to the table that has the foreign key, the *referencing table* or *child* table. The referenced table is *referenced table* or *parent table*.

Notes:
- Foreign key does not need to be unique.
- Primary key and foreign keys are usually the columns to join tables.
- Primary key and foreign keys can be created als in table creation process.

## 5. Data Definition

### 1. CREATE DATABASE

The syntax is as follows:

``` sql
CREATE DATABASE database_name;
```

Note that we can define a few properties of the database with the `WITH` keyword:

``` sql
CREATE DATABASE database_name
    WITH encoding = 'UTF-8'; --character encoding
```

Here, we defined the character encoding. Furthermore, we can add
comments of the database:

``` sql
COMMENT ON DATABASE database_name IS 'That is our database.'
```

### 2. DROP DATABASE

Similarly, we can delete the database:

```sql
DROP DATABASE database_name;
```

### 3. CREATE TABLE

We use these commands to define tables in the database. The syntax is as follows:

``` sql
CREATE TABLE table_name1 (
    column_nmae1 TYPE1 [CONSTRAINT],
    column_nmae2 TYPE2 [CONSTRAINT],
    column_nmae3 TYPE3 [CONSTRAINT]
)
```
For example, if we create a staff table,
``` sql
CREATE TABLE staff (
    staff_id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL
    UNIQUE(name, staff_id)
)
```

For example, if we want to create an online sales table:

``` sql
CREATE TABLE online_sales (
	transaction_id SERIAL PRIMARY KEY,
	customer_id INT REFERENCES customer(customer_id),
	film_id INT REFERENCES film(film_id),
	amount NUMERIC(5, 2) NOT NULL,
	promotion_code VARCHAR(10) DEFAULT 'None'
)
```

### 4. INSERT

After creating a table, we need to insert rows to the table. The syntax is as follows:

``` sql
INSERT INTO table_name1
VALUES (value1, value2, [,...])
```

For example, one can insert a row in the `online_sales` table as follows:
``` sql
INSERT INTO online_sales
VALUES (1, 269, 13, 10.99, 'BUNDLE2022')
```
The data type must match the column that we have defined amd the constraints that we have defined needs to be fulfilled and cannot be violated.

Sometimes, we only want to enter specific values in specific columns, and we don't want to enter anything to the other columns. Then, right after the table name, we mention the columns that we want to insert values to. The values must be in the same order as we have mentioned them above, and then the values will be entered accordingly. For example,

``` sql
INSERT INTO online_sales
(customer_id, film_id, amount)
VALUES (235, 11, 20.99)
```
In the above case, the `transaction_id` auto-increments to 2. The `promotion_code` has the default value, `None`.

``` sql
INSERT INTO online_sales
(customer_id, amount)
VALUES (125, 10.99)
```

In the above case, the `transaction_id` auto-increments to 3. The `film_id` will be `NULL`. The `promotion_code` has the default value, `None`.

In contrast to the above two commands, the command below violates the `NOT NULL` constraint and throws an error message:

``` sql
INSERT INTO online_sales
(customer_id)
VALUES(247)
```

If we want to add multiple rows, we add the next set of values by adding new parentheses:

``` sql
INSERT INTO online_sales
(customer_id, film_id, amount)
VALUES (269, 13, 10.99), (235, 11, 20.99)
```
Suppose we delete all rows of the `online_sales` table and insert two rows by the above command. The primary key, `transaction_id`, are 4 and 5 because the `SERIAL` keeps auto-incrementing from the last value that was inserted even though we deleted some rows. This is a nice property of `SERIAL` as we can see whether some values were deleted. Or, if an attempt of inserting a row didn't work, `SERIAL` still auto-increment by 1. 

This can be reset back to 1 as follows:

``` sql
ALTER SEQUENCE <table_name>_<column_name>_seq RESTART WITH 1
```

### 5. ALTER TABLE

We can modify the properties of a table. We can add or delete columns, add or drop constraints, rename columns or table, and alter data types. The syntax is as follows:
``` sql
ALTER TABLE table_name1
ALTER_ACTION -- E.g. DROP COLUMN <column_name>
```

For example, you can drop column named `first_name` from the `staff` table if the column exists.

``` sql
ALTER TABLE staff
DROP COLUMN IF EXISTS first_name
```

We can add a column as follows. Note that we need to specify the data type.

``` sql
ALTER TABLE staff
ADD COLUMN IF NOT EXISTS date_of_birth DATE
```

We can change the data type of the column:
``` sql
ALTER TABLE <table_name>
ALTER COLUMN <column_name> TYPE NEW_TYPE
```

For example,
``` sql
ALTER TABLE staff
ALTER COLUMN address_id TYPE SMALLINT
```

We can also rename the column:
``` sql
ALTER TABLE staff
RENAME COLUMN first_name TO name
```

We can also rename the table:
``` sql
ALTER TABLE <table_name>
RENAME TO <new_table_name>
```

We can also add constraints:
``` sql
ALTER TABLE <table_name>
ALTER COLUMN <column_name> SET DEFAULT <value>
```

``` sql
ALTER TABLE <table_name>
ALTER COLUMN <column_name> SET NOT NULL
```

Note that you need additional `SET` keyword to set additional constraints. Furthermore, you can drop the constraints by using `DROP`:

``` sql
ALTER TABLE <table_name>
ALTER COLUMN <column_name> DROP DEFAULT
```

``` sql
ALTER TABLE <table_name>
ALTER COLUMN <column_name> DROP NOT NULL
```

You can also add able constraints:

``` sql
ALTER TABLE <table_name>
ADD CONSTRAINT <constraint_name> UNIQUE(column_name1 [,...])
```

We *don't have to* specify the `<constraint_name>` like this:
``` sql
ALTER TABLE <table_name>
ADD UNIQUE(column_name1 [,...])
```

But it is a good practice to do for clarity and future maintenance.

You can also add primary keys:
``` sql
ALTER TABLE <table_name>
ADD CONSTRAINT <constraint_name> PRIMARY KEY(column_name1 [,...])
```

Note that you can have multiple of these action steps in one alter table command, apart from the `RENAME`. Use `RENAME` in a single step.

### 6. TRUNCATE and DROP TABLE

`DROP TABLE` deletes table. With `DROP`, one can delete any database objects, e.g. `DROP DATABASE`, `DROP SCHEMA`.

Syntax is as follows:
``` sql 
DROP TABLE <table_name> -- Deletes table
DROP <object> <object_name> -- Deletes object
```
We need to be careful because we cannot get this object back once it is dropped from the database.

`TRUCNATE` deletes all data in table.

``` sql
TRUNCATE [TABLE] <table_name>
```

You don't have to write `TABLE` because `TRUNCATE` only works for tables.


