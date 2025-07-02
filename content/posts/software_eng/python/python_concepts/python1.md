---
title: "1. Python: map"
date: 2025-06-05
draft: true
ShowToc: true
tags: ["python", "map", "programming language" , "software engineering", "interview prep"]
categories: ["Tech", "python"]
---
The `map()` function applies a given function to all items in an input list (or any other iterable) and returns a map object (an iterator). This is particularly useful for transforming data in a list comprehensively.
> An iterable is any python object:
> - capable of returning its members one at a time, permitting it to be iterated over in a for-loop.
> - implements `__iter__` method, which returns an iterator, or `__getitem__` method suitable for indexed lookup.
> - The examples are `list`, `tuple`, `string`, `dict`, `set`.

> An iterator is an object that:
> - keeps *state* during iteration.
> - produces the next value with the `__next__` method, which updates the state to point at the next value and signals stop by raising `StopIteration`.
> - self-iterable, i.e. `__iter__` return `self`.
> Iterators are consumed so that, after they are fully gone through, they can't be reused unless recreated.
> The examples are `iter(iterable)`, `enumerate(iterable)`, `zip(iterable1, iterable2)`, `generator`, and file objects, i.e. `open('example.txt')`


The syntax is as follows:
``` python
map(func1, iterable) # single iterable
map(func2, iterable1, iterable2) # multiple iterables
```
Functions can be either regular functions or lambda functions.

Examples:
``` python
nums1 = [1,2,3]
nums2  = [4,5,6]
print(list(map(lambda x,y: x+y, nums1, nums2)))
# output: [5, 7, 9]
```

You can do batch type casting with map:

``` python
str_nums = ['1', '2', '3']
int_nums = list(map(int, str_nums))
print(int_nums) # output: [1, 2, 3]
```

You can apply in-built function with map:

```python
words = ['apple', 'banana', 'cherry']
upper_word = list(map(str.upper, words))
print(upper_word) # output: ['APPLE', 'BANANA', 'CHERRY']
```

You can apply a map function to a list of dictionary.

``` python
dict_list = [{'age': 23}, {'age': 15}, {'age': 57}]
print(list(map(lambda d: d['age'], dict_list))) # output: 23, 15, 57
```

