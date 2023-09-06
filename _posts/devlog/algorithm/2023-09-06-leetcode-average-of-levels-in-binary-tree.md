---
layout: post
title: "[LeetCode] 637. Average of Levels in Binary Tree"
subtitle: "algorithm"
categories: devlog
tags: algorithm tree
---

> LeetCode Top Interview 150의 637번 문제입니다.

<!--more-->

📚 목차
- [🌱 Average of Levels in Binary Tree](#-average-of-levels-in-binary-tree)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Average of Levels in Binary Tree](https://leetcode.com/problems/average-of-levels-in-binary-tree/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 트리 구조의 루트 노드를 제공한다.
- 각 depth를 기준으로 존재하는 수들의 평균을 배열에 저장하여 반환하는 문제이다.


- 조건
  - The number of nodes in the tree is in the range [1, 10<sup>4</sup>].
  - -2<sup>31</sup> <= Node.val <= 2<sup>31</sup> - 1

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

각 depth에 있는 노드에 접근할 때마다 수의 총 함계와 얼만큼 접근했는지를 트래킹할 수 있는 클래스를 별로도 정의했다. 
재귀함수로 각 depth에 있는 모든 노드들을 탐색한다. depth를 인덱스 삼은 리스트에 해당 depth에 접근할 때마다 접근했던 노드의 숫자필드를 합하고 현재 depth에서 접근 횟수를 카운팅한다. 

```java
class Node {

    double sum;
    int count;

    public Node(int sum, int count) {
        this.sum = sum;
        this.count = count;
    }
}
```

---

### 🟤 문제 풀이 (Algorithm)

```java
class Solution {

    public List<Double> averageOfLevels(TreeNode root) {
        List<Node> list = new ArrayList<>();

        bfs(list, root, 0);

        List<Double> answer = new ArrayList<>();

        for (int i = 0; i < list.size(); i++) {
            Node node = list.get(i);

            double a = (double) node.sum / (double) node.count;

            answer.add(a);
        }

        return answer;
    }

    public void bfs(List<Node> list, TreeNode node, int depth) {
        if (node == null) return;

        if (list.size() <= depth) {
            list.add(new Node(node.val, 1));
        } else {
            Node n = list.get(depth);

            n.sum += node.val;
            n.count += 1;
        }

        if (node.left != null) bfs(list, node.left, depth + 1);
        if (node.right != null) bfs(list, node.right, depth + 1);
    }

    class Node {

        double sum;
        int count;

        public Node(int sum, int count) {
            this.sum = sum;
            this.count = count;
        }
    }
}
```

---
