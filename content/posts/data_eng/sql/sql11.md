---
title: "SQL 11. Views & Data Manipulation"
date: 2025-06-08
draft: false
ShowToc: true
tags: ["sql", "data manipulation", "update", "delete", "views", "materialized views", "interview prep"]
categories: ["Tech", "Databases"]
---

If you're prepping for a SQL or data engineering interview, it’s essential to be fluent not only in writing queries but also in understanding the impact of commands like `UPDATE`, `DELETE`, and views (both virtual and materialized). This post covers practical examples and deepens your understanding of data manipulation and view management in PostgreSQL.

---

## 1. `UPDATE`: Modifying Existing Rows

To modify data in a table, use the `UPDATE` statement. Here's the general syntax:

``` postgresql
UPDATE <table_name>
SET <column_name> = <value>
WHERE <condition>;
```

### Example: Update a specific row

``` postgresql
UPDATE songs
SET genre = 'Pop music'
WHERE song_id = 4;
```

### Example: Dynamic update

```postgresql
UPDATE songs
SET price = song_id + 0.99;
```

### Example: Batch transformation

```postgresql
UPDATE customer
SET email = LOWER(email);
```

> **Interview Tip:** Always include a `WHERE` clause unless you intend to update all rows.

---

## 2. `DELETE`: Removing Rows

Deletes rows from a table. The syntax:

```postgresql
DELETE FROM <table_name>
WHERE <condition>;
```

### Examples

```postgresql
DELETE FROM songs
WHERE song_id = 4;
```

```postgresql
DELETE FROM songs
WHERE song_id IN (3, 4);
```

To delete **all rows**:

```postgresql
DELETE FROM songs;
```

To return deleted rows:

```postgresql
DELETE FROM songs
WHERE song_id IN (3, 4)
RETURNING *;
```

> **Interview Tip:** Know the difference between `DELETE` (can return rows, slower) and `TRUNCATE` (faster, cannot filter or return rows, and usually not transactional).

---

## 3. `CREATE TABLE ... AS`: Creating Derived Tables

Use this to create a new table from a query:

```postgresql
CREATE TABLE <table_name> AS
SELECT ...
```

### Examples

You can copy an existing table as follows:

```postgresql
CREATE TABLE customer_test AS
SELECT * FROM customer;
```

You can use more complext queries to create derived tables.
```postgresql
CREATE TABLE customer_anonymous AS
SELECT customer_id, initials
FROM customer
WHERE first_name LIKE 'C%';
```

> **Interview Tip:** This is a *snapshot*. Future changes to the source table do not affect the new table.

---

## 4. `CREATE VIEW`: Logical Abstractions Over Data

A **view** is a stored SQL query. It behaves like a table but doesn’t store data physically.

```postgresql
CREATE VIEW <view_name> AS
SELECT ...
```

### Example

```postgresql
CREATE VIEW customer_anonymous AS
SELECT customer_id, initials
FROM customer
WHERE first_name LIKE 'C%';
```

You can now query it like a table:

```postgresql
SELECT * FROM customer_anonymous;
```

### Benefits of Views

* No redundant storage
* Always reflects latest data
* Great for reusability and abstraction

### Limitations

* Performance overhead for complex queries
* Cannot always be updated directly (e.g. joins, aggregates)

### 4.1 What is an Updatable View?

You're right about one thing: **when data in the underlying tables change, views reflect those changes** automatically. That's because views **don’t store data**—they're just saved SQL queries.

But when we say a **view is (or isn't) updatable**, we’re talking about whether you can **run `INSERT`, `UPDATE`, or `DELETE`** statements **on the view itself**, not just query it.

#### Updatable View Example

Let’s say you create a very simple view:

```sql
CREATE VIEW simple_customer AS
SELECT customer_id, email
FROM customer;
```

You can update it like this:

```sql
UPDATE simple_customer
SET email = 'new@example.com'
WHERE customer_id = 5;
```

PostgreSQL allows this because:

* It's based on **a single table**
* There are **no joins**, **aggregates**, or **GROUP BY**
* All selected columns map clearly to base table columns

#### Not Updatable View Example

```sql
CREATE VIEW customer_order_summary AS
SELECT c.customer_id, c.name, COUNT(o.order_id) AS total_orders
FROM customer c
JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name;
```

This **cannot** be updated directly because:

* It includes a **join**
* It uses **aggregate function (`COUNT`)**
* There's no clear 1-to-1 mapping between the view’s rows and the base table’s rows

> **Rule of Thumb:** If PostgreSQL can’t unambiguously map each row in the view to a single row in a base table, it’s not updatable.

---

> **Interview Tip:** Understand *updatable views*. A simple view over a single table without aggregates or joins can typically be updated.

---

## 5. `CREATE MATERIALIZED VIEW`: Cached Results

A **materialized view** stores the result of a query physically on disk.

```postgresql
CREATE MATERIALIZED VIEW <view_name> AS
SELECT ...
```

### Benefits

* Faster access for expensive queries
* Good for read-heavy workloads

### Limitation

* Data is **not automatically updated**

To refresh the data:

```postgresql
REFRESH MATERIALIZED VIEW <view_name>;
```

> **Interview Tip:** Know when to use a materialized view vs. a regular view. The key tradeoff is **performance vs. freshness**.

---

## 6. Managing Views

You can modify or delete views and materialized views with the following commands:

### For Views

```postgresql
DROP VIEW <view_name>;

ALTER VIEW <view_name> RENAME TO <new_name>;

ALTER VIEW <view_name> RENAME COLUMN old TO new;

CREATE OR REPLACE VIEW <view_name> AS
SELECT ...;
```

### For Materialized Views

```postgresql
DROP MATERIALIZED VIEW <view_name>;

ALTER MATERIALIZED VIEW <view_name> RENAME TO <new_name>;
```

> **Interview Tip:** You **cannot** use `CREATE OR REPLACE` with materialized views. Instead, drop and recreate.

---

## Bonus: Common Interview Scenarios

* **Can views be indexed?**
  No, but materialized views can be indexed.

* **What happens when you update data used in a view?**
  The next time the view is queried, it reflects the change (unless it’s a materialized view).

* **How to persist calculated fields efficiently?**
  Use a materialized view with indexed columns.

* **Can I insert into a view?**
  Only if it’s updatable (i.e., directly maps to a single base table with no joins or aggregations).

* **Is DML allowed in a materialized view?**
  No, a materialized view is a **read-only cache** of a SQL query's result, and it is not a real table.
  In-depth answer: There are two reasons:
  * **Lack of Source Table Mapping**: materialized veiws don't track where each row came from in the base table. When DML commands are executed on materialized views, PostgreSQL will not have any idea on which table(s) the value should be written back to and how to propagate this change across joins, aggregations, or filters.
  * **Could be derived from complex queries**: this is the same issue for the Not Updatable Views.
  Due to these two reasons, you have to be okay with **stale data**.


### What is an Index, and Why Can't a View Be Indexed?
#### What Is an Index?

An **index** is like a **lookup table** for your database. It makes it much faster to find rows, similar to an index in a book.

For example, if you create this:

```postgresql
CREATE INDEX idx_customer_email ON customer(email);
```

Now PostgreSQL can find rows in the `customer` table by `email` much faster—without scanning every row.

#### Why Can’t You Index a View?

Because a **view is not a table and doesn’t store any data**, there is **nothing to index**. It’s just a stored SQL query that gets run each time you use it.

> Indexing only works on actual **stored** data (like tables and materialized views), not **virtual** data (like views).

#### But Materialized Views *Can* Be Indexed

```postgresql
CREATE MATERIALIZED VIEW top_customers AS
SELECT customer_id, SUM(total) AS total_spent
FROM orders
GROUP BY customer_id;

CREATE INDEX idx_total_spent ON top_customers(total_spent);
```

Since materialized views store actual rows, indexes help you query them faster—just like a regular table.

---

## Summary Table
{{<table>}}
| Feature                 | View      | Materialized View    | Table |
| ----------------------- | --------- | -------------------- | ----- |
| Stored Physically       | X         | O                    | O     |
| Reflects Source Changes | O         | O (but needs `REFRESH`)  | X     |
| Query Performance       | Moderate  | Fast (after refresh) | Fast  |
| Can Be Indexed          | X         | O                    | O     |
| Automatically Updated   | O         | X                    | X     |
| Supports Insert/Update  | Sometimes | No                   | Yes   |
{{</table>}}
---
