---
layout: post
title: "[LeetCode] 211. Design Add and Search Words Data Structure"
subtitle: "algorithm"
categories: devlog
tags: algorithm tree
---

> LeetCode Top Interview 150의 211번 문제입니다.

<!--more-->

📚 목차
- [🌱 Design Add and Search Words Data Structure](#-design-add-and-search-words-data-structure)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 문자열 데이터를 추가하고 조회할 수 있는 자료구조를 만드는 문제이다.

- add, search 2개의 메소드를 구현해야한다. 여기서 search의 기능이 조금 독특하다.
  - search는 이전에 저장한 데이터가 있다면 true를 반환한다.
  - 또한, search는 "."이라는 기호가 있는 문자열일 경우, 해당 문자열 중 "."이 있는 문자 자리는 아무 단어가 들어갈 수 있다.


- 조건
  - 1 <= word.length <= 25
  - word in addWord consists of lowercase English letters.
  - word in search consist of '.' or lowercase English letters.
  - There will be at most 2 dots in word for search queries.
  - At most 104 calls will be made to addWord and search.

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

해당 문제를 트리 자료구조 풀기 위해 어떻게 하면 해결할 수 있을지 고민해보았다.

처음에 떠올랐던 방법은 일반적으로 알파벳 개수에 따른 자식노드를 통해 각 자릿수에 있는 문자열의 순서에 맞는 자식노드를 타고 내려간다. 이후 "."이 등장하게 되면 
"." 일 때 그 다음으로 타고 가야하는 자식노드들을 모두 queue에 넣는다. 이후 queue중 제일 앞의 노드를 pop한 다음 그 노드부터 다음 문자와 비교하여 
자식 노드로 타고 내려가는 것을 생각해보았다.

하지만, 해당 방법으로 구현하려고 하니 문제점이 있었다. 첫번째로 queue에 "."대신 들어갈 수 있는 모든 노드들을 넣고 그 중 하나를 pop해 다음 노드와 문자열을 
비교한다고 하면, 이미 비교할 문자열은 다음으로 넘어가기 때문에 다시 queue에 저장되어 있는 다음 노드를 pop하면 그 노드와 비교할 문자열을 탐색하기 까다로웠다. 
노드에 비교할 문자열의 index까지 첨부하여 queue에 넣으면 어떻게든 해결할 수 있을 것 같지만, 굉장히 큰 코드 복잡도가 요구될 것 같았다.

따라서 이 방법 대신 다른 방법을 고민해보았는데, 바로 이진탐색같은 재귀를 사용한 탐색 방법을 사용하여 값을 구하는 방법이다.

탐색해야하는 문자열이 알파벳이라면 탐색할 문자열의 알파벳 순서와 일치하는 자식노드의 순서에 저장되어 있는 노드를 타고 다음 자식 알파벳 노드를 탐색한다. 
이 때 "."이 등장하게 되면 현재 노드에서 저장된 자식 노드들을 대상으로 재귀호출한다. 이 재귀를 통해 문자열 마지막까지 탐색하고 값이 저장되어 있다면 true를, 
값이 저장되어 있지 않거나 도중에 다음으로 타고 갈 자식노드가 없다면 false를 반환한다.

> 🥕 시간복잡도는 알파벳 개수에 따라 자식노드를 생성하기 때문에 O(L) 만큼 소요된다. 하지만 여기서 "."이 등장하게 되면 최악의 경우 모든 자식 노드를 재귀로 탐색해야하기 
> 때문에 O(L * N^점의 개수)의 시간복잡도가 소요될 것이다.
> 
> 🥕 공간복잡도는 각 노드마다 알파벳 개수만큼 26 size의 배열을 생성해야하기 때문에 O(26^L)의 공간복잡도를 가진다.

---

### 🟤 문제 풀이 (Algorithm)

```java
class WordDictionary {

    private Node head;

    public WordDictionary() {
        this.head = new Node();
    }
    
    public void addWord(String word) {
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
        return searchNodes(this.head, word);
    }

    public boolean searchNodes(Node node, String word) {
		if (word.isEmpty()) {
			return node.getIsWord();
		};

		if (word.charAt(0) == '.') {

			for (int i = 0; i < node.getList().length; i++) {
				if (node.getList()[i] == null) continue;
				
				if (searchNodes(node.getList()[i], word.substring(1))) {
					return true;
				}
			}

			return false;
		} else {
			int index = word.charAt(0) - 'a';

			if (node.getList()[index] == null) {
				return false;
			} else {
				return searchNodes(node.getList()[index], word.substring(1));
			}
		}
    }

    class Node {

        private boolean isWord;
        private Node[] list;

        public Node() {
            this.isWord = isWord;
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

        public void setNodeInIndex(Node node, int index) {
            this.list[index] = node;
        }
    }
}
```

---
