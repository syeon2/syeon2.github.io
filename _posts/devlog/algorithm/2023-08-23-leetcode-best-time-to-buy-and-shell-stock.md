---
layout: post
title: "[LeetCode] 121. Best Time to Buy and Sell Stock"
subtitle: "algorithm"
categories: devlog
tags: algorithm array string
---

> LeetCode Top Interview 150의 121번 문제입니다.

<!--more-->

📚 목차
- [🌱 Best Time to Buy and Sell Stock](#-best-time-to-buy-and-sell-stock)
  - [🟤 문제 정의 - Definition](#-문제-정의-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

----

## 🌱 [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)

### 🟤 문제 정의 (Definition)

- prices라는 배열이 파라미터로 주어진다.
- prices 배열에는 각 index마다 그날에 지정된 주식 가격이 값으로 저장되어 있다.
- => index 0에는 0번째날 주식 가격 x, index 1에는 1번째날 주식 가격 y ... 이런 방식으로!


- 특정 날을 선택해서 한번 주식을 매입하고 그 이후의 날에 주식을 매매하여 이익을 극대화한 값을 반환한다.
- 만약, 이익이 발생하지 않으면 0을 대신 반환해야한다.


- 조건
  - 1 <= prices.length <= 10<sup>5</sup>
  - 0 <= prices[i] <= 10<sup>4</sup>

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

이 문제에서 주의해야할 점은 주식을 산 이후에는 이후의 날짜에만 매매할 수 있다는 점이다.

최소의 가격으로 매입하여 최대의 가격에 매수하는 것이 이익을 극대화하는 것이기 때문에 이 부분도 함께 고려해야한다.

- prices를 순환하면서 지금까지 순환한 값들 중 최소값을 저장하는 min 지역변수를 선언한다.
- prices를 순환하면서 저장된 최솟값과 현재 값의 차를 저장하는 answer 지역변수를 선언한다.

prices를 순환하면서 min은 계속 최소값이 있다면 갱신해야한다. 만약 현재 값이 min보다 더 큰 값이라면 현재값과 min값을 
빼서 최대 이익 값 (answer) 변수와 비교한다. 이 과정을 반복하면서 최대 이익을 구하도록 한다.

> 🥕 시간복잡도는 prices의 배열을 순환하기 때문에 O(N)이 소요된다.
> 
> 🥕 공간복잡도로는 추가적인 배열이나 가변 공간이 없기 때문에 O(1)의 공간복잡도를 가진다.

---

### 🟤 문제 풀이 (Algorithm)

```java
class Solution {
    public int maxProfit(int[] prices) {

        int min = prices[0];

        int answer = 0;

        for (int i = 0; i < prices.length; i++) {
            min = Math.min(min, prices[i]);

            answer = Math.max(answer, prices[i] - min);
        }

        return answer;
    }
}
```
---
