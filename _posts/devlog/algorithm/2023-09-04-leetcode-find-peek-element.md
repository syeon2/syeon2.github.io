---
layout: post
title: "[LeetCode] 162. Find Peak Element"
subtitle: "algorithm"
categories: devlog
tags: algorithm binary-search
---

> LeetCode Top Interview 150의 162번 문제입니다.

<!--more-->

📚 목차
- [🌱 Find Peak Element](#-find-peak-element)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Find Peak Element](https://leetcode.com/problems/find-peak-element/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 숫자값이 있는 1차원 배열이 주어진다.
- 배열에서 "피크 값"이라고 할 수 있는 요소의 인덱스를 반환하는 문제이다.

- 피크값이라하는 것은 인접한 값들보다 값이 크면 피크 값이라고 할 수 있다. 가령 [1, 3, 2]일 때 3은 1과 2 사이에 있기 때문에 피크값이라고 할 수 있다.
- 예외로 인덱스 0일 때의 왼쪽 인접한 값과 인덱스 n일 때의 값은 항상 -무한이다.


- 조건
  - 1 <= nums.length <= 1000
  - -2<sup>31</sup> <= nums[i] <= 2<sup>31</sup> - 1
  - nums[i] != nums[i + 1] for all valid i.
  - O(log n)의 시간복잡도로 문제를 풀기

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

방법 1.
해당 문제는 사실 O(N)의 시간복잡도 즉, 한번 배열을 순환하면서 한번에 해결할 수 있는 문제이다.

방법 2.
하지만 해당 문제에서 요구한 것은 O(logN)의 시간복잡도로 문제를 해결하도록 주어졌다. 단순히 1차원 배열을 순환하는 것은 O(logN)의 시간복잡도를 만족하지 못하기 때문에 
다른 방법을 사용해야한다.

탐색 알고리즘 중에서 O(logN)의 시간복잡도를 가지고 있는 것은 바로 이진탐색이다. 이진 탑색은 퀵정렬, 합병정렬 등의 방법처럼 배열의 가운데를 기준으로 값을 찾는 방법이다. 
해당 문제에서 요구하는 것에 접목시키면 다음과 같은 전략을 세울 수 있을 것 같다.

먼저 배열의 가운데 값을 파악한다. 가운데 값이 가운데값과 인접한 인덱스의 값들과 비교하여 값이 크다면 이 값은 peek 값이다. 그렇기 때문에 그대로 가운데 값을 반환하면 되지만, 
만일 만족하지 못한다면 가운데값보다 작은 값이 존재한다는 의미이다. 이 작은 값을 피해 큰값을 가진 배열로 다시 바이너리 탐색을 하는 방법이다.

이 방법을 사용하면 O(logN)의 시간복잡도를 사용해서 문제에서 요구하는 답을 찾아낼 수 있다.

---

### 🟤 문제 풀이 (Algorithm)

방법 1.
```java
class Solution {
    public int findPeakElement(int[] nums) {
        if (nums.length == 1) return 0;

        for (int i = 0; i < nums.length; i++) {
            if (i == 0) {
                if (nums[i] > nums[i + 1]) return i;
            } else if (i == nums.length - 1) {
                if (nums[i - 1] < nums[i]) return i;
            } else {
                if (nums[i - 1] < nums[i] && nums[i + 1] < nums[i]) return i;
            }
        }

        return 0;
    }
}
```

방법 2.
```java
class Solution {
    public int findPeakElement(int[] nums) {
        return binarySearch(nums, 0, nums.length - 1);
    }

    public int binarySearch(int[] list, int start, int end) {
        if (start == end) return start;
        else if (start + 1 == end) {
            if (list[start] > list[end]) return start;
            else return end;
        }

        int mid = (start + end) / 2;

        if (list[mid] > list[mid - 1] && list[mid] > list[mid + 1]) {
            return mid;
        } else if (list[mid] < list[mid - 1]) {
            return binarySearch(list, 0, mid - 1);
        } else {
            return binarySearch(list, mid + 1, end);
        }
    }
}
```

---
