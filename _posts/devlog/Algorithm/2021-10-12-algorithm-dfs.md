---
layout: post
title: "[Algorithm] Recursive Function(재귀 함수)"
subtitle: "recursive function"
categories: devlog
tags: algorithm RecursiveFunction
---

> 삽입 정렬 알고리즘입니다.

<!--more-->

- 목차
  - [Recursive Function이란?](#-recursive-function이란)
  - [사용 예제](#-사용-예제)
    - [나의 풀이](#-나의-풀이)
    - [다른 풀이](#-다른-풀이)

## 📌 Recursive Function이란?

---

- 간단하게 한 함수 내에서 자기 자신을 다시 호출하는 방식을 가진 함수이다.
- 보통 재귀함수는 스택에 매개변수, 지역변수, 복기주소로 구성되어 있는 스택 프레임에 정보가 저장되기 때문에 일반 반복문보다 느릴 수 있다는 단점을 가지고 있다.
- 개인적으로 느끼기에 재귀함수의 전체적인 흐름은 실행컨텍스트의 흐름으로 이해해도 될 것 같다고 생각했다. (물론 실행컨텍스트가 재귀함수를 설명하기 위한 개념은 아닐 것이지만..)

## 📌 사용 예제

---

- 1, 2, 3 순으로 나열하시오.

### 💁🏻‍♂️ 나의 풀이

```
let initNum = 0;

function selfFunc(a) {
  if(initNum < a) {
    initNum += 1;
    console.log(initNum);
    selfFunc(a);
  } else {
    return;
  }
}
```

### 💁🏻‍♀️ 다른 풀이

```
function selfFunc(a) {
  if(a === 0) return;
  else {
    selfFunc(a - 1);
    console.log(a);
  }
}
```

- 훨씬 간단해보이지만 중요한 것은 그것이 아니다.(아니..중요할수도..ㅎ)
- 각각 console.log를 찍는 위치가 재귀함수를 다시 호출하기 전/후로 나뉘지만 결과값은 1, 2, 3으로 같다.
- 여기서 스택의 개념이 사용된다.
- 재귀함수는 또 다시 호출된 자기 자신의 함수가 끝나기 전까지는 스택에서 사라지지 않고 보류된다.
- 마치 실행컨텍스트에서 사용되는 콜스택 개념처럼!!

> 🎈 재귀함수는 스택의 개념이 중요하다!
