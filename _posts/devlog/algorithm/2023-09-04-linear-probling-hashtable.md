---
layout: post
title: "[Data Structure] Linear Probing HashTable"
subtitle: "algorithm"
categories: devlog
tags: algorithm hash-table
---

> Linear Probing으로 hash collision(해시 충돌)을 해결할 자료구조 HashTable 입니다.

<!--more-->

📚 목차
- [🌱 Linear Probing Hash Table](#-linear-probing-hash-table)
  - [🟤 Hash Table이란?](#-hash-table이란)
  - [🟤 Linear Probing](#-linear-probing이란)
  - [🟤 구현 코드](#-구현코드)

----

## 🌱 Linear Probing Hash Table

### 🟤 Hash Table이란?

Hash Table이란 Hash라는 함수를 사용하여 변환된 Key값을 Index로 사용하여 Key, Value 형태의 데이터를 저장하는 자료구조이다.

> ❗️ 해싱이란 해시 함수를 사용하여 임의의 값을 특정한 값으로 추출하는 것을 의미한다. 여기서 해싱을 사용한 이유는 key의 다양한 값을 한정된 
> 데이터 저장 공간(배열 등)에 효율적으로 저장하기 위해 사용된다.

---

### 🟤 Linear Probing이란?

위의 Hash Table의 정의대로 해시함수를 사용하여 임의의 key값을 특정 값으로 변환하게 되면 한가지 문제가 발생한다. 바로 해시 충돌(Hash Collision)이다. 
가령, 숫자 Key값에서 현재 저장할 수 있는 데이터공간의 크기로 나머지 연산하는 것을 해시 함수로 설정하면, 해시 함수를 통해 얻게 되는 값은 데이터 공간에 접근할 수 있는 index값 중 하나이다. 
하지만, 위의 해시함수를 사용해서 데이터 공간이 10이며 Key값이 각각 1과 11로 얻을 수 있는 Index는 1로 동일하다. 이 때는 하나의 데이터 공간에 2개의 값을 저장해야하는 상황에 발생하고 이런 현상을 
`해시 충돌`이라고 한다.

이러한 해시 충돌을 해결하기 위한 방법으로 Linear Probing이라는 기법이 사용된다. Linear Probing은 만일 저장하고자 하는 공간에 이미 다른 값이 저장되어 있다면, 다른 빈 공간에 저장하고자 하는 값을 저장하여 
데이터 공간을 리사이징하지 않고 아끼는 방식이다. 데이터 공간을 아낄 수 있다는 장점이 있지만, 데이터를 삭제하는 로직이 생각보다 까다롭고 Input이 많은 상황이라면 쉽게 해당 자료구조에 저장할 공간이 부족해질 수 있다.

---

### 🟤 구현코드

```java
public class LinearProbing<T> {

	private HashItem<T>[] table;

	public LinearProbing(int size) {
		table = new HashItem[size];
	}

	public boolean add(int key, T data) {
		int tempIndex;
		int hashIndex = hashFunction(key);

		if (table[hashIndex] == null || table[hashIndex].isDeleted()) {
			table[hashIndex] = new HashItem<>(key, data);
			tempIndex = -1;
		} else {
			if (hashIndex == (table.length - 1)) {
				tempIndex = 0;
			} else {
				tempIndex = hashIndex + 1;
			}
		}

		while (tempIndex != -1 && tempIndex != hashIndex) {
			if (table[tempIndex] == null || table[tempIndex].isDeleted()) {
				table[tempIndex] = new HashItem<>(key, data);
				tempIndex = -1;
			} else {
				if (tempIndex == (table.length - 1)) {
					tempIndex = 0;
				} else {
					tempIndex++;
				}
			}
		}

		return tempIndex == -1;
	}

	public T getValue(int key) {
		int tempIndex;
		int hashIndex = hashFunction(key);

		if (table[hashIndex] == null || table[hashIndex].isDeleted()) {
			return null;
		} else if (table[hashIndex].getKey() == key) {
			return table[hashIndex].getData();
		} else {
			if (hashIndex == (table.length - 1)) {
				tempIndex = 0;
			} else {
				tempIndex = hashIndex + 1;
			}
		}

		while (tempIndex != -1 && tempIndex != hashIndex) {
			if (table[tempIndex] == null || table[tempIndex].isDeleted()) {
				return null;
			} else if (table[tempIndex].getKey() == key) {
				return table[tempIndex].getData();
			} else {
				if (tempIndex == (table.length - 1)) {
					tempIndex = 0;
				} else {
					tempIndex++;
				}
			}
		}

		return null;
	}

	public void delete(int key) {
		int tempIndex;
		int hashIndex = hashFunction(key);

		if (table[hashIndex] == null) {
			return;
		} else if (table[hashIndex].getKey() == key) {
			table[hashIndex].setDeleted();

			return;
		} else {
			if (hashIndex == (table.length - 1)) {
				tempIndex = 0;
			} else {
				tempIndex = hashIndex + 1;
			}
		}

		while (tempIndex != -1 && tempIndex != hashIndex) {
			if (table[tempIndex] == null) {
				return;
			} else if (table[tempIndex].getKey() == key) {
				table[tempIndex].setDeleted();

				return;
			} else {
				if (tempIndex == (table.length - 1)) {
					tempIndex = 0;
				} else {
					tempIndex++;
				}
			}
		}
	}

	private int hashFunction(int key) {
		return key % table.length;
	}

	private class HashItem<T> {
		int key;
		T data;
		boolean deleted;

		public HashItem(int key, T data) {
			this.key = key;
			this.data = data;
			this.deleted = false;
		}

		public int getKey() {
			return key;
		}

		public T getData() {
			return data;
		}

		public boolean isDeleted() {
			return deleted;
		}

		public void setDeleted() {
			this.deleted = true;
		}
	}
}

```


