---
title: "SQL 9. Union and Subqueries"
date: 2025-06-03
draft: true
ShowToc: true
tags: ["sql", "union", "union all", "join",  "subquery", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---


## 1. UNION/UNION ALL
In contrast to [*JOINS*](/posts/data_eng/sql/sql8), *UNION* combines rows instead of columns. 

### a) Basic Syntax

The syntax is as follows:
``` sql
SELECT column_name1, column_name2 FROM table_name1
UNION
SELECT column_name3, column_name4 FROM table_name2
```
- The number of columns from the first table should match the number of columns from the second table.
- Columns are matched by the order. For example `column_name1` and `column_name3` don't need to be identitcal. The merged table will have column names of the first table, i.e. `column_name1` and `column_name2`. 

- The data types of the matching columns should be the same. The data type of `column_name1` should match that of `column_name3`.

- Duplicates are decoupled. Whenever there is a duplicate row from the two tables, it only appears once in the output table. If you want to keep all duplicates, you can use `UNION ALL` instead of `UNION`.

``` sql
SELECT column_name1, column_name2 FROM table_name1
UNION ALL
SELECT column_name3, column_name4 FROM table_name2
```

### b) A common use case

When we perform `UNION` or `UNION ALL`, we often want to know from which table the particular row come from. Then, we can do this by

``` sql
SELECT column_name1, column_name2, 'table_name1' AS origin FROM table_name1
UNION ALL
SELECT column_name3, column_name4, 'table_name2' FROM table_name2
```

We can give alias `origin` to the third column by giving the alias to the third column of the first table only.

### c) FULL OUTER JOIN using UNION

From [SQL 8. Joins](/posts/data_eng/sql/sql8), we briefly metioned that some databases don't natively  support `FULL OUTER JOIN`. We note here that this can be implemented as below:

``` sql
SELECT * 
FROM table_name1 t1
LEFT JOIN table_name2 t2
ON t1.col = t2.col

UNION

SELECT * 
FROM table_name1 t1
RIGHT JOIN table_name2 t2
ON t1.col = t2.col;
```

## 2. Subquery

A **subquery** (also called **inner query** or **nested query**) is a query nested within another query. Subqueries allow you to use the result of one query **as input** to another.

They are commonly used in:

- `WHERE` clauses (for filtering)
- `FROM` clauses (as derived tables or inline views)
- `SELECT` clauses (for claculated columns)

### a) Subqueries in WHERE

These subqueries filter rows based on the result of another query. Let's look at some examples.

``` sql
SELECT name
FROM employees
WHERE department_id = (
    SELECT id
    FROM departments
    WHERE name = 'Engineering'
)
```
The subqeury returns the `id` of the Engineering department. The outer query returns the names of all employeed in that department.

If there are multiple matches, use `IN` in the sub query.

``` sql
SELECT name
FROM employees
WHERE department_id = (
    SELECT id
    FROM departments
    WHERE name ILIKE '%s%'
)
```
The subquery returns the `id`'s of all departments that have `s` in their names. The outer query returns the names of  all employees in those departments.

### b) Subqueries in FROM

These queries query from a table derived from the nested query. Let's look at the example.
``` sql
SELECT AVG(lifetime_salary) AS avg_lifetime_salary
FROM (
    SELECT employee_id, SUM(amount) AS lifetime_salary FROM salary
    GROUP BY employee_id
) AS lifetime_salary
```
The subquery builds a derived table of lifetime salary. The outer query calculates the average lifetime salary of all employees.

From PostgreSQL 16, you don't have to provide the alias of the derived table.

### c) Subqueries in SELECT

These queries use subqueries to compute values per row. Let's look at the example.
```sql
SELECT 
name,
salary,
salary - (SELECT AVG(salary) FROM employees) AS excess
FROM employees
```
The query returns the excess amount of salary from the average salary of all employees. The subqeury returns the average salary of all employees.

## 3. Correlated Subquery

*Correlated subqueries* are subqueries that reference a column from the outer query:
- They cannot run independently without the outer query. 
- Subquery gets evaluated for every single row.

They are commonly used in:

- `WHERE` clauses (for **row-by-row filtering**, e.g. comparing each row to a dynamic value)
- `SELECT` clauses (for **row-wise computed columns**, like aggregate values related to the current row)

Note that the correlated subquery cannot be used in `FROM`.

### a) Correlated Subquery in WHERE

These are used when filtering rows in the outer query based on a condition involving a value derived from the subquery. Let's look at the example below:

``` sql
SELECT name, department_id, salary
FROM employees AS e1
WHERE salary > (
    SELECT
    AVG(salary)
    FROM employees AS e2
    WHERE  e1.department_id = e2.department_id
)
```

The subquery returns the average salary of the department for each row, i.e. `e1.department_id` depends on each row. Then, the outer query filters the rows that have higher salary than the average salary of the same department. 

### b) Correlated Subquery in SELECT

These are used to return **derived values** alongside each row - usually aggregate or summary statistics based on related data. Let's look at the example below:
``` sql
SELECT name,
    (SELECT
    COUNT(*)
    FROM employees AS e1
    WHERE e1.manager_id = m.id
    ) AS report_count
FROM employees AS m
WHERE role = 'Manager'
```

For each manager, the subquery counts the number of employees that are managed. Then, the outer query returns the name of the manager and the number of employees under management.

