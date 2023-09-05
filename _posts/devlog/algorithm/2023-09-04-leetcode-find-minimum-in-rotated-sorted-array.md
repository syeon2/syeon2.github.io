---
layout: post
title: "[LeetCode] 153. Find Minimum in Rotated Sorted Array"
subtitle: "algorithm"
categories: devlog
tags: algorithm binary-search
---

> LeetCode Top Interview 150의 153번 문제입니다.

<!--more-->

📚 목차
- [🌱 Find Minimum in Rotated Sorted Array](#-find-minimum-in-rotated-sorted-array)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 1차원 배열이 주어진다.
- 주어진 1차원 배열 (nums)는 본래 오름차순이지만 한번 회전하여 특정 한 지점을 제외한 오름차순이다.
- 예를 들어 [1, 2, 3, 4, 5] 배열이 있다면 회전하여 [3, 4, 5, 1, 2]같은 구성이 되었다는 것이다.
- 주어진 배열 안의 요소 중 가장 작은 값을 반환하는 문제이다.


- 조건
  - n == nums.length
  - 1 <= n <= 5000
  - -5000 <= nums[i] <= 5000
  - All the integers of nums are unique.
  - nums is sorted and rotated between 1 and n times.

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

방법 1.
해당 문제를 가장 간단히 풀 수 있는 방법은 당연히 O(N)의 시간복잡도로 문제를 해결하는 것이다.

방법 2.
하지만 해당 문제도 O(logN)의 시간복잡도로 문제를 해결하도록 요구사항이 존재한다.

당연히 이진탐색을 사용해서 문제를 풀기로 한다. 해당 문제를 재귀 함수 호출로 풀기 위해 고민해보았다. 
비교값이 외부에서 주어진 것이 아니라 내부에서 가장 값이 작은 값을 반환해야한다. 이 말의 의미는 내부의 값끼리 
서로 비교해야한다는 것이다. 먼저 binarySearch의 개념을 활용하여 mid를 나누고 mid를 기준으로 왼쪽, 오른쪽 영역을 다시 재귀로 호출하여 
비교하도록 했다.

> 해당 방법은 이진 탐색이라고 오해했다. 하지만 이 방법은 이진탐색 방법을 모방한 그저 재귀함수 호출로 O(N)의 시간복잡도를 가지는 
> 풀이법이다. 왼쪽 오른쪽 중 택 1을 하지 않고 모든 나누어진 배열을 비교한다. 결국 start와 end가 서로 같아지는 지점까지 무조건 탐색하기 때문에 
> 결국 O(N)의 시간복잡도를 갖는다.

방법 3.
방법 2의 풀이법을 개선하기 위해서는 내부적으로 추가적인 코드가 필요하다. 왼쪽 오른쪽 둘다 탐색하지 않고 둘 중 하나를 택1하여 
탐색하는 방법을 고안해야한다.

해당 배열은 회전한 배열이기 때문에 만일 mid값이 end값보다 크다면 이는 회전의 영향을 받았기 때문이다. 즉 mid의 오른쪽에 최솟값이 
있다는 의미를 가지고 있다. 이와 반대라면 최솟값은 mid의 왼쪽에 있다는 의미이기 때문에 이를 바탕으로 반복하여 탐색하면 이진탐색을 
사용하여 해당 문제를 풀이할 수 있다.

방법 4.
이진 탐색을 재귀 호출 함수를 사용하는 방법 이외에 while문을 사용하면 더욱 가독성 있고 의도를 파악할 수 있는 코드가 되기도 한다.

while문을 사용하여 방법 3에서 제시했던 왼쪽 오른쪽 탐색을 결정하는 조건문을 넣으면 재귀를 사용하지 않고도 문제를 해결할 수 있다.


---

### 🟤 문제 풀이 (Algorithm)

방법 1.
```java
class Solution {
    public int findMin(int[] nums) {
        
        int min = Integer.MAX_VALUE;

        for (int i = 0; i < nums.length; i++) {
            min = Math.min(min, nums[i]);
        }

        return min;
    }
}
```

방법 2.
```java
class Solution {
    public int findMin(int[] nums) {
        return binarySearch(nums, 0, nums.length - 1);
    }

    public int binarySearch(int[] list, int start, int end) {
        if (start == end) return list[start];
        else if (start + 1 == end) {
            if (list[start] < list[end]) return list[start];
            else return list[end];
        }

        int mid = (start + end) / 2;

        int answer = list[mid];

        answer = Math.min(answer, binarySearch(list, mid + 1, end));
        answer = Math.min(answer, binarySearch(list, start, mid));
        
        return answer;
    }
}
```

방법 3.
```java
class Solution {
    public int findMin(int[] nums) {
        return binarySearch(nums, 0, nums.length - 1);
    }

    public int binarySearch(int[] list, int start, int end) {
        if (start == end) return list[start];
        else if (start + 1 == end) {
            if (list[start] < list[end]) return list[start];
            else return list[end];
        }

        int mid = (start + end) / 2;

        int answer = list[mid];

        if (list[mid] > list[end]) {
            return binarySearch(list, mid + 1, end);
        } else {
            return binarySearch(list, start, mid);
        }
    }
}
```

방법 4.
```java
class Solution {
    public int findMin(int[] nums) {
        
        int left = 0;
        int right = nums.length - 1;

        while (left < right) {
            int mid = (left + right) / 2;

            if (nums[mid] > nums[right]) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }

        return nums[right];
    }
}
```

---
