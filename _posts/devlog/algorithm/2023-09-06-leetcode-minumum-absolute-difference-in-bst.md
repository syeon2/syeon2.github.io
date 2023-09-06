---
layout: post
title: "[LeetCode] 530. Minimum Absolute Difference in BST"
subtitle: "algorithm"
categories: devlog
tags: algorithm tree
---

> LeetCode Top Interview 150의 530번 문제입니다.

<!--more-->

📚 목차
- [🌱 Minimum Absolute Difference in BST](#-minimum-absolute-difference-in-bst)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Minimum Absolute Difference in BST](https://leetcode.com/problems/minimum-absolute-difference-in-bst/description/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 이진탐색트리 구조를 가지는 루트 노드를 받는다.
- 각 노드 중 2개를 선택하여 2개의 노드 필드의 차가 최소인 값을 반환하는 문제이다.


- 조건
  - The number of nodes in the tree is in the range [2, 10<sup>4</sup>].
  - 0 <= Node.val <= 10<sup>5</sup>

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

해당 문제를 풀기 위해서는 어떻게 하면 LinkedList로 연결되어있는 노드의 필드를 알아내서 랜덤한 값의 최소값을 구할 수 있을지에 대해 고민해야한다.

방법 1.

먼저 매우 직관적으로 문제를 풀면 다음과 같다.

1. 노드의 각 필드값을 모두 ArrayList에 넣는다.
2. 그리고 O(N<sup>2</sup>)의 시간복잡도를 사용하여 모든 필드의 값을 구한 리스트를 이중 for문으로 순환하며 최소값을 구한다.

위의 문제 중 O(N<sup>2</sup>)의 시간복잡도를 가지는 코드가 제일 많은 시간을 소요할 것이다. 그렇기 때문에 해당 코드를 어떻게하면 더 효율적으로 개선할 수 있을까? 
조금 더 생각해보면 각 요소간의 차이가 최소인 값을 구해 반환하는 문제이다. 각 요소간의 최소값은 각 값과 가장 숫자 상으로 가까운 값과의 차이가 최소가 될 것이다. 즉, 
리스트를 정렬한다면 각 인접한 값들을 O(N)의 시간복잡도로 1차원 순환하며 최소값을 구할 수 있다. (여기서는 모든 노드를 탐색할 때 DFS의 방식인 Stack을 사용하여 모든 노드를 탐색하였다.)

> 🥕 시간복잡도로는 DFS 방식으로 Stack을 사용하여 방문하기 때문에 O(N)의 시간복잡도를 가진다. 또한, 배열을 정렬하는 API의 시간복잡도 O(NlogN)와 O(N)으로 순환하는 코드도 있기 때문에 
> O(2N + N logN) 이며, 총 O(N logN)의 시간복잡도를 가진다.
> 
> 🥕 공간복잡도로는 숫자필드를 저장하는 리스트 O(N)과 DFS로 Node를 저장하는 Stack O(logN)이 사용되었기 때문에 O(2N) => O(N)의 공간복잡도를 필요로 한다.

...

방법 2.

방법 1번 문제를 풀면서 떠오른 아이디어이다. 결국 최소값은 숫자상으로 가까운 값끼리의 차가 최소가 될 수 있다는 아이디어이다.

BST의 특징은 현재 기준 값보다 값이 작다면 왼쪽으로, 크다면 오른쪽으로 자식 노드를 가진다. 그렇기 때문에 자식 노드가 왼쪽으로 뻗어나간 값들을 먼저 리스트에 넣고, 그 이후에 오른쪽으로 뻗어나간 값들을 
리스트에 넣는다면 굳이 정렬을 할 필요없이 이미 정렬되어 나온 리스트를 얻을 수 있다.

이 방법을 사용하면 재귀를 사용하여 모든 노드르 탐색하기 때문에 O(N)의 시간복잡도가 사용되지만, 정렬 API로 사용되는 시간이 필요없어지기 때문에 조금 더 빠른 코드를 완성할 수 있다.

---

### 🟤 문제 풀이 (Algorithm)

방법 1.
```java
import java.util.*;

class Solution {
    public int getMinimumDifference(TreeNode root) {

        List<Integer> list = new ArrayList<>();

        Stack<TreeNode> stack = new Stack<>();
        stack.add(root);

        while (!stack.isEmpty()) {
            TreeNode node = stack.pop();

            list.add(node.val);

            if (node.left != null) stack.add(node.left);
            if (node.right != null) stack.add(node.right);
        }

        int min = Integer.MAX_VALUE;

        List<Integer> sortedList = list.stream().sorted().collect(Collectors.toList());

        for (int i = 0; i < sortedList.size() - 1; i++) {
            min = Math.min(min, sortedList.get(i + 1) - sortedList.get(i));
        }

        return min;
    }
}
```

...

방법 2.

```java
import java.util.*;

class Solution {
    public int getMinimumDifference(TreeNode root) {
        List<Integer> list = new ArrayList<>();

        bfs(list, root);

        int min = Integer.MAX_VALUE;

        for (int i = 0; i < list.size() - 1; i++) {
            min = Math.min(min, list.get(i + 1) - list.get(i));
        }
        
        return min;
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
