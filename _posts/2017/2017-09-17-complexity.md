---
title: "Introduction to Algorithm Complexity"
tags:
  - study
  - algorithm
  - complexity
toc: true
toc_sticky: true
toc_label: "Table of Contents"
classes: wide
---



## What is _Complexity_?

> "In computer science, (computational) complexity of an algorithm is the `amount of resources` required to run it. Particular focus is given to `computation time` (generally measured by the number of needed elementary operations) and `memory storage requirements`. The complexity of a problem is the complexity of the `best algorithms` that allow solving the problem."
> <br><br>
> [https://en.wikipedia.org/wiki/Computational_complexity](https://en.wikipedia.org/wiki/Computational_complexity)

**Algorithm**<br>
A well-defined `computational procedure` that consists of input, computational procedure, and output.<br>
   1) *input* : a value or a set of values.<br>
   2) *computational procedure* : a sequence of computational steps that transform the input to output.<br>
   3) *output* : a value or a set of values.<br>
{: .notice--success}



## Why consider Complexity?

Different algorithms devised to solve the same problem differ in their efficiency. Thus, `system performance` depends on choosing efficient algorithms. Also, today, the amount of data processed by program(s) becomes more massive. If the amount of data is small, the efficiency of algorithm may not be of significant concern. However, as the `data volume` increases, the importance of efficiency becomes greater. Therefore, it is essential to consider the complexity for software development.

**Readability**<br>
The ease with which a developer can understand a written code.
{: .notice--success}



## Analyzing the Algorithm

It predicts the `resources` (ex. computational time, memory, etc) that algorithms require. For your reference, each instruction takes a constant amount of time.

- `Time Complexity` : the amount of `computational time` taken by an algorithm. (# of primitive instructions executed)
- `Space Complexity` : the amount of `memory space` taken by an algorithm. (memory usage)



## Asymptotic Notation

It is convenient for describing the worst-case running time function which usually is defined only on integer input sizes, because it abstracts away some details of the running time function.

- `Big-O Notation` (O) : an asymptotically `upper bound` for input(n). (for expression of `worst-case` running time)
- `Big-theta Nontation` (θ) : an asymptotically `tight bound` for input(n). (for expression of ??? running time)
- `Big-omega Notation` (Ω) : an asymptotically `lower bound` for input(n). (for expression of `best-case` running time)



## Order(Rate) of Growth

It is used to simplify the analysis of algorithm by ignoring the constant coefficients in general formula. The `simplified order` is used to compare the algorithm's running time.

| Order of growth | Name | Description | example |
|---|---|---|---|
| O(1)      | Contant         | Regardless of input size, it always takes a fixed number of steps. | add two numbers |
| O(log₂ n) | Logarithmic     | As input size(n) increases, #(operations) increases proportionally to log₂ n. | binary search |
| O(n)      | Linear time     | As input size(n) increases, #(operations) increases proportionally to n. | loop |
| O(n²)     | Quadratic time  | As input size(n) increases, #(operations) increases proportionally to n². | double loop |
| O(2ⁿ)     | Expotential time| As input size(n) increases, #(operations) increases proportionally to 2ⁿ. | exhaustive search |



## How to reduce the Time Complexity?

1. Utilize the useful `data structure`.
2. Utilize the efficient `algorithm technique`.
3. Reduce the `iterations` of loop.



## Trade-off between Time Complexity and Space Complexity

> _Time Complexity_   vs   _Space Complexity_

Because there is a `trade-off relationship` between time complexity and space complexity, it is advisable to develop algorithms that achieve an appropriate balance between the two. However, in situations where only one aspect can be prioritized, it is preferable to approach the problem by reducing time complexity. This is because algorithm performance is more affected by computational costs(CPU) than spacial costs(memory).

In this regard, there are some techniques that use slightly more memory to avoid repetitive computations or manage more information. For example, _`memoization`_ (dynamic programming) is a way to reduce computational time at the expense of using more memory.
