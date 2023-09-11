---
layout: post
title: "[LeetCode] 208. Implement Trie (Prefix Tree)"
subtitle: "algorithm"
categories: devlog
tags: algorithm tree
---

> LeetCode Top Interview 150의 208번 문제입니다.

<!--more-->

📚 목차
- [🌱 Implement Trie (Prefix Tree)](#-implement-trie--prefix-tree-)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 트리 자료구조를 구현하는 문제이다.
- Tree 생성자와 insert, search, startsWith 메소드를 구현해야한다.

- insert는 단어를 트리 자료구조에 저장해야한다.
- search는 찾고자 하는 단어가 자료구조에 저장되었다면 true를, 없다면 false를 반환한다.
- startsWith은 주어진 단어로 시작하는 단어가 있다면 true, 없다면 false를 반환한다.


- 조건
  - 1 <= word.length, prefix.length <= 2000
  - word and prefix consist only of lowercase English letters.
  - At most 3 * 10<sup>4</sup> calls in total will be made to insert, search, and startsWith.

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

트리라고 한다면 이진트리가 가장 기초적으로 알고 있는 트리구조이다. 하지만 현재 구현해야하는 트리구조는 숫자가 아니라 문자를 저장해야한다. 
문자를 트리 자료구조 형태로 저장할 수 있는 방법을 고민해야하는 것이 이 문제의 관건이다.

문자를 트리 자료구조에 저장할 때 가장 효율적인 방안은 이렇다. 바로 각 문자열에서의 문자를 정수로 변환하는 것이다. 그리고 그 정수에 대한 값을 각 노드의 
자식노드에 추가시킨다.

예를 들어 "apple"이라는 단어가 있다고 가정한다면, apple 중 a는 알파벳의 제일 처음의 단어이다. 따라서 알파벳 순서에 따라 'a'는 인덱스 0에 노드 형태로 저장한다. 
해당 노드는 'a'값이 저장된 자식 노드 리스트의 인덱스 0에 저장됨과 함께 해당 노드는 해당 노드의 자식 노드를 가지고 있다. 알파벳은 총 26개이니 하나의 노드가 생길 때마다, 
그 자식노드는 최대 26개가 생성될 수 있다. 여러 단어들이 생기면 생길 수록 공간복잡도는 기하급수적으로 증가할 것이다. 왜냐하면 노드가 하나 추가될 때마다 자식 노드가 저장될 리스트의 공간 26개를 
할당해야하기 때문이다. 비슷한 형태의 단어까리 모여있는 리스트일수록 필요한 메모리 공간이 작은 폭으로 증가할 것이다. 반면에, 해당 단어가 존재하는지를 판단하는 시간복잡도는 
굉장히 빠르다. 모든 단어를 찾아야하는 기본적인 O(N)의 시간복잡도가 아니라, 단어의 길이만큼 탐색하면 된다.

apple은 a로 만들어진 노드에서 자식노드 p가 존재하는지 파악한다. 또 p는 자식 노드로 p를 가지고 있는지, 그 다음 p에서는 l을, l에서는 e를 가지고 있는지의 여부를 판단한다. 
저장한 단어가 얼마나 많이 존재하는지와는 관계없이 총 5번만 값을 조회하면 해당 단어의 유무를 조회할 수 있다는 장점을 가지고 있다.

> 🥕 시간복잡도는 O(L)이다. L은 조회하려는 단어의 길이아다.
> 
> 🥕 공간복잡도는 최악의 경우 O(26^L)이 될 것이다. L은 저장한 단어들 중 가장 긴 단어의 길이를 의미하며, 그 길이만큼 트리의 depth가 길어지기 때문에 
> 최악의 경우 길이만큼 각각 노드에 26개의 자식노드를 가지고 있어 26의 L제곱이 될 것이다.

---

### 🟤 문제 풀이 (Algorithm)

```java
class Trie {

    private Node head;

    public Trie() {
        this.head = new Node();
    }
    
    public void insert(String word) {
        Node temp = head;

        for (int i = 0; i < word.length(); i++) {
            int index = word.charAt(i) - 'a';

            if (temp.getList()[index] == null) {
                temp.getList()[index] = new Node();
            }

            temp = temp.getList()[index];
        }

        temp.setIsWord(true);
    }
    
    public boolean search(String word) {
        Node temp = head;

        for (int i = 0; i < word.length(); i++) {
            int index = word.charAt(i) - 'a';

            if (temp.getList()[index] == null) {
                return false;
            } else {
                temp = temp.getList()[index];
            }
        }

        if (temp.getIsWord()) return true;
        else return false;
    }
    
    public boolean startsWith(String prefix) {
        Node temp = head;

        for (int i = 0; i < prefix.length(); i++) {
            int index = prefix.charAt(i) - 'a';

            if (temp.getList()[index] == null) {
                return false;
            } else {
                temp = temp.getList()[index];
            }
        }

        return true;
    }

    class Node {
        
        private boolean isWord;
        private Node[] list;

        public Node() {
            this.isWord = false;
            this.list = new Node[26];
        }

        public boolean getIsWord() {
            return this.isWord;
        }

        public void setIsWord(boolean isWord) {
            this.isWord = isWord;
        }

        public Node[] getList() {
            return this.list;
        }
    }
}
```

---
