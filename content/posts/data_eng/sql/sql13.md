---
title: "SQL 13. Grouping Sets, Rollups, and Self-Joins"
date: 2025-06-11
draft: true
ShowToc: true
tags: ["sql", "grouping sets", "rollup", "cube", "self joins", "cross join", "natural join", "interview prep"]
categories: ["Tech", "Databases"]
---

## Overview

In SQL interviews, understanding advanced grouping and join techniques is crucial. This post dives into `GROUPING SETS`, `ROLLUP`, `CUBE`, and types of joins like self-joins, `CROSS JOIN`, and `NATURAL JOIN`. We also touch on how to interpret `NULL`s in grouping results and share interview tips along the way.

---

## 1. GROUPING SETS

`GROUPING SETS` let you define multiple `GROUP BY` combinations in a single query—ideal for multi-dimensional reports.

### Problem

You want to calculate:

- Revenue per **staff** per **month**
- Revenue per **month**
- Revenue per **staff**

Naively, you’d write:

```sql
SELECT NULL AS month, staff_id, SUM(amount)
FROM payment
GROUP BY staff_id

UNION

SELECT month, NULL AS staff_id, SUM(amount)
FROM payment
GROUP BY month

UNION

SELECT month, staff_id, SUM(amount)
FROM payment
GROUP BY month, staff_id
ORDER BY 1, 2;
```

### Better: Use `GROUPING SETS`

```sql
SELECT
    month,
    staff_id,
    SUM(amount) AS total_amount
FROM payment
GROUP BY GROUPING SETS (
    (staff_id),
    (month),
    (month, staff_id)
)
ORDER BY 1, 2;
```

### Interview Tip

* Explain how `GROUPING SETS` improves performance and code clarity.
* Mention that `NULL` in output may mean "total" for a dimension—distinguish from actual NULLs using the `GROUPING()` function.

---

## 2. CUBE & ROLLUP

Both are shortcuts for predefined grouping sets.

### 2.1 ROLLUP

Generates subtotals moving **top-down** in a hierarchy.

#### Syntax

```sql
GROUP BY ROLLUP (col1, col2, col3)
```

Equivalent to:

```sql
GROUP BY GROUPING SETS (
  (col1, col2, col3),
  (col1, col2),
  (col1),
  ()
)
```

#### Example: Revenue by Date → Month → Quarter

```sql
SELECT 
    'Q' || TO_CHAR(payment_date, 'Q') AS quarter,
    EXTRACT(month FROM payment_date) AS month,
    DATE(payment_date) AS date,
    SUM(amount) AS total_amount
FROM payment
GROUP BY ROLLUP (
    'Q' || TO_CHAR(payment_date, 'Q'),
    EXTRACT(month FROM payment_date),
    DATE(payment_date)
)
ORDER BY 1, 2, 3;
```

### 2.2 CUBE

Returns all **possible combinations** of columns.

#### Syntax

```sql
GROUP BY CUBE (col1, col2, col3)
```

Equivalent to:

```sql
GROUP BY GROUPING SETS (
  (col1, col2, col3),
  (col1, col2),
  (col1, col3),
  (col2, col3),
  (col1),
  (col2),
  (col3),
  ()
)
```

#### Example: Revenue by Customer, Staff, and Day

```sql
SELECT
    customer_id,
    staff_id,
    DATE(payment_date) AS payment_date,
    SUM(amount) AS total_amount
FROM payment
GROUP BY CUBE (
    customer_id,
    staff_id,
    DATE(payment_date)
)
ORDER BY 1, 2, 3;
```

---

## 3. Understanding `GROUPING()` Function

Use `GROUPING(col)` to distinguish real `NULL` from `NULL` added by rollups/cubes.

```sql
SELECT 
    customer_id,
    staff_id,
    GROUPING(staff_id) AS is_staff_grouped,
    SUM(amount)
FROM payment
GROUP BY CUBE (customer_id, staff_id);
```

* `GROUPING(col)` returns `1` if the column was aggregated (i.e., NULL due to subtotal), else `0`.

---

## 4. SELF JOIN

A *self-join* joins a table to itself—commonly used to query hierarchical relationships.

### Example: Employee & Manager Names

```sql
SELECT
    e.employee_id,
    e.name AS employee_name,
    m.name AS manager_name
FROM employee e
LEFT JOIN employee m
  ON e.manager_id = m.employee_id;
```

### Example: Manager of Manager

```sql
SELECT
    e.name AS employee_name,
    m1.name AS manager_name,
    m2.name AS manager_of_manager
FROM employee e
LEFT JOIN employee m1 ON e.manager_id = m1.employee_id
LEFT JOIN employee m2 ON m1.manager_id = m2.employee_id;
```

---

## 5. CROSS JOIN

Returns the Cartesian product—every row from one table paired with every row from another.

### Syntax

```sql
SELECT
    t1.column1,
    t2.column2
FROM table1 t1
CROSS JOIN table2 t2;
```

### Interview Tip

Mention potential performance risks—used in scenarios like generating all combinations (e.g., calendar x product).

---

## 6. NATURAL JOIN

Automatically joins on columns with the same name in both tables.

### Syntax

```sql
SELECT *
FROM table1
NATURAL JOIN table2;
```

### Caution

* Implicit behavior—may join on **unintended** columns.
* **Avoid in production code**, but know it for interviews.

---

## Summary for Interviews

{{<table>}}
| Concept         | Use Case                                       | Caution                           |
| :---------------: | :----------------------------------------------: | :---------------------------------: |
| `GROUPING SETS` | Combine multiple `GROUP BY` clauses into one   | Avoid duplicate logic via `UNION` |
| `ROLLUP`        | Hierarchical totals (e.g., Year → Month → Day) | Interpreting NULLs                |
| `CUBE`          | All combinations (no hierarchy)                | Can be expensive                  |
| `GROUPING()`    | Detect subtotal rows                           | Must use with ROLLUP/CUBE         |
| `SELF JOIN`     | Hierarchies (e.g. manager → employee)          | Alias carefully                   |
| `CROSS JOIN`    | All row combinations                           | Performance risk                  |
| `NATURAL JOIN`  | Auto-join on same column names                 | Unpredictable in large schemas    |
{{</table>}}
---

## Further Reading

* [PostgreSQL GROUPING SETS Docs](https://www.postgresql.org/docs/current/queries-table-expressions.html#QUERIES-GROUPING-SETS)
* [Self Joins](https://mode.com/sql-tutorial/sql-self-joins/)

---
