---
title: "SQL 13. Grouping sets, rollups, self-joins"
date: 2025-06-11
draft: true
ShowToc: true
tags: ["sql", "grouping sets", "rollup", "cube", "self joins", "cross join", "natural join", "interview prep"]
categories: ["Tech", "Databases"]
---

## 1. GROPUING SETS

`GROUPING SETS` allows us to define all of the combinations that our data should be grouped by in one query. For example, let's say we want to get the revenue obtained by each staff for each month, revenue obtained for each month, and the revenue obtained by each staff for the entire database in one table. These are combinations of many `GROUP BY`'s with different columns. This can certainly be achieved by using multiple `GROPU BY`'s and `UNION`'s. For the above example, one can execute the following code:

``` postgresql
SELECT
NULL AS month,
staff_id,
SUM(amount)
FROM payment
GROUP BY staff_id

UNION

SELECT
month,
NULL AS staff_id,
SUM(amount)
FROM payment
GROUP BY month

UNION

SELECT
month,
staff_id,
SUM(amount)
FROM payment
GROUP BY
month, staff_id
ORDER BY 1, 2
```

Here, we are grouping the table by `staff_id`, `month`, `staff_id` and `month`. Note that we need to make a `NULL` column for the grouped table by `staff_id` and `month` because one of the two columns will not be present for each gropued table. 

Though the above code achieves what we want, the above query is inefficient. Alternatively, we can use `GROUPING SETS` to achieve the same result.

``` postgresql
SELECT
month,
staff_id,
SUM(amount)
FROM payment
GROUP BY
    GROUPING SETS (
        (staff_id),
        (month),
        (staff_id, month) 
    )
ORDER BY 1,2
```

## 2. CUBE & ROLLUP 

We can use `CUBE` and `ROLLUP` to get a predefined grouping set.

### 1. ROLLUP
`ROLLUP` is used to create a hierarchy in the grouping set: If a set of columns are connected via one-to-many relationships, then this forms a natural hierarchy.

The syntax is as follows:

``` postgresql
GROUP BY 
ROLLUP(column1, column2, column3)
```
`column1` is at the top of the hierarchy, followed by `column2` and `column3`.

The above is equivalent to the code below:
``` postgresql
GROUP BY
GROUPING SETS (
    (column1, column2, column3),
    (column1, column2),
    (column1),
    ()
)
```

Suppose that you want to calculate the total revenue of a quarter, month, and date.
They form a hierarchy because a date should be in one month, and a month should be in one quater. We can use the following command to calculate those:

``` postgresql
SELECT 
'Q' || TO_CHAR(payment_date, 'Q') AS quarter,
EXTRACT(month from payment_date) AS month,
DATE(payment_date),
SUM(amount)
FROM payment
GROUP BY
ROLLUP ('Q' || TO_CHAR(payment_date, 'Q'), 
EXTRACT(month from payment_date), 
DATE(payment_date))
ORDER BY 1,2,3
```

### 2. CUBE

`CUBE` also gives you a predefined grouping set but for columns that are not strictly in a hierarchical order.
``` postgresql
GROUP BY 
CUBE (column1, column2, column3)
```

In this case, you get all possible combinations from the three columns:

``` postgresql
GROUP BY
GROUPING SETS (
    (column1, column2, column3),
    (column1, column2),
    (column1, column3),
    (column2, column3),
    (column1),
    (column2),
    (column3),
    ()
)
```

For example, suppose that you want to calculate the amount paid per customer per staff_id per day. There is no natural hierarchy between these three columns, i.e. they are connected many-to-many relationships. We can use the following command:
``` postgresql
SELECT
customer_id,
staff_id,
DATE(payment_date),
SUM(amount)
FROM payment
GROUP BY
CUBE(
customer_id,
staff_id,
DATE(payment_date)
)
ORDER BY 1,2,3
```

## 3. Self-join
*Self-join* is the join of the table with itself. The syntax is the same as the normal join except we have the same table appearing twice:
``` postgresql
SELECT
t1.column1,
t2.column2,
[, ...]
FROM table1 t1
LEFT JOIN table1 t2
ON t1.column1 = t2.column1
```

For example, if we have a table, `employee`, which has three columns, `employee_id`, `name`, and `manager_id`. The `manager_id` column is just referencing the `employee_id`.

To get the name of the manager for each employee, one need to do self-join:

``` postgresql
SELECT
e.employee_id, 
e.name,
m.name AS manager,
FROM employee e
LEFT JOIN employee m
ON e.manager_id = m.employee_id
```

You can also get the manager of managers:
``` postgresql
SELECT
e.employee_id,
e.name,
m1.name AS manager,
m2.name AS manger_of_manager
FROM employee e
LEFT JOIN employee m1
ON e.manager_id = m1.employee_id
LEFT JOIN employee m2
ON m1.manager_id = m2.employee_id
```
## 4. CROSS JOIN

With the `CROSS JOIN`, we get the cartesian product of two tables, i.e. all combinations of rows. 

The syntax is as follows:

``` postgresql
SELECT
t1.column1,
t2.column2
FROM table1 t1
CROSS JOIN table2 t2
```

Since this is a combination of all rows, but a combination of all values, you can have duplicate values.

## 5. NATURAL JOIN
`NATURAL JOIN` is like a normal join except it automatically joins tables using the columns of the same name. The synatx is as follows:

``` postgresql
SELECT 
* 
FROM table2
NATURAL LEFT JOIN table2
```

You have to make sure that `NATURAL JOIN` is using the right columns. If we have multiple columns with same names in both tables, natural join is confused and join the two tables using a wrong column.