---
layout: post
title: "[LeetCode] 167. Two Sum II - Input Array Is Sorted"
subtitle: "algorithm"
categories: devlog
tags: algorithm array two-pointer
---

> LeetCode Top Interview 150의 167번 문제입니다.

<!--more-->

📚 목차
- [🌱 Two Sum II - Input Array Is Sorted](#-two-sum-ii---input-array-is-sorted)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 오름차순으로 정렬된 배열과 목표 값이 주어진다.

- 문제의 요구사항은 배열 안에 있는 요소 2개를 더하여 목표값이 될 수 있는 경우를 찾아 요소 2개의 인덱스 값을 반환하는 것이다.

- 특이사항은 이번 배열은 1-indexed의 특징을 가진 배열이다. 본래 배열은 index 0부터 시작해야하지만, 이 배열은 index 1부터 시작한다.
- 또한, 2개의 인덱스 값은 서로 같을 수 없다. 즉, 배열의 서로 다른 index에 접근한 값의 합이 목표값과 일치해야한다. (1 <= index<sub>1</sub> < index<sub>2</sub>)
- 배열에는 반드시 목표값과 일치할 수 있는 2개의 index 합이 될 수 있는 조건을 가진 값이 존재한다.


- 조건
  - 2 <= numbers.length <= 3 * 10<sup>4</sup>
  - -1000 <= numbers[i] <= 1000
  - numbers is sorted in non-decreasing order.
  - -1000 <= target <= 1000
  - The tests are generated such that there is exactly one solution.

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

해당 문제는 전형적인 투 포인터 방법으로 풀 수 있다. 이번 문제에서 투 포인터를 사용할 수 있도록 미리 준비해둔 것은 바로 오름차순으로 정렬된 배열이다.

- 포인터를 한개는 index 0에, 다른 한개는 index array.length - 1 에 두고 알고리즘을 실행한다.
- 알고리즘은 이렇게 구현한다.
  - index 0과 index array.length - 1에 있는 두개의 값을 더한다.
  - 더한 sum이 target보다 작으면 index 0을 1 상승시킨다. => 이렇게 더한 값이 목표값보다 작을 때 왼쪽에 있는 포인터 (index 0부터 시작한) 값을 1씩 상승시키면 지금의 sum보다 한단계 더 큰 값을 얻을 수 있다. (배열이 정렬되어 있기 때문에!!)
  - 더한 sum이 target보다 크다면 index array.length - 1에 있는 index를 1 감소시킨다. => 위와 같은 이유로 오른쪽에 있는 포인터 index를 1 감소시키면 현재 값보다 한단계 작은 값을 얻을 수 있다.
- 이것을 반복하면 문제에서 요구하는 index 2개를 찾을 수 있다.

> 🥕 시간복잡도는 O(N)이다. 투포인터는 서로 엇갈려 증감할 수 없다. 그렇기 때문에 2개의 포인터는 하나의 배열 안에서 조건을 검사하기 때문에 O(N)의 시간복잡도가 소요된다.
> 
> 🥕 공간복잡도는 추가된 배열 및 가변공간이 없기 때문에 O(1)의 공간복잡도를 가진다.
---

### 🟤 문제 풀이 (Algorithm)

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {

        int left = 0;
        int right = numbers.length - 1;

        while (true) {
            if (numbers[left] + numbers[right] == target) break;

            if (numbers[left] + numbers[right] > target) {
                right--;
            } else {
                left++;
            }
        }

        int[] answer = {left + 1, right + 1};

        return answer;
    }
}
```

