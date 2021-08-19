---
layout: post
title: "[Algorithm] Hash"
subtitle: "임의의 길이의 데이터를 고정된 길이의 데이터로 변환시키는 알고리즘"
categories: devlog
tags: algorithm Hash
---

> 임의의 길이의 데이터를 고정된 길이의 데이터로 변환시키는 알고리즘입니다.

<!--more-->

- 목차
  - [Hash란?](#-hash란)
  - [사용 예제](#-사용-예제)

## 📌 Hash란?

---

- [사전적 정의](https://ko.wikipedia.org/wiki/%ED%95%B4%EC%8B%9C_%ED%95%A8%EC%88%98){:target="\_blank"}로 임의의 길이를 가지고 있는 데이터를 고정된 길이의 데이터로 변환시켜주는 자료 구조이다.
- Hash는 내부적으로 배열을 사용하여 데이터를 저장하기 때문에 빠른 검색 속도를 갖는다.
- 데이터를 추가/삭제 시 기존 데이터를 밀어내거나 당기는 작업이 없도록 하는 알고리즘을 이용하여 데이터와 연관된 고유의 숫자를 만들어 낸 뒤 이를 인덱스로 사용한다. ([출처](https://jroomstudio.tistory.com/10){:target="/\_blank"})

=> 쉽게 설명하자면 식당 메뉴판에 정렬된 메뉴를 볼때는 순차적으로 위에서 아래로 훑는게 아니라 아예 메뉴 이름에 대한 값을 알고 있는 것이라고 비유할 수 있을 것 같다.

## 📌 사용 예제

---

- array안에 있는 과일들 중 가장 많이 들어있는 과일을 찾으시오.

```
let fruits = ['사과', '바나나', '배', '사과', '사과', '배', '배', '사과', '사과']
const question = (fruits) => {
    let answer;
    let hash = new Map();

    for(let i of fruits) {
        if(hash.has(i)) hash.set(i, hash.get(i) + 1);
        else hash.set(i, 1);
    }

    let max = 0;
    for(let [key, value] of hash) {
        if(value > max) {
            max = value;
            answer = key;
        }
    }
    return answer; // '사과'
}
```

- for문을 사용한 것은 배열 안에 있는 요소들이 map을 사용한 hash 안에 있는지 없는지 탐색하기 위함이다.
- 키값에 대한 데이터를 저장할 때 배열의 특징이 아닌 객체같은 특징을 보여준다.(데이터를 추가시 밀어내거나 당기는 이벤트가 발생하지 않는 부분)
