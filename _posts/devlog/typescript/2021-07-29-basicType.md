---
layout: post
title: "[TypeScript] 기초 개념"
subtitle: "typescript의 기초 개념과 변수 지정 방법"
categories: devlog
tags: typescript TypeScript basic
---

> TypeScript의 기본 타입 종류에 대한 포스트입니다.

<!--more-->

- 목차

  - [TypeScript란?](#-typescript란)
  - [Basic](#-basic)
    - [변수 지정](#-변수-지정)
    - [변수 지정2](#-변수-지정2)
  - [사용 예제](#-사용-예제)

## 📌 TypeScript란?

---

- 기존에 사용하는 JavaScript는 Runtime 언어, 즉 동적 언어이다.<br>
  동적 언어란 실행될 때 변수의 타입이 결정되기 때문에 사용자가 오류를 그대로 볼 수 있다.
- 하지만 TypeScript는 정적 언어이다.
- 정적 언어란 컴파일될 때 변수의 타입이 결정되기 때문에 실행 전에 오류를 볼 수 있습니다.

- 언뜻보면 동적 언어가 자동으로 변수의 타입을 정해주기 때문에 정적 타입보다 동적 타입이 좋아보일 수 있지만...
- 개발자가 의도하지 않았던 동작까지 동적 언어가 처리할 가능성이 있기 때문에 정적 언어로 바꾸어주어 안전하게 코드를 관리하는 것이 선호되고 있다.

=> 이러한 이유로 TypeScript를 Project에 적용시키는 것이 증가하고 있는 추세이다.📈

## 📌 Basic

---

### 🌱 변수 지정

- 먼저 변수를 지정하는 것은 간단하다!

```
const num = 1;
=> const num: number = 1;
```

- 이런 식으로 변수옆에 변수가 무슨 타입의 데이터를 가지는지 명시해주는 것이다.<br>
  (Boolean, string, undefined, null 등이 있을 것이다.)

### 🌱 변수 지정2

- 일반적인 기본형 데이터 타입으로 보지 못했던 것들이 있다.

#### 1. unknown

```
let a: unknown = 0;
a = 'hello';
a = '2';
```

- 아직 값이 정해지지 않았을 경우 unknown을 사용한다.
- TypeScript에서 JavaScript의 라이브러리를 사용할 경우 간혹 사용되곤 한다.

#### 2. any

```
let a: any = 0;
a = 'hello';
a = 2;
```

- any도 unknown과 기능적으로는 비슷하지만 의미론적으로 다르다.
- any는 표현 그대로 '아무거나 다 가능!'이다.
- TypeScript 자체가 정적 언어인데 이러한 특성을 무너뜨리는(?) 특성으로 잘 사용되지 않는다.

#### 3. void

- void는 함수에서 아무것도 리턴되지 않는 경우를 나타낸다.

```
function hello():void {
    return;
}
```

- hello함수는 현재 아무것도 return하고 있지 않다..!
- 이럴 경우 함수의 값으로 void를 선언해주는데, void는 생략이 가능하다! 그래서 이 부분은 각 사마다의 컨벤션에 따라서!!

#### 4. never

- never는 함수가 에러를 리턴하거나 리턴 값 자체를 내보내지 않는 경우(무한 루프)에 사용된다.

```
function error(): never {
    throw new Error(message);
    while(true) {console.log('infinity loop')}
}
```

#### 5. object

- object는 데이터 타입이 object일 경우에 사용 가능하다.

```
function obj(obj: object) {
}
obj({one: 1});
obj({two: 2});
```

## 📌 사용 예제

---

```
function addNum(num1: number, num2: number2): number {
    return num1 + num2
}
```

- 이렇게 각 함수의 매개변수에도 number로 선언해줄수 있고 함수에 대한 return값의 타입도 지정해 줄 수 있다.

---

- TypeScript의 함수에서는 지정된 매개변수를 일일히 선언해주지 않으면 에러가 일어난다.
- 때문에 만일 함수에 매개변수의 값을 넘겨주지 않고 싶을 때는 <b>Optional parameter</b>로 지정해준다.

```
function printName(firstName: string, lastName?: string): string {
    console.log(firstName);
    console.log(lastName);
}
printName(SuYeon, Kim);
printName(SuYeon)
```

- 이렇게 매개 변수 뒤에 ?를 붙여주면 그 매개변수는 전달해주지 않아도 동작하게 된다.

---

- 함수의 매개변수에 전달되는 값이 없을 경우 default값도 지정해줄 수 있다.

```
function greeting(message: string = 'hello') {
    console.log(message);
}
greeting();
```

---

- 함수 매개변수의 타입이 일정할 때, 특정 개수가 지정되지 않고 여러개를 할당하고 싶으면 Rest parameter를 사용한다.

```
function addNum(...numbers: number[]): number {
    return numbers.reduce((a, b) => a + b);
}
console.log(addNum(1, 2)); // 3
console.log(addNum(1, 2, 3)); //6
console.log(addNum(1, 2, 3, 4, 5, 6, 7)); // 28
```

---

- array에서도 사용할 수 있다.

```
const students: string[] = ['철수', '영희'];
const scores: Array<number> = [1, 2, 3];

function printArray(students: readonly string[]) {
}
```

- 보통 string[]로 할 수 있지만 Array<number>같이 선언할 수도 있다.

> 🎈 여기서 readonly는 선언 시, 생성자 내부에서만 값을 할당할 수 있다.(불변성을 가질 수 있다!)

## 📌 정리

---

- TypeScript는 정적 언어이기 때문에 코드의 오류를 최소화하기 위한 목적을 가지고 사용한다.
- 각 변수의 데이터 값에 맞는 타입을 선언해주어야 한다.
- 함수에도 여러 가지 기능들을 활용하며 TypeScript를 사용하자!
