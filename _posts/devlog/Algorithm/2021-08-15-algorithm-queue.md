---
layout: post
title: "[Algorithm] Queue"
subtitle: ""
categories: devlog
tags: algorithm Queue
---

> Queue 자료구조를 활용한 알고리즘입니다.

<!--more-->

- 목차

## 📌 Queue란?

---

- 먼저 넣은 데이터를 먼저 나오게 하는 FIFO(First in First Out) 자료구조 형식이다.
- FILO(First in Last Out) 형식인 Stack과 다르게 Queue는 한쪽에서는 삽입 연산, 다른 한쪽은 삭제 연산이 이루어지는 형식이다.
  <img alt='queue' src='/assets/img/algorithm/queue.png' width="500px">

## 📌 사용 예제(요세푸스 순열)

---

- 8명의 사람들을 원형으로 세우고 순서대로 돌아가면서 1부터 번호를 부르는데 3에 해당하는 사람은 제외시킨다.
- 제외가 되면 제외된 다음 사람부터 1을 부르고 마찬가지로 3에 해당되는 사람은 제외시키는 것을 반복한다.

```
const question = (n, k) => {
    let answer;
    let queue = Array.from({length: n}, (v, i) => i + 1); ----- (1)

    while(queue.length) {     -------(2)
        for(let i = 1; i < k; i++) queue.push(queue.shift());
        queue.shift();
        if(queue.length === 1) answer = queue.shift();
    }
    return answer;
}

question(8, 3);
```

- (1)에서 함수형 프로그래밍으로 1부터 8까지의 요소들을 갖는 배열을 만든다.(함수형 프로그래밍 공부해야겠다..🔥)
- (2)에서 queue.length가 0이 될 때까지 반복문을 돌린다.
- 먼저 queue의 첫번째 요소, 두번째 요소를 for문을 사용하여 차례로 queue 배열 뒤에 쌓고 앞의 숫자는 제거한다.
- 그 다음 queue의 첫번째 요소가 되는 숫자는 처음 queue의 3번째 숫자였으므로 뒤에 넣지 않고 제거만 한다.
- 이것을 반복하여 마지막 queue의 요소가 1개가 남으면 그 요소를 answer에 할당한다.

=> 정리하여 앞에 숫자가 조건에 만족하면 배열 마지막 요소로 push하고 shift를 사용하여 제거하지만, 조건에 만족하지 않으면 shift만 사용하여 제거함으로 최후에 남는 데이터를 찾을 수 있는 알고리즘이다.
