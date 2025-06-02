---
title: "SQL 9. "
date: 2025-06-03
draft: true
ShowToc: true
tags: ["sql", "joins", "databases", "interview prep"]
categories: ["Tech", "Databases"]
---

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