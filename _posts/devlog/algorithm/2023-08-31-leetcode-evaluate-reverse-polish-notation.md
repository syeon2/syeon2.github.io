---
layout: post
title: "[LeetCode] 150. Evaluate Reverse Polish Notation"
subtitle: "algorithm"
categories: devlog
tags: algorithm stack
---

> LeetCode Top Interview 150의 150번 문제입니다.

<!--more-->

📚 목차
- [🌱 Evaluate Reverse Polish Notation](#-evaluate-reverse-polish-notation)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 연산해야할 숫자(피연산자)와 연산자("+", "-", "*", "/")가 1차원 배열에 담겨 주어진다.
- 역폴란드 표기법으로 산술 표현식을 나타냈다.

- 1차원 배열에 주어진 문자 순서대로 연산하여 최종 값을 반환하는 문제이다.


- 조건
  - 1 <= tokens.length <= 10<sup>4</sup>
  - tokens[i] is either an operator: "+", "-", "*", or "/", or an integer in the range [-200, 200].
---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

일반적으로 숫자 연산은 2 + 1 같이 연산자가 피연산자의 가운데에 들어가지만 위에서 제공하는 표기법은 2 1 + 같이 연산할 피연산자를 미리 주고 
이후 앞서 나온 피연산자를 연산하기 위한 연산자를 뒤에 표기한다.

이 문제를 풀기 위해서는 Stack 자료구조를 사용하면 쉽게 해결할 수 있다. 앞서 나온 숫자를 stack에 add하고 연산자가 등장하면 stack에서 
2개의 숫자를 pop하여 연산한다. 이 때 주의해야할 점이 2가지 있다.

1. stack에서 pop한 2개의 피연산자를 연산한 값은 다시 stack에 넣어야한다. 이후에 이어질 수 있는 연산을 위해서 반드시 연산결과를 stack에 
다시 넣어야한다.

2. -와 /를 할 때에는 2번째로 pop한 값을 1번째로 pop한 값으로 나누어야한다. 생각해보면 a b - 가 있다면 a - b가 되어야 한다. 해당 값을 stack에 
넣으면 a가 b보다 나중에 pop되기 때문에 순서를 잘 지켜야 연산의 오류가 없다.

> 🥕 시간복잡도는 tokens 배열을 한번 순환하며 처리하기 때문에 O(N)의 시간복잡도가 걸린다.
> 
> 🥕 공간복잡도는 stack에 들어갈 값이 최대 tokens에서 숫자를 가지고 있는만큼의 값이지만 상수이기 때문에 O(N)의 공간복잡도를 가진다. 

---

### 🟤 문제 풀이 (Algorithm)

```java
import java.util.*;

class Solution {
    public int evalRPN(String[] tokens) {
        Stack<Integer> stack = new Stack<>();

        for (int i = 0; i < tokens.length; i++) {

            String str = tokens[i];

            if (str.charAt(str.length() - 1) >= '0' && str.charAt(str.length() - 1) <= '9') {
                stack.add(Integer.parseInt(str));
            } else {
                Integer a = stack.pop();
                Integer b = stack.pop();

                if (str.equals("+")) {
                    stack.add(a + b);
                } else if (str.equals("-")) {
                    stack.add(b - a);
                } else if (str.equals("*")) {
                    stack.add(a * b);
                } else {
                    stack.add(b / a);
                }
            }
        }

        return stack.pop();
    }
}
```

---
