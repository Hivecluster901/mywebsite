---
title: "SQL 10. Managing Tables & Databases"
date: 2025-06-06
draft: true
ShowToc: true
tags: ["sql", "data definition", "table", "create", "drop", "foreign key", "primary key", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

In previous posts, we focused on querying tables. In this post, we’ll explore **data definition** in SQL: how to create, modify, and delete database objects like tables, constraints, and databases themselves.

> Mastering these operations shows that you not only know how to *query* data, but also how to *design* and *manage* the structures that hold it.

We'll cover:

- Data types
- Constraints
- Primary and foreign keys
- Creating and altering tables and databases

> We'll cover **views**, **indexes**, and **data manipulation (INSERT, UPDATE, DELETE)** in separate posts.

---

## 1. Data Types

Choosing the correct data type ensures efficient storage and reliable data validation. PostgreSQL supports many types. Learn more from the [official documentation](https://www.postgresql.org/docs/current/datatype.html).

### 1.1 Numeric Data Types

{{<table>}}
| Type | Storage | Range | Notes |
|------|---------|-------|-------|
| `INT` | 4 bytes | `-2^31` to `2^31-1` | Common choice |
| `SMALLINT` | 2 bytes | `-2^15` to `2^15-1` | Small integers |
| `BIGINT` | 8 bytes | `-2^63` to `2^63-1` | Large integers |
| `NUMERIC(p, s)` | Variable | User-defined precision and scale | Exact values |
| `SERIAL` | 4 bytes | Auto-increments (1 to `2^31`) | Often used for IDs |
{{</table>}}

- Use `NUMERIC(p, s)` for money or precise decimal values.
- Use `SERIAL` for auto-incrementing primary keys.

```sql
NUMERIC(4, 2) -- Allows values like 24.99 (4 total digits, 2 after decimal)
```

### 1.2 String Data Types

{{<table>}}

| Type         | Description                        | Notes                      |
| ------------ | ---------------------------------- | -------------------------- |
| `VARCHAR(n)` | Variable-length string (limit `n`) | Use when length cap needed |
| `CHAR(n)`    | Fixed-length string, space-padded  | Rarely used                |
| `TEXT`       | Unlimited-length string            | Common and flexible        |

{{</table>}} 

> Store ZIP codes and phone numbers as `TEXT`, not integers — they are not meant for mathematical operations.

### 1.3 Date/Time Data Types

{{<table>}}

| Type         | Description   | Example                 |
| ------------ | ------------- | ----------------------- |
| `DATE`       | Date only     | `'2023-01-01'`          |
| `TIME`       | Time only     | `'13:45:30'`            |
| `TIMESTAMP`  | Date and time | `'2023-01-01 13:45:30'` |
| `INTERVAL`   | Time span     | `'2 days 3 hours'`      |
{{</table>}}

### 1.4 Other Data Types

{{<table>}}

| Type         | Description                      | Notes                     |
| ------------ | -------------------------------- | ------------------------- |
| `BOOLEAN`    | `TRUE`, `FALSE`, `NULL`          | Logical values            |
| `ENUM`       | Custom set of allowed values     | Enforce strict categories |
| `ARRAY`      | Multiple values of the same type | e.g., `TEXT[]`, `INT[]`   |
{{</table>}} 
```sql
CREATE TYPE mppa_rating AS ENUM('G', 'PG', 'PG-13', 'R', 'NC-17');
```

Arrays are powerful for storing multiple values, e.g. phone numbers:

```sql
SELECT name, phone[1] FROM customers
WHERE '+44-66764453' = ANY(phone);
```

---

## 2. Constraints

Constraints enforce rules at the column or table level:

* `NOT NULL`: Value must be provided.
* `UNIQUE`: No duplicate values allowed.
* `DEFAULT`: Value used if none provided.
* `PRIMARY KEY`: Uniquely identifies each row (`NOT NULL + UNIQUE`).
* `FOREIGN KEY`: Ensures values exist in another table.
* `CHECK`: Ensures values meet a condition.

### Example: Column-level

```sql
name TEXT NOT NULL DEFAULT 'Unknown'
```

### Example: Table-level

```sql
PRIMARY KEY (id, version)
UNIQUE (email)
CHECK (price > 0)
```

---

## 3. CHECK Constraints

`CHECK` limits the values inserted into a table.

### Example (Column-level):

```sql
CREATE TABLE director (
    name TEXT CHECK (length(name) > 1)
);
```

### With custom name:

```sql
CREATE TABLE director (
    name TEXT CONSTRAINT name_length CHECK (length(name) > 1)
);
```

### Table-level:

```sql
CHECK (end_date > start_date)
```

### ALTER to add:

```sql
ALTER TABLE director
ADD CONSTRAINT date_check CHECK (start_date > date_of_birth);
```

### ALTER to drop or rename:

```sql
ALTER TABLE director DROP CONSTRAINT date_check;
ALTER TABLE director RENAME CONSTRAINT date_check TO valid_dates;
```

---

## 4. Primary Keys & Foreign Keys

### Primary Key

* Uniquely identifies each row
* Implicitly `NOT NULL` + `UNIQUE`

```sql
CREATE TABLE staff (
    staff_id SERIAL PRIMARY KEY,
    name TEXT NOT NULL
);
```

### Foreign Key

* Points to a primary key in another table
* Enforces referential integrity

```sql
CREATE TABLE online_sales (
    transaction_id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customer(customer_id),
    film_id INT REFERENCES film(film_id)
);
```

> Foreign keys don’t need to be unique — multiple rows can reference the same parent.

---

## 5. Creating and Managing Databases & Tables

### CREATE DATABASE

```sql
CREATE DATABASE shop WITH ENCODING 'UTF8';
COMMENT ON DATABASE shop IS 'Main e-commerce database';
```

### DROP DATABASE

```sql
DROP DATABASE shop;
```

---

### CREATE TABLE

```sql
CREATE TABLE customers (
    customer_id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE
);
```

---

### INSERT INTO

```sql
INSERT INTO customers (name, email)
VALUES ('Alice', 'alice@example.com');
```

Insert multiple rows:

```sql
INSERT INTO customers (name, email)
VALUES ('Bob', 'bob@example.com'), ('Charlie', 'charlie@example.com');
```

> `SERIAL` values continue incrementing even after deletes.

Reset sequence:

```sql
ALTER SEQUENCE customers_customer_id_seq RESTART WITH 1;
```

---

### ALTER TABLE

Add a column:

```sql
ALTER TABLE staff ADD COLUMN birth_date DATE;
```

Rename column or table:

```sql
ALTER TABLE staff RENAME COLUMN name TO full_name;
ALTER TABLE staff RENAME TO employees;
```

Change data type:

```sql
ALTER TABLE staff ALTER COLUMN salary TYPE NUMERIC(10, 2);
```

Set or drop default:

```sql
ALTER TABLE staff ALTER COLUMN role SET DEFAULT 'Staff';
ALTER TABLE staff ALTER COLUMN role DROP DEFAULT;
```

Add constraints:

```sql
ALTER TABLE staff
ADD CONSTRAINT unique_email UNIQUE (email),
ADD CONSTRAINT staff_pk PRIMARY KEY (staff_id);
```

> You can add multiple constraints in a single `ALTER TABLE`, **except for `RENAME`**, which must be done separately.

---

### DROP TABLE vs. TRUNCATE

```sql
DROP TABLE orders;      -- Removes table structure + data
TRUNCATE TABLE orders;  -- Clears all rows, keeps structure
```

---

## Summary: Key Interview Concepts

{{<table>}}
| Concept                          | Must Know For Interviews      |
| -------------------------------- | ----------------------------- |
| `SERIAL` & sequences             | Auto-increment behavior     |
| `CHECK` constraints              | Column vs. table-level      |
| `ALTER TABLE` usage              | Syntax for modifying tables |
| `PRIMARY` / `FOREIGN` keys       | Relationship modeling       |
| `VARCHAR` vs `TEXT`              | When to use each            |
| `TRUNCATE` vs `DELETE` vs `DROP` | Effects on data and schema  |
{{</table>}}
---






