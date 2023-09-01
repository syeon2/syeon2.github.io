---
layout: post
title: "[LeetCode] 383. Ransom Note"
subtitle: "algorithm"
categories: devlog
tags: algorithm hash-table
---

> LeetCode Top Interview 150의 383번 문제입니다.

<!--more-->

📚 목차
- [🌱 Ransom Note](#-ransom-note)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Ransom Note](https://leetcode.com/problems/ransom-note/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 2개의 문자열이 주어진다.
- 각 문자열을 a와 b로 표기하면, a를 구성하고 있는 문자들의 구성을 b가 가지고 있는 문자들의 구성으로 만들 수 있는지의 여부를 
파악하는 문제이다.
- b에서 a에 사용되고 있는 문자열인지 파악하기 위한 조건이 있다.
  - b에서 a를 구성하는 문자열로 선택한 문자는 2번 이상 선택할 수 없다.


- 조건
  - 1 <= ransomNote.length, magazine.length <= 10<sup>5</sup>
  - ransomNote and magazine consist of lowercase English letters.
---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

해당 문제는 HashMap 자료구조를 사용한 시간복잡도 O(N)으로 간단하게 풀 수 있다.

먼저, a에 사용된 문자열들을 Map 자료구조에 넣는다. 이 때 key값은 문자를, value값은 얼마나 해당 문자가 등장하는지의 개수를 넣는 것이다.
a에 대한 문자들을 map에 다 넣었다면 b의 문자열을 순환하면서 일치하는 문자를 하나씩 제거한다.

만일 제거하려고 하는 값이 map에 존재하지 않거나 b에 대한 값을 모두 제거했는데 map에 일부 문자가 남았다면 문제에서 요구하는 
조건을 충족하지 못하기 때문에 false를 반환한다.

> 🥕 시간복잡도로는 a 문자열을 1번 순환하는데 O(N), b 문자열을 한번 순환하는데 O(N) 총 O(2N) => O(N)의 시간복잡도가 소요된다.
> 
> 🥕 공간복잡도는 map 자료구조를 사용하여 N만큼의 공간을 사용하기 때문에 O(N)의 공간복잡도를 가진다.

---

### 🟤 문제 풀이 (Algorithm)

```java
class Solution {
    public boolean canConstruct(String ransomNote, String magazine) {

        Map<Character, Integer> map = new HashMap<>();

        for (int i = 0; i < magazine.length(); i++) {
            char c = magazine.charAt(i);

            map.putIfAbsent(c, 0);
            map.put(c, map.get(c) + 1);
        }

        for (int i = 0; i < ransomNote.length(); i++) {
            char c = ransomNote.charAt(i);

            if (!map.containsKey(c) || (map.containsKey(c) && map.get(c) == 0)) return false;

            map.put(c, map.get(c) - 1);
        }

        return true;
    }
}
```

---
