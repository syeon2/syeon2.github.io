---
layout: post
title: "[LeetCode] 242. Valid Anagram"
subtitle: "algorithm"
categories: devlog
tags: algorithm hash-table
---

> LeetCode Top Interview 150의 242번 문제입니다.

<!--more-->

📚 목차
- [🌱 Valid Anagram](#-valid-anagram)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Valid Anagram](https://leetcode.com/problems/valid-anagram/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 s, t 2개의 문자열이 주어진다.
- t의 문자열을 재구성하여 s를 만들 수 있는지를 파악하는 문제이다.
- t의 문자열을 재구성할 때 한번 사용한 문자는 재사용이 불가하다.
  - ex) cac 라는 단어가 있다면 ccc는 불가능, acc, cca 등 2개의 c와 한개의 a로 재구성 가능


- 조건
  - 1 <= s.length, t.length <= 5 * 10<sup>4</sup>
  - s and t consist of lowercase English letters.

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

ransom note 문제와 매우 유사한 문제이다. 해당 문제를 풀이한 것과 동일하게 HashMap을 사용하여 
O(N)의 시간복잡도로 해결할 수 있다.

s의 문자열을 map의 자료구조에 넣는다. map의 key값은 문자를, value값은 등장한 횟수를 나타낸다. 
s의 모든 값들을 map 자료구조에 넣었다면 t의 문자열을 순환하며 map과 일치하는 문자의 횟수를 차감한다.

문자의 횟수를 차감할 때, 해당 문자열이 존재하지 않거나 map의 요소가 남아있다면 false를 반환한다.

단, Ransom Note 문제와 약간 다른점은 Ransom note 문제는 b의 문자열 중 a의 문자열에 등장하지 않는 문자가 
있더라도 a를 만들수만 있다면 등장하지 않는 문자열이 있어도 관계없었다. 하지만 이번 문자는 무조건 a와 b의 문자열의 구성이 
일치해야한다. 이 부분을 고려하여 문제를 풀어야한다.

a와 b의 length가 다르면 fasle를 반환하여 빠르게 처리할 수 있다. 이후 위의 절차대로 a와 b의 구성을 비교한다.

> 🥕 시간복잡도는 2개의 문자열을 각각 순환하기 때문에 O(N)의 시간복잡도를 가진다.
> 
> 🥕 공간복잡도는 Map 자료구조로 문자열을 모두 넣어야하기 때문에 최악의 경우 문자열 length 모두 저장할 공간이 필요하다. 
> 그래서 O(N)의 공간복잡도가 필요하다.

---

### 🟤 문제 풀이 (Algorithm)

```java
class Solution {
    public boolean isAnagram(String s, String t) {

        if (s.length() != t.length()) return false;

        Map<Character, Integer> map = new HashMap<>();

        for (int i = 0; i < s.length(); i++) {

            map.putIfAbsent(s.charAt(i), 0);

            map.put(s.charAt(i), map.get(s.charAt(i)) + 1);
        }

        for (int i = 0; i < t.length(); i++) {
            if (map.get(t.charAt(i)) == null) return false;
            else {
                if (map.get(t.charAt(i)) == 1) map.remove(t.charAt(i));
                else map.put(t.charAt(i), map.get(t.charAt(i)) - 1);
            }
        }

        return true;
    }
}
```

---

## 🌱 풀이 개선

Map 자료구조를 사용하여 문제를 푸는 것이라 안내로 나와있지만, 다른 방법으로 풀 수 있는 방법이 있다. 바로 
계수 정렬의 개념을 사용하는 방법이다.

a부터 z까지의 개수는 26개이며 a에서 z는 순차적으로 증가한다. 이 특성을 이용하여 int[] list = new int[26]; 으로 배열을 만든 뒤 
문자 - 'a'를 하게 되면 각 문자를 숫자로 변환한 값의 개수를 list에서 카운팅할 수 있다.

```java
public class Solution {
    public boolean isAnagram(String s, String t) {
        int[] a = new int[26];
        for (int i = 0; i < s.length(); i++) a[s.charAt(i) - 'a']++;
        for (int i = 0; i < t.length(); i++) a[t.charAt(i) - 'a']--;
        for (int i : a) if (i != 0) return false;
        return true;
    }
}
```
