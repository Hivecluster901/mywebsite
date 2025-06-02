---
title: "SQL 5. Date and Time Functions"
date: 2025-06-01
tags: ["sql", "function", "date", "time", "databases", "interview prep"]
categories: ["Tech", "Databases"]
draft: false
ShowToc: true

---

In this post, we are going to look at basic date and time functions in SQL. Before we dive deeper into the date and time functions, let's look at standard date/time types.

### 0. Data/Time types
{{< table >}}

| Types                               | Description            | Examples                         |
|:------------------------------------:|:----------------------------:|:---------------------------------:|
| date                                | Just date <br> without time | '2025-06-01'                |
| time <br> (with/without time zone)  | Just time <br> without date | '01:02:03.678+02'           |
| timestamp <br> (with/without time zone) | Date and time      | '2022-11-28 01:02:03.678+02'     |
| intervals                           | Time interval           | '3 days 01:02:03.678'           |
{{< /table >}}

## 1. EXTRACT
*EXTRACT* is used to extract parts of timestamp/date. The syntax is as follows:

``` sql
EXTRACT (field from date/time/interval)
```
`field` is part of date/time, and `date/time/interval` represents column we want to extract from.

### 2. TO_CHAR

*TO_CHAR* is used to get custom formats for timestamp/date/numbers. The output type is always text. The syntax is as follows:
``` sql
TO_CHAR (date/time/interval/number, custom format)
```
For example,
``` sql
TO_CHAR (rental_date, 'MM-YYYY')
```
For the specific formatted strings, refer to the [official webpage for formatting functions](https://www.postgresql.org/docs/current/functions-formatting.html).

### 3. Helful Interval and Timestamp
`CURRENT_DATE` and ``CURRENT_TIMESTAMP` are used to get current dates and timestamps. These are useful if we want to obtain the interval between the current timestamp and the timestamps in the database. For example,
``` sql
SELECT
CURRENT_TIMESTAMP - timestamp1
fROM rental
```