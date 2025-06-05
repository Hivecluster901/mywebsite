---
title: "2. Python: filter"
date: 2025-06-05
draft: false
ShowToc: true
tags: ["python", "filter", "programming language" , "software engineering", "interview prep"]
categories: ["Tech", "python"]
---

The `filter()` function constructs an iterator from elements of an iterable for which a function returns `true`. It is used to filter out items from a list (or any other iterable) based on a condition.

The syntax is similar to [map](/posts/software_eng/python/python_concepts/python1):

``` python
filter(func1, iterable)
```

Examples:

``` python
num_list = [i for i in range(1, 13)] # [1, ..., 12] 
print(list(filter(lambda x: x%2==0, num_list)))
# output: [2, 4, 6, 8, 10, 12 ]
```

You can filter with multiple conditions:

``` python 
num_list = [i for i in range(1,10)] #[1,2,...,9]
even_and_greater_than_five = list(filter(lambda x: x> 5 and x%2==0, num_list))
print(even_and_greater_than_five) # output: [6, 8]
```

You can perform filter operation on dictionaries:
``` python
dict_list = [{'age': 23}, {'age': 15}, {'age': 57}]
print(list(filter(lambda d: d['age'] > 25,  dict_list))) # output: [{'age': 57}]
```