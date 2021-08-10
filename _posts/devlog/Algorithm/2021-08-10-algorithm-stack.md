---
layout: post
title: "[Algorithm] Stack"
subtitle: ""
categories: devlog
tags: algorithm Stack
---

> Stack 자료구조를 활용한 알고리즘입니다.

<!--more-->

- 목차
  - [Stack이란?](#-stack이란)
  - [사용 예제(배열)](#-사용-예제배열)

## 📌 Stack이란?

---

- '쌓다', '더미' 라는 의미를 가진 알고리즘
- 이름의 의미와 같이 한쪽으로만 열려있기 때문에 한쪽으로만 데이터를 받고 빼는 구조로 되어 있다. (후입선출)
  <img alt="" src="/assets/img/algorithm/stack1.png"  width="400px"> Last In First Out

- Stack의 구현 방법은 배열, 연결 리스트 2가지가 있다. (Stack이라고 해서 특별한 저장 공간을 가지는 array나 object가 있는 것이 아니라 stack이라는 자료구조 형식으로 알맞게 구현하는 것!)
- 각각 배열과 연결 리스트의 장단점이 있는데 <br>
  배열은 '데이터 접근 속도가 빠르기 때문에 데이터 양이 많으면 많을수록 효율이 좋지만, 데이터 삽입과 삭제에서는 비효율적'이다. <br>
  연결리스트는 '데이터 접근할 경우에는 연결되어 있는 노드를 따라 확인하며 접근하기 때문에 접근, 속도에서는 효율이 떨어지지만 데이터 삽입, 삭제는 배열보다 효율이 좋다'.

## 📌 사용 예제(배열)

---

- ()의 짝이 맞는 경우는 true, 안맞으면 false

```
let str = '(()()(())' // false

const question = (str) => {
    let stack = [];

    for(let i of str) {
        if(i === '(') {
            stack.push(i);
        }
        else {
            if(stack.length === 0) {
                return false;
            } else {
                stack.pop();
            }
        }
    }
    if(stack.length > 0) return false
    else return true
}
```

- stack이라는 배열을 만들고 난 후 '('가 있으면 stack에 push하고 ')'가 있으면 기존에 있는 '('를 pop하는 방식이다.
- 이렇듯 하나의 배열에 데이터를 순서대로 쌓고 순서대로 빼는 방식이 stack 자료구조이다.
