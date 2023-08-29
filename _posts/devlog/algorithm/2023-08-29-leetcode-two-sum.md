---
layout: post
title: "[LeetCode] 1. Two Sum"
subtitle: "algorithm"
categories: devlog
tags: algorithm hash-map
---

> LeetCode Top Interview 150의 1번 문제입니다.

<!--more-->

📚 목차
- [🌱 Two Sum](#-two-sum)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

- [🌱 풀이 개선](#-풀이-개선)

----

## 🌱 [Two Sum](https://leetcode.com/problems/two-sum/description/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 1차원 배열, 목표값이 주어진다.
- 주어진 1차원 배열 nums는 정렬되지 않은 숫자 값들이 있는 배열이다.


- 배열 안의 2개의 값을 더해 목표값 target이 될 수 있는 각각 값의 인덱스를 구하는 문제이다.
- 같은 인덱스에 접근하여 더한 값은 허용되지 않으며, 배열 안에는 무조건 target이 될 수 있는 값 2개가 존재한다.


- 조건
  - 2 <= nums.length <= 10<sup>4</sup>
  - -10<sup>9</sup> <= nums[i] <= 10<sup>9</sup>
  - -10<sup>9</sup> <= target <= 10<sup>9</sup>
  - Only one valid answer exists.

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

해당 문제는 정렬되어 있지 않기 때문에 투포인터를 사용하여 문제를 해결할 수 없다.

문제에서 배열 안의 요소들 중 2개의 값의 합은 반드시 target이 될 수 있다고 전제하고 있으므로 2개의 값이 존재하는지 빠르게 판단할 수 있는 방법을 고려해야한다.

인덱스 0의 값을 통해 target을 만들기 위해서는 target - nums[0]의 값이 배열 안에 있어야 한다. 배열은 그저 리스트 자료구조이기 때문에 해당 값이 있는지 
빠르게 조회할 수 없다. 빠르게 해당 값이 있는지 조회하기 위하여 Map을 사용하게 되었다. 먼저 배열을 순환하여 배열의 요소들과 인덱스 정보를 모두 저장한다. 
이후에 다시 배열을 순환하면서 target - nums[0] 값이 map 자료구조에 있는지 확인한다. 이 때, map에 저장된 값의 인덱스와 순환하고 있는 배열의 인덱스가 같지 않아야 한다.

이 방법을 통해 문제를 해결할 수 있다.

> 🥕 시간복잡도는 Map 자료구조에 값들을 저장하는 시간 O(N)과 배열을 순환하며 값이 있는지 확인하는 코드에서 O(N)의 시간복잡도가 소요된다. 
> 총 O(2N) => O(N)의 시간복잡도가 소요된다.
> 
> 🥕 공간복잡도로는 Map 자료구조를 사용하여 최악의 경우 배열만큼의 공간을 추가적으로 사용하기 때문에 O(N)의 공간복잡도를 가진다.

---

### 🟤 문제 풀이 (Algorithm)

```java
class Solution {
  public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();

    for (int i = 0; i < nums.length; i++) {
      map.put(nums[i], i);
    }

    for (int i = 0; i < nums.length; i++) {
      int num = target - nums[i];

      if (map.containsKey(num) && map.get(num) != i) {
        int a = map.get(num);

        return new int[]{a, i};
      }
    }

    return null;
  }
}
```

---

## 🌱 풀이 개선

위의 풀이법보다 더 빠른 방법이 있다. 위의 시간복잡도는 비록 상수이긴 하지만 O(2N)의 시간복잡도를 가진다. 최대한의 효율적인 코드를 작성하기 위해서는 
이러한 부분도 개선할 필요가 있다.

위의 코드에서는 target - nums[i] 의 값이 map이 있는지 없는지 확인하기 위해서 사전에 모든 배열의 값을 map에 넣어야했다. 하지만 배열을 처음부터 순환함과 함께 
현재 인덱스 값과의 합이 target이 될 수 있는지의 여부를 판단하는 방법도 생각해볼 수 있다.

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int num = target - nums[i];

            if (map.containsKey(num)) {
                return new int[]{map.get(num), i};
            }
			
            map.put(nums[i], i);
        }

        return null;
    }
}
```

다른 것은 다 동일하지만 하나 다른 부분을 보자면 map.put(nums[i], i); 코드를 배열을 순환하면서 제일 마지막에 처리한다.

배열을 순환할 때 O(N)의 시간복잡도로 순환하지만 map.containsKey API는 O(1)의 시간복잡도를 가지기 때문에 한번 순환하면 target이 되기위한 합의 요소를 찾을 수 있다.

---
