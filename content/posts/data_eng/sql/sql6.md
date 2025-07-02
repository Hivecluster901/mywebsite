---
title: "SQL 6. Numerical Functions and Operators"
date: 2025-06-01
draft: true
ShowToc: true
tags: ["sql", "function", "operators", "numerics", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---
## 1. Operators

{{< table >}}

| Operator                              | Description            | Examples                         | Result
|:------------------------------------:|:----------------------------:|:---------------------------------:|:--------:|
| +                           | addition | 4+3                | 7 |
| -  | subtraction | 5-3           |2 |
| * | multiplication      | 4*2     | 8 |
| /                         | division (integer division truncates the result)          | 8/4          | 2 |
| % | modulo (remainder) | 10%4 | 2|
| ^ | exponentiation | 2^3 | 8 |
{{< /table >}}

For example, `9/4` will give `2` while `9.0/4` will give `2.2500...00`.

## 2. Functions


{{< table >}}

| Function                           | Description            | Examples                         | Result
|:------------------------------------:|:----------------------------:|:---------------------------------:|:--------:|
| abs(x)                           | absolute value | abs(-2)                | 2 |
| round(x, d)  | round x to d decimal places | round(7.4632, 2)           | 7.46 |
| ceiling(x) | round up to integer     | ceiling(7.4632)    | 8 |
| floor(x)                         | round down to integer          | floor(7.4632)          | 7 |
{{< /table >}}

