---
title: "SQL 7. Conditional Expressions"
date: 2025-06-01
draft: true
ShowToc: true
tags: ["sql", "conditions", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

Here, we go through conditional expressions.

## 1. CASE WHEN
*CASE WHEN* goes through a set of conditions and returns a value if a condition is met. The syntax is as follows:
``` sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    WHEN conditionN THEN resultN
    ELSE result
END
```
Start the statement with `CASE` and **end with `END`**. In between, the statement has conditions and results. If multiple conditions are met, the result of the highest priority is returned. For example, if both `condition1` and `condition2` are true, only `result1` will be returned. If there is no `ELSE` statement, and all the conditions after `WHEN` are not met, `NULL` is returned.

Now, we look at a very speical use case of `CASE WHEN` in conjunction with `SUM`.

### a) CASE WHEN & SUM

We can count a column with certain value using `CASE WHEN` and `SUM`. The syntax is as follows:

``` sql
SELECT
SUM(CASE WHEN column_name1 = val1 THEN 1 ELSE 0)
FROM table_name1
```

## 2. COALESCE
*COALESCE* returns first value of a list of values which is not null. The syntax is as follows:

``` sql
COALESCE (column1, column2)
```
Instead of specifying other columns, one can also specify the default value.
``` sql
COALESCE (column1, '1970-01-01 0:00')
```
`column1` and the default value should be the same type. Due to this problem, it is useful to be able to do type-casting.

### a) CAST
`CAST` changes the data type of a value. The syntax is as follows:

``` sql
CAST (value/column AS data_type)
```

Often, data are not clean to do type-casting. For example, if there is a white space between two numbers, which is saved as `VARCHAR` type, it wouldn't be possible to type-cast it to an integer. Thus, we need to be able to replace the white space with something else before type-casting.

### b) REPLACE

*REPLACE* is used to replace text from a string in a column with another text. The syntax is
``` sql
REPLACE (column, old_text, new_text)
```
