---
layout: post
title: "[LeetCode] 141. Linked List Cycle"
subtitle: "algorithm"
categories: devlog
tags: algorithm array linked-list
---

> LeetCode Top Interview 150의 141번 문제입니다.

<!--more-->

📚 목차
- [🌱 Linked List Cycle](#-linked-list-cycle)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

- [🌱 풀이 개선](#-풀이-개선)

----

## 🌱 [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/description/)

### 🟤 문제 정의 (Definition)

- 파라미터로 이미 만들어진 class NodeList의 인스턴스 하나를 받는다.
- 주어진 객체 인스턴스는 Linked List 방식으로 필드가 구현되어 있다.

- 주어진 링크드리스트 자료구조를 가진 인스턴스가 순환참조를 하고 있는지의 여부를 파악하는 문제이다.


- 조건
  - The number of the nodes in the list is in the range [0, 10<sup>4</sup>].
  - -10<sup>5</sup> <= Node.val <= 10<sup>5</sup>
  - pos is -1 or a valid index in the linked-list.

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

이 문제는 너무 간단하게 풀었다. (하지만 성능이 좋지 않다는 것을 알게 되어 이것 말고 또 무슨 방법이 있지? 생각이 들었다.)

while 문을 사용하여 주어진 링크드 리스트의 next값이 null이 되면 false를 반환하고, 자료구조를 순환하며 Node를 저장하여 
이미 추가된 노드라면 순환한다는 의미이기 때문에 true를 반환한다.

> 🥕 시간복잡도는 링크드리스트의 최악의 경우 링크드리스트 next 전체를 순환하게 되기 때문에 O(N)의 시간복잡도를 가진다.
> 
> 🥕 공간복잡도로는 Node의 정보를 트래킹할 Set 자료구조를 사용하기 때문에 O(N)의 공간복잡도를 가진다.

---

### 🟤 문제 풀이 (Algorithm)

```java
public class Solution {
  public boolean hasCycle(ListNode head) {
    Set<ListNode> set = new HashSet<>();

    ListNode start = head;

    while (start != null) {
      if (!set.contains(start)) {
        set.add(start);
        start = start.next;
      } else {
        return true;
      }
    }

    return false;
  }
}
```

## 🌱 풀이 개선

이 방법이 최선이라고 생각했는데 아니었다. 더 효율적인 방법이 있다고 하여 나름 고민해봤지만 떠오르지 않았다..ㅠ

더 효율적으로 문제를 풀 수 있는 방법의 핵심은 "순환"을 이용해야하는 것이다. 위의 방법도 나름 순환하게 되면 이미 저장한 중복의 값이 
생기기 때문에 순환을 이용한 방법이라고 할 수 있겠다. 하지만 이러한 방법이 있다.

```java
public class Solution {
    public boolean hasCycle(ListNode head) {
        ListNode fast = head;
        ListNode slow = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;

            if (slow == fast) return true;
        }

        return false;
    }
}
```

이 방법은 하나의 순환되는 사이클에서 2개의 포인터를 두어 언젠가 만나는 성질을 이용하여 푼 방식이다. 
fast 노드는 언젠가 slow 노드를 따라잡을 것이고, 각 두 노드가 움직이는 거리만큼의 최소공배수만큼 움직여 만나기 때문에 1부터 N까지 
모두 순환하는 것보다 더 빠르게 동작할 수 있다. 또한, 추가적인 공간이 필요하지 않기 때문에 공간복잡도도 O(1)로 빠르다.
