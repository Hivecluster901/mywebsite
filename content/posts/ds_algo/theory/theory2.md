---
title: "2. Array/List"
date: 2025-06-05
draft: false
ShowToc: true
tags: ["basics", "array", "list", "data structure", "algorithm", "interview prep"]
categories: ["Tech", "data structure", "algorithm"]
---

Arrays, more commonly known as **lists** in Python, are one of the most fundamental and widely used data structures. They store items in a sequential, ordered manner, and support a wide range of operations efficiently.

This post is designed as a refresher for interviews or technical review, with code examples, time complexity analysis, and clarifications on memory behavior.

---

## 1. What is an Array (List)?

In Python, the built-in `list` type is a **dynamic array** implementation. Arrays:

- Store elements **in order**, using **contiguous memory**.
- Allow fast access to elements via **indices**.
- Are **resizable**, meaning you can add/remove elements dynamically.
- Have a relatively small **memory footprint** compared to more complex data structures like linked lists or trees.

---

### a) Time Complexity of Array Operations

{{<table>}}
| Operation      | Time Complexity | Description                                               |
|----------------|-----------------|-----------------------------------------------------------|
| Lookup         | O(1)            | Accessing an element by index                             |
| Append (push)  | O(1) *amortized*| Add item at the end (usually constant time)               |
| Pop (end)      | O(1)            | Remove last item                                          |
| Insert (middle)| O(n)            | Insert at arbitrary position; may shift many elements     |
| Delete (middle)| O(n)            | Removing by index or value; may require shifting elements |
| Membership     | O(n)            | Checking whether a value is in a list                     |
{{</table>}}

---

### b) Example Code

```python
strings = ['a', 'b', 'c', 'd']

# Lookup (O(1))
strings[2]  # returns 'c'

# Append (O(1) amortized)
strings.append('e')  # adds 'e' to the end

# Pop (O(1))
strings.pop()  # removes 'e' from the end

# Insert (O(n))
strings.insert(0, 'x')  # inserts 'x' at index 0, shifts all others

# Delete (O(n))
strings.pop(2)  # removes element at index 2, shifts following elements

# Membership Test (O(n))
'c' in strings # Python performs a linear search, checking each element until it finds a match (or not).
```

> Amortized O(1)
> Amortized time measures the average time per operation over a sequence of operations, even if some individual operations are more expensive. Amortized time of append in list is O(1).
> - Most append() calls are O(1)
> - But occasionally, Python resizes the internal array (copies it to a bigger one) — that takes O(n) time.
> - However, resizing happens rarely (e.g., when the list grows past its current capacity).
> - So over many appends, the average cost per operation stays constant.

> Insertion of deletion anywhere except at the end requires shifting subsequent elements, which takes linear time.

## 2. Static vs Dynamic Arrays
### Static Arrays
- Fixed size: Must declare size upfront (e.g., in C or Java).
- Contiguous memory: Entire block is allocated at once.
- Fast access, but no resizing.

### Dynamic Arrays (like Python lists)
**Growable**: Automatically resize when more space is needed.

When capacity is exceeded:

- A larger block of memory is allocated (usually 2x).
- The entire array is copied to the new location.
- The new item is appended.

Example (Conceptual):
```text
Initial Capacity: 4
Current Items:    ['a', 'b', 'c', 'd']
Append 'e'
Next Append:      Resize to capacity 8, then add item
```

---
The overallocation of memory can be shown here:
``` python
import sys

arr = []
for i in range(20):
    arr.append(i)
    print(f"Length: {len(arr)}, Size in bytes: {sys.getsizeof(arr)}")
"""
output:
Length: 1, Size in bytes: 88
Length: 2, Size in bytes: 88
Length: 3, Size in bytes: 88
Length: 4, Size in bytes: 88
Length: 5, Size in bytes: 120
Length: 6, Size in bytes: 120
Length: 7, Size in bytes: 120
Length: 8, Size in bytes: 120
Length: 9, Size in bytes: 184
Length: 10, Size in bytes: 184
Length: 11, Size in bytes: 184
Length: 12, Size in bytes: 184
Length: 13, Size in bytes: 184
Length: 14, Size in bytes: 184
Length: 15, Size in bytes: 184
Length: 16, Size in bytes: 184
Length: 17, Size in bytes: 248
Length: 18, Size in bytes: 248
Length: 19, Size in bytes: 248
Length: 20, Size in bytes: 248
"""
```

The size consists of base size and the number of allocated slots for pointers, which are 8 bytes on 64-bit.
On 64-bit macOS, the base size of the list is `56` bytes. When there are less than 5 elements in the list, 88 bytes are allocated, i.e. 56 bytes + 4 * 8 bytes.

---
## Additional Interview notes
- Python lists can store heterogeneous data (e.g. mix of ints, strings, and objects)
- Under the hood, each element is a pointer to a Python object.
- Use `collections.deque` for fast appends/pops from both ends (`O(1)` for both ends)
- Use `array.array` or `numpy.array` if you need memory-efficient,typed arrays. `array` and `numpy` module provides a memory-efficient array of **primitive C types**. Unlike lists, which store references to Python objects, `array.array` and `numpy.array` stores **raw values** like C arrays.

