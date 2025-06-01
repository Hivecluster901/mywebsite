---
title: "SQL 4. String Functions"
date: 2025-06-01
draft: false
ShowToc: true
---

In this post, we are going to look at basic string functions in SQL.

## 1. UPPER, LOWER, LENGTH

*UPPER* changes all lowercase alphabets to uppercase. *LOWER* changes all uppercase letters to lowercase. Finally, *LENGTH* gives the number of characters. The example is as follows: 
``` sql
SELECT 
UPPER(first_name)
LOWER(last_name)
FROM customer
WHERE LENGTH(first_name) > 10
```
The above code outputs the first names in uppercase and last names in lowercase from the table `customer` where the length of the first name is more than `10`.

## 2. LEFT/RIGHT

Sometimes, we want to have only a part of strings. *LEFT* and *RIGHT* take two arguments, i.e. the string and a integer, where it takes an integer number of characters from the left or right. For example,
``` sql
SELECT
LEFT(first_name, 2),
first_name
FROM customer
```
The above code selects two characters from the left of the first names of the customers. If you want to extract the middle part of the string, you can nest `LEFT` and `RIGHT`. For example,
``` sql
SELECT
RIGHT(LEFT(first_name, 2),1),
first_name
FROM customer
```
Now, the above code extracts only the second letter from the first names of the customers.

## 3. Concatenate

You can concatenate two strings by using `||` operator. For example, if you want an inital of customers, you can run the following code:

``` sql
SELECT
LEFT(first_name, 1) || LEFT(last_name, 1)
FROM customer
```

## 4. POSITION
*POSITION* returns the position of certain characters from a string. For example, if we want to get the position of `@` in emails of customers, we can do the following:

``` sql
SELECT
POSITION('@' IN email)
FROM customer
```

Combined with `LEFT` and `RIGHT`, one can obtain the left or right of the given character.

## 4. SUBSTRING
*SUBSTRING* is used to extract a substring from a string. It is often more convenient to use than `LEFT` and `RIGHT` when the substring we want to extract is right in the middle. The syntax is as follows.

``` sql
SUBSTRING (string from start [for length])
```
`string` can be column name, `start` is the starting position of the substring and `for length` sets how many characters to extract. `for length` is inside parentheses to show that it is optional. If this part is not specified, substring is extracted to the end of the string.

Often, the starting position, `start`, and the length, `length`, can be made dynamic by using `POSITION`, `LENGTH`.
For example, if you want to get the last name from an email address, you can do as follows:
``` sql
SUBSTRING(email from POSITION('.' in email) + 1 for POSITION('@' in email) - POSITION('.' in email) -1)
```