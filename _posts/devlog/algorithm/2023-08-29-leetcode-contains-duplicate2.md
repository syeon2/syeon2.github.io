---
layout: post
title: "[LeetCode] 219. Contains Duplicate II"
subtitle: "algorithm"
categories: devlog
tags: algorithm hash-map
---

> LeetCode Top Interview 150의 219번 문제입니다.

<!--more-->

📚 목차
- [🌱 Contains Duplicate II](#-contains-duplicate-ii)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Contains Duplicate II](https://leetcode.com/problems/contains-duplicate-ii/description/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 1차원 배열과 int 타입 k가 주어진다.


- 1차원 배열 중 같은 값을 가지는 서로 다른 인덱스의 간격이 k와 같거나 작아야한다.
- 위의 조건에 만족하는 경우가 있다면 true를 반환하고 없다면 false를 반환해야한다.


- 조건
  - 1 <= nums.length <= 10<sup>5</sup>
  - -10<sup>9</sup> <= nums[i] <= 10<sup>9</sup>
  - 0 <= k <= 10<sup>5</sup>

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

1차원의 배열을 순환하면서 각 인덱스마다의 값에 접근했을 때 비교할 수 있는 같은 값들이 있는지를 파악해야한다.

true가 될 수 있는 경우는 같은 값을 가진 요소들이 1차원 배열에 k만큼 근접해있는 경우이다. 1차원 배열을 순환하면서 
이전에 자신이 거쳐간 값들 중 자신과 동일한 값이 있었는지, 동일한 값의 인덱스는 몇인지를 트래킹하기 위해 map 자료구조를 활용했다.

map의 key값에 값을, value에 값에 대한 index를 저장한다. index가 1씩 증가하면서 map 자료구조에 해당 값과 중복된 값이 있는지 containsKey API로 확인하고 
있다면 get(key)를 활용하여 현재 인덱스와 get(key)로 받은 인덱스의 차이를 구한다.

Math.abs(i - get(key)) <= k가 만족하면 true를 반환하고, 모든 배열을 순환했지만 해당 조건에 만족하지 않는다면 false를 반환한다.

> 🥕 시간복잡도는 1차원 배열을 한번 순환하는 코드에서 O(N)의 시간복잡도가 소요된다.
> 
> 🥕 공간복잡도는 최악의 경우 배열의 size만큼 map에 값이 할당될 수 있기 때문에 O(N)의 공간복잡도를 가진다. 

---

### 🟤 문제 풀이 (Algorithm)

```java
class Solution {
    public boolean containsNearbyDuplicate(int[] nums, int k) {
        
        Map<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {

            if (!map.containsKey(nums[i])) {
                map.put(nums[i], i);
            } else {

                int num = map.get(nums[i]);

                if (k >= Math.abs(i - num)) return true;
                else map.put(nums[i], i);
            }
        }

        return false;
    }
}
```

---
