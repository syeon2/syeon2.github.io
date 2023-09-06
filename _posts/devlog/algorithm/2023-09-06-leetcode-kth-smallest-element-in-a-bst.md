---
layout: post
title: "[LeetCode] 230. Kth Smallest Element in a BST"
subtitle: "algorithm"
categories: devlog
tags: algorithm tree
---

> LeetCode Top Interview 150의 230번 문제입니다.

<!--more-->

📚 목차
- [🌱 Kth Smallest Element in a BST](#-kth-smallest-element-in-a-bst)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 BST 구조를 가지는 루트 노드와 int 값 k가 주어진다.
- 각 노드의 모든 필드 값 중 k번째로 작은 값을 구하는 문제이다.


- 조건
  - The number of nodes in the tree is n.
  - 1 <= k <= n <= 10<sup>4</sup>
  - 0 <= Node.val <= 10<sup>4</sup>

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

해당 문제는 문제 [530번(Minimum Absolute Difference in BST)](https://syeon2.github.io/devlog/leetcode-minumum-absolute-difference-in-bst.html)와 매우 유사하게 풀 수 있다. 
결국 LinkedList로 구성되어 있는 각 노드의 필드를 리스트 형태로 오름차순으로 정렬하면, 자연스럽게 K번째로 작은 값을 index에 접근하여 찾을 수 있다.

재귀를 사용하여 왼쪽 자식 노드와 오른쪽 자식 노드로 다시 재귀호출하되, 왼쪽 자식 노드를 가진 재귀 함수를 먼저 호출하고 현재 노드 값을 리스트에 추가한 다음 오른쪽 자식 노드를 사용한 재귀 함수를 호출해야한다. 
그 이유는 현재 값을 리스트에 추가하기 전 왼쪽 자식 노드로 재귀 호출하게 되면 왼쪽의 제일 끝 노드에 도달한 후에 다음 코드 라인을 실행하기 때문이다. 이러한 진행 방식은 자연스럽게 오름차순으로 정렬할 수 있도로 만들어 준다.

> 🥕 시간복잡도로는 전체 노드를 탐색하기 때문에 O(N)의 시간복잡도가 쇼요된다.
> 
> 🥕 공간복잡도는 각 노드의 필드를 저장할 list O(N)과 재귀 함수 호출로 생겨나는 O(N)으로 총 O(2N) => O(N)의 공간복잡도를 가진다. 

---

### 🟤 문제 풀이 (Algorithm)

```java
class Solution {
    public int kthSmallest(TreeNode root, int k) {
        List<Integer> list = new ArrayList<>();

        bfs(list, root);

        return list.get(k - 1);
    }

    public void bfs(List<Integer> list, TreeNode node) {
        if (node == null) return;

        bfs(list, node.left);
        list.add(node.val);
        bfs(list, node.right);
    }
}
```

---
