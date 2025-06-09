---
title: "SQL 12. Window Functions"
date: 2025-06-09
draft: false
ShowToc: true
tags: ["sql", "window function", "over", "rank", "first_value", "lead", "lag", "interview prep"]
categories: ["Tech", "Databases"]
---

Window functions in PostgreSQL allow you to perform calculations across sets of rows **related to the current row**, without collapsing rows like `GROUP BY` does. They’re essential for solving ranking, running totals, moving averages, and row-by-row comparisons.

This post reviews key window functions with practical, interview-friendly examples.

---

## 1. `OVER (PARTITION BY ...)`

Window functions let you run aggregations **without reducing the number of rows**.

A correlated subquery alternative:

```postgresql
SELECT
  transaction_id,
  payment_type,
  customer_id,
  price_in_transaction,
  (SELECT SUM(price_in_transaction)
   FROM sales s2
   WHERE s1.customer_id = s2.customer_id) AS total_by_customer
FROM sales s1;
```

This works but is inefficient. Instead, use:

```postgresql
SELECT
  transaction_id,
  payment_type,
  customer_id,
  price_in_transaction,
  SUM(price_in_transaction) OVER(PARTITION BY customer_id) AS total_by_customer
FROM sales;
```

### Notes:

* `PARTITION BY` defines the group of rows for aggregation.
* All original rows are preserved—no collapsing.
* You can partition by multiple columns:

```postgresql
SELECT
  *,
  COUNT(*) OVER(PARTITION BY customer_id, payment_type) AS customer_payment_type_count
FROM sales;
```

To apply the window to the entire table (no partitioning):

```postgresql
SELECT
  *,
  COUNT(*) OVER() AS total_rows
FROM sales;
```

To round window output:

```postgresql
SELECT
  *,
  ROUND(AVG(amount) OVER(), 2) AS avg_amount_rounded
FROM sales;
```

---

## 2. `OVER (ORDER BY ...)`

Adds ordering to your window function. Common for **running totals** or **cumulative aggregations**:

```postgresql
SELECT
  *,
  SUM(amount) OVER(ORDER BY payment_date) AS running_total
FROM sales;
```

You can combine both `PARTITION BY` and `ORDER BY`:

```postgresql
SELECT
  *,
  SUM(amount) OVER(PARTITION BY customer_id ORDER BY payment_date) AS customer_running_total
FROM sales;
```

Or even order by multiple columns:

```postgresql
SELECT
  *,
  SUM(amount) OVER(PARTITION BY customer_id ORDER BY payment_date, payment_id) AS detailed_running_total
FROM sales;
```

---

## 3. `RANK()` & `DENSE_RANK()`

Use these for ranking rows. The difference:

* `RANK()`: Skips ranks for ties
* `DENSE_RANK()`: No gaps in rank values

```postgresql
-- Global rank
SELECT
  *,
  RANK() OVER(ORDER BY length) AS rank_by_length
FROM film;

-- Rank within each category
SELECT
  *,
  RANK() OVER(PARTITION BY category ORDER BY length) AS rank_in_category
FROM film;
```

### Filtering by Rank

Use subqueries when filtering on window functions:

```postgresql
SELECT *
FROM (
  SELECT *,
         RANK() OVER(PARTITION BY category ORDER BY length) AS film_rank
  FROM film
) ranked
WHERE film_rank = 1;
```

### Ranking Aggregates

```postgresql
SELECT
  customer_id,
  COUNT(*) AS num_payments,
  RANK() OVER(ORDER BY COUNT(*) DESC) AS rank_by_activity
FROM payment
GROUP BY customer_id;
```

---

## 4. `FIRST_VALUE()`

Returns the value of a specified column for the **first row** in the window frame:

```postgresql
SELECT
  *,
  FIRST_VALUE(title) OVER(ORDER BY length) AS shortest_film_title
FROM film;
```

Compare to the lowest value using aggregates:

```postgresql
SELECT
  customer_id,
  COUNT(*) AS num_payments,
  FIRST_VALUE(COUNT(*)) OVER(ORDER BY COUNT(*)) AS lowest_payment_count,
  COUNT(*) - FIRST_VALUE(COUNT(*)) OVER(ORDER BY COUNT(*)) AS excess
FROM payment
GROUP BY customer_id;
```

---

## 5. `LEAD()` and `LAG()`

Used to fetch the **next** or **previous** value in a sorted window.

```postgresql
-- Compare each customer to the next
SELECT
  customer_id,
  COUNT(*) AS num_payments,
  LEAD(COUNT(*)) OVER(ORDER BY COUNT(*)) AS next_customer_count
FROM payment
GROUP BY customer_id;
```

To fetch the previous:

```postgresql
-- Compare each customer to the previous
SELECT
  customer_id,
  COUNT(*) AS num_payments,
  LAG(COUNT(*)) OVER(ORDER BY COUNT(*)) AS prev_customer_count
FROM payment
GROUP BY customer_id;
```

### Use Case: Day-over-Day Growth

```postgresql
SELECT
  DATE(payment_date) AS day,
  SUM(amount) AS daily_total,
  LAG(SUM(amount)) OVER(ORDER BY DATE(payment_date)) AS prev_day_total,
  SUM(amount) - LAG(SUM(amount)) OVER(ORDER BY DATE(payment_date)) AS difference,
  ROUND(
    100.0 * (SUM(amount) - LAG(SUM(amount)) OVER(ORDER BY DATE(payment_date)))
    / NULLIF(LAG(SUM(amount)) OVER(ORDER BY DATE(payment_date)), 0),
    2
  ) AS percent_growth
FROM payment
GROUP BY day
ORDER BY day;
```

---

## Interview Tips

* Window functions **don't reduce row count** like `GROUP BY`.
* Use `PARTITION BY` to group within a window.
* Use `ORDER BY` for sequencing and ranking.
* Use subqueries when filtering on window-based values (e.g., `WHERE rank = 1`).
* For moving averages, cumulative sums, or period-over-period growth, `LAG/LEAD` are your best friends.