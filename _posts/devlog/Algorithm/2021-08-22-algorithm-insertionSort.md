---
layout: post
title: "[Algorithm] Insertion Sort"
subtitle: "insertion sort"
categories: devlog
tags: algorithm InsertionSort
---

> 삽입 정렬 알고리즘입니다.

<!--more-->

- 목차
  - [Insertion Sort란?](#-insertion-sort란)
  - [사용 예제](#-사용-예제)

## 📌 Insertion Sort란?

---

- 순서에 맞춰 하나의 요소를 기억한 후에 조건에 맞는 배열의 위치에 삽입시키는 알고리즘이다.
- 버블 정렬과 비슷해보이지만 삽입 정렬의 핵심은 요소를 '기억'하는 것에 있다.
- 각각 옆을 비교하는 것을 요소의 개수만큼 비교해야하는 버블 정렬과 달리 삽입 정렬은 최선의 결과로 시간복잡도 O(n)을 가질 수 있는 알고리즘이다.
- 하지만 다르게 표현하면 최악으로는 역시나 O(n<sup>2</sup>)을 가질 수도 있다는 것으로 데이터의 상태에 따라 성능차이가 심한 알고리즘이다.

## 📌 사용 예제

---

- 주어진 배열을 오름차순으로 정렬하시오.

```
const question = (arr) => {
  let answer = arr;

  function compareProps(arr, i) {
    if(arr[i] > arr[i + 1]) {
      [arr[i], arr[i + 1]] = [arr[i + 1], arr[i]];
      i--;
      compareProps(arr, i);
    }
  }

  for(let i = 0; i < answer.length; i++) {
    compareProps(answer, i);
  }

  return answer;
}

let arr = [11, 7, 5, 6, 10, 9];
```

- 처음에는 양 옆이 정렬 조건에 불일치한다면 바꿔주고 다시 바뀐 요소를 한번 더 바꿔주는 것이 삽입 정렬이라고 생각했다.
- 요소가 바뀌면 계속 체크를 해줘야 할 것 같아서 재귀문을 사용하여 코드를 구현해보았다.
- 가만히 생각해보니 삽입보다는 버블에 가까운 듯한데.. 그렇다고 버블처럼 시간복잡도를 항상 O(n<sup>2</sup>)를 가지지는 않는것 같아서 무슨 정렬인지 잘 모르겠다...(삽입인가..?)

```
let arr2 = [11, 7, 5, 6, 10, 9];

const question2 = (arr) => {
  let answer = arr;

  for(let i = 0; i < answer.length; i++) {
    let tmp = answer[i], j

    for(j = i -1; j >= 0; j--) {
      if(answer[j] > tmp) answer[j + 1] = answer[j];
      else break;
    }
    answer[j+1] = tmp;
    console.log(answer);
  }

  return answer;
}

question2(arr2);
```

- 이 코드가 정석적인 삽입 정렬이다.
- 처음에 요소를 기억한 후, 다음 for문에서 기억된 요소와 해당 요소를 비교한다.
- 비교한 후 바뀌어야한다면 한단계씩 순차적으로 조건에 만족할 때까지 비교하고 바꿔준다.

> 🎈 지금보니까 1번째 코드와 2번째 코드의 차이로 '변수에 할당하는가'의 차이밖에 없는 것 같은데... 삽입에 가까운 것 같다!
