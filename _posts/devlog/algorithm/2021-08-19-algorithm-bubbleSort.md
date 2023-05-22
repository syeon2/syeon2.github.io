---
layout: post
title: "[Algorithm] Bubble Sort"
subtitle: "bubble sort"
categories: devlog
tags: algorithm BubbleSort
---

> 버블 정렬 알고리즘입니다.

<!--more-->

- 목차
  - [Bubble Sort(버블 정렬)란?](#-bubble-sort버블-정렬란)
  - [사용 예제](#-사용-예제)

## 📌 Bubble Sort(버블 정렬)란?

---

- 서로 인접한 두 요소를 검사하여 정렬하는 알고리즘
- 선택정렬과 마찬가지로 시간복잡도 측면에서 O(n<sup>2</sup>)를 가진다.
- 단순하고 구현하기 어렵지 않지만, 많은 데이터를 다루게 된다면 상당히 비효율적인 정렬 방법이 될 수 있다.

## 📌 사용 예제

- 주어진 배열 내에서 오름차순으로 정렬하시오.

```
let arr = [13, 5, 2, 7, 19, 20, 9, 17];

const question = arr => {
    let answer = [];

    while(arr.length > 0) {
        for(let i = 0; i < arr.length; i++) {
            if(arr[i] < arr[i + 1]) {
                [arr[i], arr[i + 1]] = [arr[i + 1], arr[i]];
            }
        }
        answer.push(arr.pop());
    }

    return answer;
}

question(arr);
```

- 이런 방법으로 처음에 구현했었는데 직관적으로 보기에는 정렬 순서가 내림차순 느낌이 나서 가독성에 좋지 않았던 것 같다.

```
let arr = [13, 5, 2, 7, 19, 20, 9, 17];

const question = arr => {
    let answer = arr;

    for(let i = 0; i < answer.length -1; i++) {
        for(let j = 0; j < answer.length - i - 1; j++) {
            if(answer[j] > answer[j + 1]) {
                [answer[j], answer[j + 1]] = [answer[j + 1], answer[j]];
            }
        }
    }

    return answer;
}
```

- 가독성이 훨씬 좋아졌다. 그리고 전보다 불필요한 메서드를 사용하지 않아도 구현이 가능해졌다.
