---
layout: post
title: "[LeetCode] 2. Add Two Numbers"
subtitle: "algorithm"
categories: devlog
tags: algorithm array linked-list
---

> LeetCode Top Interview 150의 2번 문제입니다.

<!--more-->

📚 목차
- [🌱 Add Two Numbers](#-linked-list-cycle)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Add Two Numbers](https://leetcode.com/problems/add-two-numbers/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 2개의 노드가 주어진다.
- 각 2개의 노드의 자식 노드들에 담겨있는 숫자를 reverse하여 만든 숫자값을 더한다.
- 더한 값을 다시 reverse하여 split하고, split되어 쪼개진 숫자들을 값으로 가지는 linked list를 반환한다.
- => 이 때, 최종적으로 만들어진 split된 배열의 값들은 index 1씩 증가되면 이전의 노드의 자식 노드가 된다.


- 조건
  - The number of nodes in each linked list is in the range [1, 100].
  - 0 <= Node.val <= 9
  - It is guaranteed that the list represents a number that does not have leading zeros.

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

이 문제는 여러가지 잡다한? 구현을 많이 해야하는 문제이다.

방법 1. 먼저 문제에서 요구하는데로 단순히 따라하며 구현해보았다. 파라미터로 주어진 노드의 size가 어느정도 제한되어 있었다면 맞출 수 있었지만, 
[1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ..., 9] 같은 값으로 int나 long으로 변환할 수 없는 값들도 파라미터로 주어질 수 있었다.

...

방법 2. 조금 더 구현해야하는 기능들을 생각하면서 구현해보아야 한다. 방법 1은 에러를 반환하는데 이 에러는 OOME같은 것도 아닌
NumberFormatException 예외이다. 즉, 파리머터로 주어진 배열을 조건에 맞추어 파싱할 때 int, long 같은 타입으로 파싱이 되지 않는 엄청 큰 값이라는 것이다.

이 숫자를 어떻게 계산할 것인지에 대한 부분만 해결한다면 문제를 해결할 수 있을 것이다.

반환해야하는 값은 reverse하여 합친 값을 다시 reverse하여 반환해야한다. 그렇다면 reverse하지 않고 주어진 그대로 처리하되, 자리수의 증가를 
별도로 처리하도록 하는 로직을 구현하면 해결할 수 있을 것 같다.

> 🥕 시간복잡도로는 l1, l2의 자식 노드의 개수만큼 while문이 돌기 때문에 O(N)의 시간복잡도를 가질 것이다.
> 
> 🥕 공간복잡도로는 주어진 링크드 리스트 자료구조 이외에 다른 추가적인 공간이 없기 때문에 O(1)의 공간복잡도를 가질 것이다.

---

### 🟤 문제 풀이 (Algorithm)

방법 1. => 예외 발생
```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {

		Stack<Integer> stack1 = new Stack<>();
		Stack<Integer> stack2 = new Stack<>();

		ListNode l1Start = l1;
		while (l1Start != null) {
			stack1.add(l1Start.val);
			l1Start = l1Start.next;
		}

		ListNode l2Start = l2;
		while (l2Start != null) {
			stack2.add(l2Start.val);
			l2Start = l2Start.next;
		}

		if (stack1.size() > stack2.size()) {
			while (stack1.size() != stack2.size()) {
				stack2.add(0);
			}
		} else if (stack1.size() < stack2.size()) {
			while (stack1.size() != stack2.size()) {
				stack1.add(0);
			}
		}

		StringBuilder sb1 = new StringBuilder();
		while (!stack1.isEmpty()) {
			sb1.append(stack1.pop());
		}

		StringBuilder sb2 = new StringBuilder();
		while (!stack2.isEmpty()) {
			sb2.append(stack2.pop());
		}

		long a = Long.parseLong(sb1.toString());
		long b = Long.parseLong(sb2.toString());

		String[] sum = String.valueOf(a + b).split("");

		int[] list = new int[sum.length];

		for (int i = 0; i < list.length; i++) {
			list[i] = Integer.parseInt(sum[sum.length - i - 1]);
		}

		ListNode answer = new ListNode(list[0]);
		ListNode next = answer;

		for (int i = 1; i < list.length; i++) {
			ListNode temp = new ListNode(list[i]);

			next.next = temp;

			next = temp;
		}

        return answer;
    }
}
```


방법 2.
```java
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {

        ListNode head = new ListNode(0);
        ListNode tail = head;
        int remain = 0;

        while (l1 != null || l2 != null || remain != 0) {
            int digit1 = (l1 != null) ? l1.val : 0;
            int digit2 = (l2 != null) ? l2.val : 0;

            int sum = digit1 + digit2 + remain;
            int digit = sum % 10;
            remain = sum / 10;

            ListNode newNode = new ListNode(digit);
            tail.next = newNode;
            tail = tail.next;

            l1 = (l1 != null) ? l1.next : null;
            l2 = (l2 != null) ? l2.next : null;
        }

        ListNode result = head.next;
        head.next = null;
        return result;
    }
}
```
