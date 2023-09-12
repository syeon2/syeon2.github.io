---
layout: post
title: "[LeetCode] 212. Word Search II"
subtitle: "algorithm"
categories: devlog
tags: algorithm tree
---

> LeetCode Top Interview 150의 212번 문제입니다.

<!--more-->

📚 목차
- [🌱 212. Word Search II]()
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Word Search II](https://leetcode.com/problems/word-search-ii/description/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 각 문자를 담은 2차원 배열와 단어를 담은 1차원 배열이 주어진다.
- 2차원 배열은 각 문자 하나하나가 저장되어 있으며 마치 보드와 같은 형태를 띈다.
- 이 보드 안에 있는 문자들을 조합하여 1차원 배열에 있는 단어를 만들 수 있는지, 만들 수 있는 단어를 정답으로 반환하는 문제이다.

- 문자들을 조합할 때 상하좌우로 인접한 문자끼리만 조합이 가능하다. 문자가 상하좌우로 인접해있지 않다면 조합할 수 없다.


- 조건
  - m == board.length
  - n == board[i].length
  - 1 <= m, n <= 12
  - board[i][j] is a lowercase English letter.
  - 1 <= words.length <= 3 * 10<sup>4</sup>
  - 1 <= words[i].length <= 10
  - words[i] consists of lowercase English letters.
  - All the strings of words are unique.

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

> 해당 문제는 Hard 문제답게 조금 접근하기 어려웠던 것 같다. 그래서 어떻게 사람들이 풀이했는지를 참고했다.

해당 문제를 풀이한 방법으로는 DFS와 Tree를 적절히 활용하여 문제를 풀이하였다. 각 자료구조와 탐색 알고리즘의 쓰임을 먼저 파악하고자 한다.

Tree 자료구조는 탐색해야할 각 단어들을 저장하는 목적을 가지고 있다. 각 문자들을 저장하고 DFS의 알고리즘에서 문자의 다음 문자 (apple 이라면 a 다음에 p라는 문자)가 
있는지를 현재 board의 위치에서 상하좌우로 재귀를 통해 탐색한다. 만약 상하좌우에 해당 자식 노드의 값이 없다면 해당 문자는 조합할 수 없다는 의미가 된다.

또 한가지는 "#"이라는 문자를 방문한 board의 값에 넣어 이미 그 position은 다녀가서 다시 재귀로 인해 재방문하지 않도록 하는 것이 포인트이다. 상하좌우의 재귀를 모두 
호출하고 난 이후는 다음 단어를 비교해야하기 때문에 다시 board의 "#"을 넣은 위치에 원래 값을 돌려놓아야 한다. 이 부분은 "#"이라는 문자를 활용하는 것도 좋아보이지만, 
많이 사용하는 isVisit이라는 캐시 테이블을 만들어 조금 더 명확하게 문제를 풀이해도 괜찮겠다는 생각을 했다.

> 🥕 시간복잡도 

---

### 🟤 문제 풀이 (Algorithm)

---
