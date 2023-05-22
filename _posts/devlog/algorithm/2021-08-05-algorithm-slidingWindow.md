---
layout: post
title: "[Algorithm] Sliding Window"
subtitle: "배열에서 일정한 간격을 유지하면서 이동하며 탐색하는 알고리즘"
categories: devlog
tags: algorithm TwoPointer
---

> 배열에서 일정한 간격을 유지하면서 이동하며 탐색하는 알고리즘

<!--more-->

- 목차
  - [Sliding Window란?](#-sliding-window란)
  - [사용 예제](#-사용-예제)

## 📌 Sliding Window란?

---

- 배열같은 형태를 띈 자료 구조에서 일정한 간격을 유지하면서 탐색하는 알고리즘 기법이다.
- 이중 for문보다 시간복잡도 측면에서 효율이 좋다.

## 📌 사용 예제

---

🎈 배열 안에서 연속되는 3개의 수의 합 중 가장 큰 합을 구하시오.

```
// For문을 사용했을 경우
const useFor = (m, arr) => {
    let answer = 0;

    for(let i = 0; i < arr.length - m + 1; i++) {
        let sum = 0;

        for(let z = i; z < i + m; z++) {
            sum += arr[z];
            answer = Math.max(answer, sum);
        }
    }
    return answer;
}
```

- 이중 For문을 사용하게 되면 시간복잡도로 최대 O(N<sup>2</sup>)가 될 수 있다.

---

```
const useSliderWindow = (m, arr) => {
    let answer = 0;
    let sum = 0;

    for(let i = 0; i < m; i++) {
        sum += arr[i];
    }

    for(let i = m; i < arr.length; i++) {
        sum += arr[i] - arr[i - m];
        answer = Math.max(answer, sum);
    }
    return answer;
}
```

- 이중 For문과 달리 Slider Window는 배열의 처음과 끝을 <b>한번만</b>탐색한다. (시간 효율이 좋다);
- 일정한 간격을 유지하기 때문에 범위의 앞과 뒤에서는 삭제와 삽입이 일어난다. (Deque 자료구조)
