---
title: "Keys in the table"
date: 2025-06-03
draft: true
ShowToc: true
tags: ["sql", "key", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

## 1. Primary Key

A *pimary key* is a column that uniquely identifies every row in a table. 

- There can be only **one primary key per table**. But, the primary key can be **composite**, i.e. it can consist of multiple columns.
- A primary key cannot contain `NULL`'s.
- A primary key must contain unique values.

You can define ia simple primary key as follows:

``` sql
CREATE TABLE employees (
  employee_id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  department_id INT
);
```
`employee_id` uniquely indentifies each row. `SERIAL` auto-increments. Instead, `GENERATED ALWAYS AS IDENTITY` for stricter SQL compliance.

For the composite primary key, look at the following example:
``` sql
CREATE TABLE enrollments (
  student_id INT,
  course_id INT,
  PRIMARY KEY (student_id, course_id)
);
```
While no single column is unique on its own, the combination of `student_id` and `course_id` is unique.