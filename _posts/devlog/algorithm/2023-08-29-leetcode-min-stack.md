---
layout: post
title: "[LeetCode] 155. Min Stack"
subtitle: "algorithm"
categories: devlog
tags: algorithm stack
---

> LeetCode Top Interview 150의 155번 문제입니다.

<!--more-->

📚 목차
- [🌱 Min Stack](#-min-stack)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

- [🌱 풀이 개선](#-풀이-개선)

----

## 🌱 [Min Stack](https://leetcode.com/problems/min-stack/description/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 주어진 조건을 만족하는 MinStack 클래스를 구현하라.


- 구현해야하는 메소드
  - MinStack()은 stack 객체를 초기화하는 생성자이다.
  - void push(int val)은 val이라는 파라미터를 stack에 쌓는 메소드
  - void pop()은 스택 제일 위에 있는(제일 최근에 추가된 값) 값을 삭제하는 메소드
  - int top()은 스택에서 가장 최근에 추가된 값을 반환하는 메소드
  - int getMin()은 추가된 값 중 가장 작은 val값을 반환


- 위의 조건을 만족함과 함께 모든 메소드는 O(1)의 시간복잡도를 가져야한다.

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

방법 1. O(1)의 조건을 지키지 않은 방법..

O(1)의 시간복잡도를 지키기 위해 2가지의 자료구조를 함께 사용하면 해당 문제를 해결할 수 있을 것이라 판단했다. Stack은 본래 Stack을 구현해야하기 때문에 
Stack 자료구조는 필수로 사용하지만 TreeMap 자료구조를 사용해서 getMin() 메소드를 O(1) 시간복잡도로 풀어보려했다.

결과를 보고 알게 되었지만, TreeMap은 일명 Red-Black Tree 자료구조를 사용한다. 루트 노드 값을 기준으로 작은 값들은 왼쪽으로, 큰 값들은 오른쪽으로 보내어 데이터의 
균형을 맞추는 자료구조이다.

TreeMap 구현체는 대체로 O(logN)의 시간복잡도를 가지는 자료구조이기 때문에 문제에서 주어진 O(1)보다 시간이 더 소요된다. 성능은 떨어지지만 정답으로 맞추어 코드를 공유하려 한다.

stack에는 기존에 stack이 동작하는 메커니즘으로 각 메소드가 호출할 때 stack 자료구조에 값들을 할당하거나 삭제, 참조한다. TreeMap을 사용한 이유는 
전적으로 getMin() 메소드를 O(1)로 하기 위해서 구현했지만, TreeSet을 오해하여 O(1)로 구현하지 못했다.


...

방법 2. 노드와 링크드리스트 개념을 활용한 구현방법

MinStack에서 조심해야할 기능은 getMin()이다. 이외의 메소드는 기존 stack과 동일하며 기본적으로 O(1)의 시간복잡도를 가진다.

그렇다면, 링크드리스트 스택에 노드를 추가할 때 해당 노드의 필드에 값을 추가하는 방법을 사용하면 쉽게 해겷할 수 있다. 바로, 해당 노드가 push될 때 
가장 작은 값이 무엇인지를 저장하는 필드를 추가하는 것이다.

그래서 getMin() 메소드가 호출될 때 제일 위에 있는 노드의 min값을 참조하면 해결할 수 있다.

---

### 🟤 문제 풀이 (Algorithm)

방법 1.
```java
class MinStack {

    TreeMap<Integer, Integer> map = new TreeMap<>();
    Stack<Integer> stack = new Stack<>();

    public MinStack() {
        
    }
    
    public void push(int val) {
        map.putIfAbsent(val, 0);

        map.put(val, map.get(val) + 1);
        stack.add(val);
    }
    
    public void pop() {
        Integer num = stack.pop();
        
        int mapNum = map.get(num);

        if (mapNum == 1) map.remove(num);
        else map.put(num, map.get(num) - 1);
    }
    
    public int top() {
        return stack.peek();
    }
    
    public int getMin() {
        return map.firstKey();
    }
}
```

...

방법 2.

```java
class MinStack {

  private Node node;

  class Node {
    private int value;
    private int min;
    private Node prev;
    private Node next;

    public Node() { }

    public Node(int value, int min, Node prev, Node next) {
      this.value = value;
      this.min = min;
      this.prev = prev;
      this.next = next;
    }
  }

  public MinStack() { }

  public void push(int val) {
    if (node == null) {
      node = new Node(val , val, null, null);
    } else {
      node.next = new Node(val, Math.min(node.min, val), node, null);
      node = node.next;
    }
  }

  public void pop() {
    if (node.prev == null) {
      node = null;
    } else {
      node = node.prev;
    }
  }

  public int top() {
    return node.value;
  }

  public int getMin() {
    return node.min;
  }
}
```

---

