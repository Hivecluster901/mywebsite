---
title: "1. Big O asymptotic analysis"
date: 2025-06-02
draft: false
ShowToc: false
tags: ["basics", "Big O", "data structure", "algorithm", "interview prep"]
categories: ["Tech", "data structure", "algorithm"]
---

## 1. What is good code?

- Readability : How easily humans can understand, modify, and maintain a piece of code.
- Scalability: How well the code can handle increased load and complexity.

*Big O*, a shorthand for *Big O asymptotic analysis*, measures the scalability of code. Scalability consists of runtime and memory. There is usally a trade-off between runtime and memory. In this post, we introduce the Big O notation in terms of runtime and look at how to measure the space complexity of an algorithm at the end.

## 2. Big O

*Big O* is the language to talk about how long an algorithm takes to run, or a runtime of an algorithm, **regardless of external factors such as differences in hardwares**. This is a better measure than actual execution time as execution time dependes on the computing power of the machines.

***Big O* measures algorithmic efficiency, i.e. how many operations needs to be done as we increase the size of our input.**

For example,

``` python
n = 100 # Example
for i in range(n):
    print(i)
```

The above code takes linear time as the number of operations increases linearly with `n`. We denote `O(n)`.
On the other hand, if the number of operations is independent of the size of the input, we denote the time complexity as `O(1)`. For example, if you are accessing the first element of an array in Python:

``` sql
n = 100 # Example
array1 = [i for i in range(n)]
print(array1[0])
```

The number of operations is `1` no matter how many elements there are in the array.

We can be extra precise when calculating the Big O notations. For example, look at the code below:
``` python
def function1(input):
    a = 10 # 1 (assignment)
    a = 50 + 3 # 2 (addition + assignment)
    for _ in input:
        stranger = True # n (assignment for n times)
        a += 1  # 2n (read + add + assign 
                # -> could be 3 but 2 meaningful operations)
    return a # 1
```

The total complexity is `3n+4`, but asymptotically, it is `O(n)`. However, we don't always have to go through line-by-line to calculate the Big O. There are a few rules to infer the time complexity directly:

1. Always look at the worst case: There can be particular inputs that can make the algorithm perform faster. However, since we are dealing with general inputs, always look for the scaling of the worst case scenarios.
2. Drop all the constants. Asymptotically, as `n` increases, only the leading order term matters and, furthermore, we are only interested in how the runtime will change with increasing input, e.g. `O(3n + 4) = O(n)`.

3. Different terms for inputs: This means, if there are two different terms to represent the input size, the Big O should be accounted for each term. For example

``` python
n = 100 # Example
m = 10 # Example
for i in range(n):
    print(i)
for j in range(m):
    print(j)
```
The time complexity is not `O(n)` but `O(n + m)`. In the case a nested loop,

``` python
n = 100 # Example
m = 10 # Example
for i in range(n):
    for j in range(m):
        print((i,j))
```
We multiply the time complexity instead of summing them, and, thus, we have `O(n*m)`.

4. Drop non dominants. If the Big O is `O(n^2 + 3 * n)`, then we only keep the leading order term, i.e. `O(n^2)`.

### Space Complexity

So far, we focused on the runtime of algorithms. However, there is an equally important part of scalability of code, i.e. space complexity. *Space complexity* refers to how much memory an algorithm uses as the input size grows. It measures how much space an algorithm needs to store data, variables, and the code itself during it execution. The core contributors of space complexity are:

- Primitive Variables
- Function Call Stack (Recursion Depth)
- Explicit Data Structures (arrays, lists, trees, etc.)
- Implicit Allocations (temp arrays, return values, etc.)
- Caching / Memoization tables (especially in dynamic programming)
- Input Copying (when inputs are cloned or transformed)

These live in either stack or heap. Primitive varaibles like `int`, `float`, and `bool` are usually stored on stack. However, explicit data structures, e.g. arrays and lists, are stored in heap as they are dynamic.


**Only consider additional space. Don't consider the size of the input.** Let's look at a few examples.

``` python
def example(n):
    for i in range(10):
        print('Hello')
```
The space complexity is `O(1)` as we only need additional space for `i` and `range(10)`.

``` python
def example(n):
    hello_array = []
    for i in range(n):
        hello_array.append('Hello')
    return hello_array
```
The space complexity is `O(n)` as we created a new data structure, `hello_array` on top of `i` and `range(10)`.

