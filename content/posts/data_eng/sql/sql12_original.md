---
title: "SQL 12. Window Functions"
date: 2025-06-09
draft: true
ShowToc: true
tags: ["sql", "window function", "over", "rank", "first_value", "lead", "lag", "interview prep"]
categories: ["Tech", "Databases"]
---

## 1. OVER(PARTITION BY ...)

Window functions allow us to calculate aggregated data and display them alongside the original data without affecting the number of rows. 

This can also be achieved by using correlated subqueries in `SELECT`. The example is as follows:

``` postgresql
SELECT
transaction_id,
payment_type,
customer_id,
price_in_transaction,
(SELECT SUM(price_in_transaction)
FROM sales s2
WHERE s1.customer_id = s2.customer_id)
FROM sales s1
```

The above command calculates the total spending of each customer and display them for each row of the `sales` table.

This is not only complicated but also very cost-intensive performance-wise. This is because we have to iterate through every single row. T

So use window function instead! The syntax is as follows:

``` postgresql
AGG(agg_column) OVER(PARTITION BY partition_column)
```
We have an aggregation function with the column that we want to aggregate. Then, we use the `OVER` keyword to oper up our, so-called, window. We define what should be partitioned by.
`(PARTITION BY partition_column)` is called a window because
we don't group the data really, we just open a window of aggregation.
``` postgresql
SELECT
transaction_id,
payment_type,
customer_id,
price_in_transaction,
SUM(price_in_transaction) OVER(PARTITION BY customer_id)
FROM sales s
```

The number of rows are unaffected, we can still use all columns in the same level of detail, and we have an addition column that is aggregated by a certain partition. Furthermore, when we use a window function, the result is returned in the order of the column that the table was partitioned by.

You can also partition by multiple columns:

``` postgresql
SELECT
*,
COUNT(*) OVER(PARTITION BY customer_id, payment_type)
FROM sales s
```

The above command aggregates the number of transaction by each customer for each payment type.

Furthermore, to query the total number of transaction, one can do as follows:

``` postgresql
SELECT
*,
COUNT(*) OVER()
FROM sales s
```

Note that, when you want to round the aggregated results, you have to act `ROUND` for the whole expression. For example,

``` postgresql
SELECT
*,
ROUND(AVG(amount) OVER(), 2)
FROM sales s
```

## 2. OVER(ORDER BY ...)

This command is often used to calculate the running total of some column over another column. For example, one can calculate the running total of the amount of payment over the payment dates.

``` postgresql
SELECT
*,
SUM(amount) OVER(ORDER BY payment_date)
FROM sales s
```

Another interesting use case is to combine it with `PARTITION BY`:

``` postgresql
SELECT
*,
SUM(amount) OVER(PARTITION BY customer_id ORDER BY payment_date)
FROM sales s
```

The above command calculates the running total of the payment amount over the payment date grouped by the `customer_id`. Also, you can order by more than one columns.

``` postgresql
SELECT
*,
SUM(amount) OVER(PARTITION BY customer_id ORDER BY payment_date, payment_id)
FROM sales s
```

## 3. RANK/DENSE_RANK
`RANK` and `DENSE_RANK` is used to rank the rows ordered by a certain column. The difference between the two is that `DENSE_RANK` gives you ranks sequentially even though are there multiple rows that have the same rank.

The syntax is as follows:
``` postgresql
RANK() OVER(ORDER BY column_name1)
```

The above calculates the rank of each row by the column `column_name1`. One can also use `PARTITION BY` to parition the table by a certain column:

``` postgresql
RANK() OVER(PARTITION BY column_name2 ORDER BY column_name1)
```

For example, let's say we want to calculate the ranking of movies by their running time. Then, we can use the following:
``` postgresql
SELECT 
*,
RANK() OVER(ORDER BY length)
FROM film
```

Furthermore, if you want to calculate the rank of the film by the length of film for each category, you need to use `PARTITION BY`:

``` postgresql
SELECT
*,
RANK() OVER(PARTITION BY category ORDER BY length)
FROM film
```

One thing to note is that it is impossible to use those functions in a `WHERE` filtering. This is impossible because `WHERE` filtering takes place before the window function is evaluated. So, if we want to filter by the any window function, we need to use subqueries. For example, if you want to obtain only the movies that ranked first in terms of its length from each category, one can use the following command:

``` postgresql
SELECT
*
FROM(
    SELECT
    *,
    RANK() OVER(PARTITION BY category ORDER BY length) AS "Rank"
    FROM film
)
WHERE "Rank" = 1
```

You can use `RANK` inside the `GROUP BY` as well. For example, if you first want to calculate the total number of payments made by each customer in the payment table and rank by the number of payments, you can do as follows:
``` postgresql
SELECT
customer_id,
COUNT(*),
RANK() OVER(ORDER BY COUNT(*) DESC)
FROM payment
GROUP BY customer_id
```

## 4. FIRST_VALUE

`FIRST_VALUE` displays the value of a column that corresponds to rank 1 in terms of the criterion column. For example,

``` postgresql
SELECT 
*,
FIRST_VALUE(title) OVER(ORDER BY length)
FROM film
```
The above command displays the title of the movie that ranked first in term of its length.

``` postgresql
SELECT
customer_id,
COUNT(*),
FIRST_VALUE(COUNT(*)) OVER(ORDER BY COUNT(*)),
COUNT(*) - FIRST_VALUE(COUNT(*)) OVER(ORDER BY COUNT(*)) AS excess
FROM
payment
GROUP BY customer_id
```

The above command returns the lowest number of payment among all customers, and the excess number of payment made by each customer compared to that lowest number of payment.

## 5. LEAD & LAG

`LEAD` gives us the next value ordered by a column.
For example,

``` postgresql
SELECT
customer_id,
COUNT(*),
LEAD(COUNT(*)) OVER(ORDER BY COUNT(*)),
FROM
payment
GROUP BY customer_id
```

This gives you the next count value after the current one ordered by the count value.

`LAG` gives you the previous value ordered by a column.

`LEAD`and `LAG` are useful in calculating the differences between the next value and the current value and between the current value and the previous value. So, if we want to calculate the difference or growth compared to, for example, the last year, we can use this kind of operations. For example, suppose that we want to calculate the difference and percentage growth of sales from the previous day. Then,

``` postgresql
SELECT
SUM(amount),
DATE(payment_date) AS "day",
LAG(SUM(amount)) OVER(ORDER BY DATE(payment_date)) AS previous_day,
SUM(amount) - LAG(SUM(amount)) OVER(ORDER BY DATE(payment_date)) AS differences,
(SUM(amount) - LAG(SUM(amount)) OVER(ORDER BY DATE(payment_date))) / LAG(SUM(amount)) OVER(ORDER BY DATE(payment_date)) * 100 AS percentage_growth
FROM payment
GROUP BY "day"
```