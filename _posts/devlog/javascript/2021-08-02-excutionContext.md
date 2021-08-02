---
layout: post
title: "[JavaScript] 실행 컨텍스트"
subtitle: "실행 컨텍스트(Excution context)의 개념"
categories: devlog
tags: javascript CoreJavaScript ExcutionContext LexicalEnvironment VariableEnvironment
---

> 실행 컨텍스트(Excution context)의 개념입니다.

<!--more-->

- 목차
  - [실행 컨텍스트란?](#-실행-컨텍스트란)
  - [실행 컨텍스트 동작 원리](#-실행-컨텍스트-동작-원리)
  - [실행 컨텍스트의 구조](#-실행-컨텍스트의-구조)
    - [Hoisting](#-호이스팅hoisting)
  - [예제](#-예제)
    - [Scope](#-scope)
  - [정리](#-정리)

## 📌 실행 컨텍스트란?

---

- 실행 컨텍스트는 실행할 코드에 제공할 변수, 함수, 매개 변수등 환경 정보들을 수집하여 모아놓은 객체
- Variable Environment, Lexical Environment, thisBinding으로 구성되어 있다.

## 📌 실행 컨텍스트 동작 원리

---

### 🌱 사전에 알아야할 CS지식

#### ○ Stack

- Stack은 출구가 하나뿐인 데이터 구조이다.
- 여러 가지 데이터를 순서대로 쌓아 저장했다면, 데이터를 꺼낼 때는 역순으로 꺼내야한다.

<img src="/assets/img/javascript/excutionContext/stack.png" width="500px">

#### ○ Queue

- Queue는 양쪽이 모두 열려있는 데이터 구조이다.
- 여러 가지 데이터를 순서대로 저장했다면, 순서대로 데이터를 꺼낼 수 있는 구조이다.

<img src="/assets/img/javascript/excutionContext/queue.png" width="500px">

### 🌱 그래서 실행 컨텍스트는?

- 실행할 코드에 필요한 환경 정보들을 모아 놓은 실행 컨텍스트는 일반적으로 함수가 호출될 때 구성된다.<br>
  (전역은 코드가 실행되는 즉시 전역 환경 정보들을 모아 놓는다.)
- 전역을 첫번째로, 함수가 호출되면 각각 콜스택에 쌓인다.<br />
  (이 때, 콜스택에 쌓이면 해당 변수나 함수를 할당하다가 또다른 실행컨텍스트가 생기면 할당하던 컨텍스트는 잠시 중지된다.)

## 📌 실행 컨텍스트의 구조

---

- 실행 컨텍스트는 크게 3가지로 구성되어 있다. (ThisBinding은 다음에 공부..!)

1. Variable Environment <br />
   실행 컨텍스트가 환경 정보를 수집하는 최초의 값을 유지하며 담는다.

2. Lexical Environment <br />
   Variable Envionment에 정보를 담은 후, Variable Environment를 토대로 Lexical Environment를 만든다. Lexical Environment는 Variable Environment와 다르게 환경 정보의 변경 값을 적용한다.

> - Variable Environment와 Lexical Environment는 또 environmentRecord, outerEnvironmentReference로 구성되어 있다.
> - environmentRecord가 실행 컨텍스트의 환경 정보 수집하는 역할을 이행하는 역할을 맡고 있다
> - outerEnvironmentReference는 만약 해당 컨텍스트에서 참조해야하는 정보가 없을 경우 제일 근접한 상위 컨텍스트의 Lexical environment를 참조하는 역할을 한다.

#### 🎈 호이스팅(Hoisting)?

> 호이스팅은 선언한 변수들이 위치와 상관없이 제일 상단에 선언되어 변수나 함수를 호출할 수 있는 현상을 말한다.
> 실제로 코드가 제일 상단에 올라가는 것이 아니라, 코드가 실행되면 실행 컨텍스트가 만들어지고 환경 정보를 수집하는데 자바스크립트 엔진은 환경 정보로 수집한 변수명들을 먼저 알게 되는 것이다. 이 과정을 호이스팅이라 한다.

## 📌 예제

---

```
var a = 1
function b() {
    function c() {
        console.log(a);
        var a = 1;
    }
    c();
    console.log(a);
}
b()
console.log(a);
```

- 전역 실행 컨텍스트가 형성되고 변수 a와 함수 b를 환경 정보에 저장했다. / 콜스택에 전역 실행 컨텍스트가 쌓였다.
- 변수 a에 1을 할당하고, b에 함수를 할당하고 함수를 호출한다. / 콜스택에 함수 b의 실행 컨텍스트가 쌓였다.
- 함수 b의 환경 정보로 변수 c에 함수가 할당되어 저장된다.
- 함수 b안에 함수 c가 호출되었다. / 콜스택에 함수 c의 실행 컨텍스트가 쌓였다.
- 함수 c안에 환경 정보로 a가 선언되어 호이스팅 되었다. / c안의 console은 호이스팅된 a를 가리키기 때문에 호이스팅으로 선언만 된 a를 undefined를 가진다.
- 함수 c의 실행 컨텍스트가 종료됨으로 b의 실행 컨텍스트로 넘어간다. / 함수 b의 console.log(a)는 해당 컨텍스트 안에 a가 없기 때문에, outerEnvironmentReference로 인해 a를 가지고 있는 제일 근접한 상위 컨텍스트의 Lexical Environment를 참조하려 한다. (정확히는 해당 컨텍스트가 실행되기 전 Lexical Environment를 참조한다.)
- 마침 전역에 a가 있으므로 함수 b의 console.log(a)는 1값을 가진다.
- b의 실행 컨텍스트가 종료되고 전역 console.log(a)는 마지막 전역에 있는 a를 참조하여 1값을 가진다. / 전역 실행 컨텍스트가 완료되어 종료됨으로 콜스택에는 아무것도 남아있지 않다.

#### 🎈 Scope

> scope는 쉽게 말해 변수의 유효 범위이다. 실행 컨텍스트와 가리키는 범위는 같지만, 스코프는 실행 컨텍스트가 접근할 수 있는 모든 변수와 함수에 순서를 정하는 것이 목적이다.
> 위의 예시에 있는 outerEnvironmentReference가 상위 Lexical Environment를 참조하게되는 연결점, 외부 실행 컨텍스트에서 내부 실행 컨텍스트에 저장된 변수들은 접근할 수 없는 것, 이러한 것들을 스코프 체인이라 한다.

## 📌 정리

---

- 실행 컨텍스트는 실행할 코드에 제공할 변수나 함수같은 환경 정보들을 모아놓은 객체이다.
- 실행 컨텍스트의 정보들은 순서에 따라 콜스택 형태로 저장되고 호출된다.
- 호이스팅은 컨텍스트와 연관이 있으며, 코드가 실행되기 전 환경 정보를 저장하여 미리 변수명 등을 알게되는 자바스크립트 엔진에 의해 일어난다.
- 스코프는 실행 컨텍스트가 접근할 수 있는 변수나 함수들에게 순서를 부여하는 역할이다.
