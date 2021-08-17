---
layout: post
title: "[Algorithm] Selection Sort"
subtitle: "selection sort"
categories: devlog
tags: algorithm Selection Sort
---

> 선택 정렬 알고리즘입니다.

<!--more-->

- 목차

## 📌 Selection Sort란?

---

- 입력 배열(정렬되지 않는 값들) 이외에 다른 추가 메모리가 필요하지 않다.
- 해당 순서에 원소를 넣을 위치는 이미 정해져 있고, 어떤 원소를 넣을지 선택하는 알고리즘
- 일반적으로 이중 for문을 사용하는 느낌이다. (첫번째 요소를 두번째 요소부터 차례대로 비교하여 조건이 충족하면 첫번째 요소와 조건에 맞는 요소를 바꾸고, 다음 두번째 요소를 세번째 요소부터 비교하는 작업을 반복하여 정렬한다.)

## 📌 사용 예제

---

- 주어진 배열의 요소를 오름차순으로 정렬하시오.

```
const question = (arr) => {
    let answer = arr;

    for(let i = 0; i < answer.length; i++) {
        let idx = i;

        for(let j = i + 1; j < answer.length; j++) {
            if(answer[idx] > answer[j]) idx = j;
        }

        [answer[i], answer[idx]] = [answer[idx], answer[i]];
    }

    return answer;
}

question([13, 5, 7, 3, 9, 16, 2]);
```

- 첫번째 for문에서는 첫번째 요소를 지정해준다.
- 두번째 for문에서는 지정된 첫번째 요소와 두번째 이후부터의 요소들과 비교한다.
- 두번째 for문이 종료되면 첫번째 요소와 조건에 충족된 요소의 위치를 바꾸어준다.
- 이것을 첫번째 for문이 종료될 때까지 반복한다.

🎈 시간복잡도 측면에서는 O(n<sup>2</sup>)으로 느린 속도를 가진다.
