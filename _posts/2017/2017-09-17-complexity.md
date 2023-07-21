---
title: "Introduction to Algorithm Complexity"
tags:
  - study
  - algorithm
  - complexity
toc: true
toc_sticky: true
toc_label: "Table of Contents"
---


## What is _Complexity_?
**알고리즘**  
특정한 크기의 입력(input)에 대한 결과(output)을 계산하는 절차

**알고리즘의 성능 지표**  
1) `시간 복잡도` (Time Complexity) : 알고리즘의 `수행 시간`(연산 횟수) 분석  
2) `공간 복잡도` (Space Complexity) : 알고리즘의 `메모리 사용량` 분석

> 동일한 기능을 수행하는 알고리즘의 경우,  
> 일반적으로 "*좋은 알고리즘 = 복잡도 낮은 알고리즘*" 이다.


**Note**  
Readability : 코드 해석이 용이한 정도
{: .notice--success}

> **Note**
> 가독성(readability) : 코드 해석의 용이한 정도


> **Why consider Complexity?**
> `프로그램의 규모(처리 데이터의 양)`는 시간이 지날수록 방대해지고 있다. 데이터의 양이 적은 경우에는 알고리즘(효율성)을 고려하지 않아도 크게 상관이 없을 수 있지만, 데이터의 양이 많아지면 알고리즘 간의 `효율성 차이`는 커질 수 밖에 없으므로, 프로그램 성능(복잡도)를 고려할 필요가 있다.


## 빅-오(Big-O) 표기법
상한 점금(asymptotically upper bound)
 - 알고리즘의 Worst-case 실행 시간을 표현할 때 사용
 - 편의상, Best, Average, Worst Case 실행 시간을 표현할 때 사용

| 시간복잡도<br>(order of growth) | 의미<br>(name) | 설명<br>(description) |
|---|---|---|
| O(1) | 상수 시간(Contant time) | 입력 크기(n)에 상관없이 일정한 연산을 수행 |
| O(log₂ n) | 로그 시간(Logarithmic time) | 입력 크기(n)이 커지면 연산 횟수가 log₂ n 에 비례해서 증가 |
| O(n) | 선형 시간(Linear time) | 입력크기(n)이 커지면 연산 횟수가 n에 비례해서 증가 |
| O(n²) | 2차 시간(Quadratic time) | 입력크기(n)이 커지면 연산 횟수가 n²에 비례해서 증가 |
| O(2ⁿ) | 지수 시간(Expotential time) | 입력 크기(n)가 커지면 연산 횟수가 2ⁿ에 비례해서 증가 |

> **Note**
> 빅-오메가(Big-Ω) : 하한 점금(asympotically lower bound)  
> **Note**
> 빅-세타(Big-θ) : Big-O와 Big-Ω의 평균(sandwitch; asympotically tight bound)


## How to reduce the Time Complexity?
1. 반복문의 숫자를 줄이자
2. 적절한 자료구조의 사용
3. 적절한 알고리즘의 이용


## Trade-off between Time Complexity and Space Complexity
> _Time Complexity_   vs   _Space Complexity_

Because there is a `trade-off relationship` between time complexity and space complexity,
it is advisable to develop algorithms that achieve an appropriate balance between the two.
However, in situations where only one aspect can be prioritized, it is preferable to approach the problem by reducing time complexity.
This is because algorithm performance is more affected by computational costs(CPU) than spacial costs(memory).

In this regard, there are some techniques that use slightly more memory to avoid repetitive computations or manage more information.
For example, _memoization_ (dynamic programming) is a way to reduce computational time at the expense of using more memory.