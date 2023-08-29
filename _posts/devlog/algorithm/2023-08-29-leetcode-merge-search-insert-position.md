---
layout: post
title: "[LeetCode] 35. Search Insert Position"
subtitle: "algorithm"
categories: devlog
tags: algorithm binary-search
---

> LeetCode Top Interview 150의 35번 문제입니다.

<!--more-->

📚 목차
- [🌱 Search Insert Position](#-search-insert-position)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Search Insert Position](https://leetcode.com/problems/search-insert-position/description/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 오름차순으로 정렬된 배열과 int 목표값이 주어진다.
- 목표값과 배열 요소에 동일한 값이 있을 경우 해당 요소의 인덱스를 반환하고 없을 경우에는 해당값이 삽입되었을 경우 삽입될 인덱스의 값을 반환한다.
- O(logN)의 시간복잡도를 가지는 알고리즘으로 풀어야한다.

- 조건
  - 1 <= nums.length <= 10<sup>4</sup>
  - -10<sup>4</sup> <= nums[i] <= 10<sup>4</sup>
  - nums contains distinct values sorted in ascending order.
  - -10<sup>4</sup> <= target <= 10<sup>4</sup>

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

문제에서 요구한 것은 O(N)의 시간복잡도가 아닌 O(logN)의 시간복잡도를 가진 알고리즘으로 풀이해야한다. 

이미 배열이 정렬되어 있으면서 O(logN)의 시간복잡도로 문제를 풀 수 있는 알고리즘은 Binary Search 알고리즘이다.

Binary Search 알고리즘의 핵심은 pivot이다. pivot이라고 명명한 배열의 가운데 값과 조회할 값을 비교하여 pivot이 크면 
조회할 값은 pivot을 기준으로 해당값이 왼쪽에 있다는(index 0으로 향하는) 것이 된다. (오름차순으로 정렬되어 있기 때문에!) 

이러한 특징을 활용하여 binary search 알고리즘을 활용하면 O(logN)의 시간복잡도로 문제를 해결할 수 있다.

> 🥕 시간복잡도는 O(logN)이다.
> 
> 🥕 공간복잡도는 binarySearch를 logN만큼 재귀적으로 호출하기 때문에 O(logN)이 될 수 있다.

---

### 🟤 문제 풀이 (Algorithm)

```java
class Solution {
    public int searchInsert(int[] nums, int target) {
        
        return binarySearch(nums, 0, nums.length - 1, target);
    }

    public int binarySearch(int[] list, int start, int end, int target) {
        int result = -1;

        if (start == end && list[start] != target) {
            if (list[start] > target) return start;
            else return start + 1;
        }
        else if (start == end && list[start] == target) return start;

        int pivot = (start + end) / 2;

        if (target <= list[pivot]) {
            result = binarySearch(list, start, pivot, target);
        } else {
            result = binarySearch(list, pivot + 1, end, target);
        }

        return result;
    }
}
```

---
