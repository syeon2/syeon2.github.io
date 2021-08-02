---
layout: post
title: "[Algorithm] Two Pointer"
subtitle: "1차원의 배열에서 2개의 포인터를 주어 풀어내는 알고리즘"
categories: devlog
tags: algorithm TwoPointer
---

> 1차원의 배열에서 2개의 포인터를 주어 풀어내는 알고리즘

<!--more-->

- 목차
  - [Two Poiner란?](#-two-pointer란)
  - [사용 예제](#-사용-예제)

## 📌 Two Pointer란?

---

- 의미 그대로 2개의 포인터를 가진다.
- 1차원 배열에 포인터를 2개를 두어 각각 포인터들을 사용해 순차적으로 비교할 수 있다.
- 이중 for문을 사용해도 되지만, 이중 for문은 시간복잡도가 O(N<sup>2</sup>)이고 Two Pointer는 O(N)이어서 시간적으로 효율이 더 좋다.

## 📌 사용 예제

---

- 간단하게 두 배열을 오름차순으로 합치는 알고리즘을 구현

```
let arr1 = [1, 3, 5];
let arr2 = [2, 3, 4, 6, 8];

const twoPointer = (arr1, arr2) => {
   let answer = [];
   let i = z = 0;

    while(i < arr1.length & z < arr2.length) {
       if(arr1[i] < arr2[z]) {
           answer.push(arr1[i++]);
       }
       if(arr1[i] > arr2[z]) {
           answer.push(arr2[z++]);
       }
       if(arr1[i] === arr2[z]) {
        answer.push(arr1[i++], arr2[z++])
       }
   }

   while(i < arr1.length) answer.push(arr1[i++]);
   while(z < arr2.length) answer.push(arr2[z++]);

   return answer;
}

twoPointer(arr1, arr2)
```

- 간단하게 concat으로 합친다음 sort으로 정렬해도 되는 문제라고 생각이 든다.
- 하지만 sort는 최악의 경우 O(N<sup>2</sup>)이 될 수 있기 때문에 시간복잡도 측면에서는 효율이 좋지 못하다.
- Two Pointer 알고리즘은 포인터가 배열의 끝에 다다르면 마무리되기 때문에 시간복잡도 측면에서 sort보다 효율이 좋다.
