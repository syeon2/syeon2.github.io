---
layout: post
title: "[LeetCode] 169. Majority Element"
subtitle: "algorithm"
categories: devlog
tags: algorithm array string
---

> LeetCode Top Interview 150의 169번 문제입니다.

<!--more-->

📚 목차
- [🌱 Majority Element](#-majority-element)
  - [🟤 문제 정의 - Definition](#-문제-정의-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

- [🌱 풀이 개선](#-풀이-개선)

----

## 🌱 [Majority Element](https://leetcode.com/problems/majority-element/description/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 크기가 n인 nums 배열가 파라미터로 주어진다.
- 이 배열 (nums)안에는 여러 value가 존재한다. 이 때, 다수의 element (값 중 제일 많은, 중복되는 값) 를 찾아 반환해야한다.


- ex) [3, 2, 3] => 3
- ex) [2, 2, 1, 1, 1, 2, 2] = => 2


- 조건
  - n == nums.length
  - 1 <= n <= 5 * 10<sup>4</sup>
  - -10<sup>9</sup> <= nums[i] <= 10<sup>9</sup>


- 추가 조건
  - 추가적인 가변 조건 없이 O(1)의 공간복잡도를 사용하여 문제를 풀이하라.

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

방법 1. 추가조건을 생각하지 않고 풀기

사실 처음에 풀 때는 추가조건이 존재하는지 몰랐다. 어떻게 하면 가장 빠르게 풀수 있을지 고민해보았는데 Map 자료구조를 사용하면 
O(2N) => O(N)의 시간복잡도로 빠르게 풀 수 있지 않을까 생각해보았다.

Map 자료구조를 사용하면 간단하게 구현할 수 있다. Map의 key에는 nums의 element 값을, value에는 nums의 배열을 순환하면서 
해당 값이 등장한 카운트 값을 put한다. 순환을 마쳤다면 다시 Map 자료구조를 순환하면서 value가 가장 큰 값을 반환한다.

하지만 이 풀이법은 시간복잡도가 2N이지만 공간복잡도(추가 조건)에도 맞지 않기에 다른 방법이 있을까 고민하게 되었다.

> 🥕 시간복잡도는 nums를 순환하며 Map 자료구조에 key, value를 put하는 O(N)과 다시 Map 자료구조를 순환하며 가장 큰 값을 
> 조회하기 위한 O(N)의 시간복잡도, 총 O(2N)의 시간복잡도를 가진다.
> 
> 🥕 공간복잡도는 Map 자료구조를 사용하고 있기 때문에 O(N)이 된다.

---

방법 2. 추가조건에 만족하도록 풀기

이 문제의 추가 조건은 결국 추가적인 배열을 생성하지 않고 O(1)의 공간복잡도로 풀 수 있는지에 대한 것이다.

nums 배열 안에 값이 중복되어 있는 값들이 인접하도록 정렬하고 각 값을 카운팅하여 제일 많은 카운트 수를 가진 값을 반환하도록 구현해보았다.

- 현재 카운팅된 값이 제일 큰 수를 저장하는 answer 지역변수
- 카운팅된 값 중 최대 카운팅 수를 저장하는 count 지역변수
- 현재 값의 개수를 카운팅하는 tempCount 지역변수

총 3개의 지역변수를 두어 알고리즘을 구현하였다.

nums 배열을 순환하면서 현재값과 이전값이 같다면 tempCount에 1씩 더하고, 다를 때에는 임시 카운트가 카운트보다 클 경우 
count = tempCount하고 tempCount를 1로 초기화, 반환해야할 answer에 nums[i - 1]로 갱신해준다.

nums[i]가 아니라 num[i - 1]로 갱신하는 이유는 nums[i - 1]은 정렬되어 가장 마지막에 중복된(현재 값까지 카운팅한 값)이기 때문이다. 
그렇기 때문에 제일 마지막 영역에 모여있는 값이 가장 클 경우를 대비하여 추가적인 로직을 더해주어야 한다.


> 🥕 시간복잡도는 먼저 nums 를 정렬하는 O(NlogN)과 nums를 순환하는 O(N)이 소요된다. 총 O(N + NlogN)의 시간복잡도가 소요된다.
> 
> 🥕 공간복잡도로는 추가적인 배열 등 가변 공간을 할당하지 않있기 때문에 O(1)의 공간복잡도를 가진다.

---

### 🟤 문제 풀이 (Algorithm)

방법 1.
```java
import java.util.*;

class Solution {
  public int majorityElement(int[] nums) {
    Map<Integer, Integer> map = new HashMap<>();

    for (int i = 0; i < nums.length; i++) {
      map.putIfAbsent(nums[i], 0);

      if (map.get(nums[i]) != null) {
        map.put(nums[i], map.get(nums[i]) + 1);
      }
    }

    int temp = 0;
    int answer = 0;

    for (Integer key : map.keySet()) {
      if (temp < map.get(key)) {
        temp = map.get(key);
        answer = key;
      }
    }

    return answer;
  }
}
```

방법 2.
```java
import java.util.*;

class Solution {
    public int majorityElement(int[] nums) {
        Arrays.sort(nums);

        int answer = nums[0];
        int count = 1;

        int tempCount = 1;

        for (int i = 1; i < nums.length; i++) {
            if (nums[i - 1] == nums[i]) {
                tempCount++;
            } else {
                if (count < tempCount) {
                    count = tempCount;
                    tempCount = 1;
                    answer = nums[i - 1];
                }
            }
        }

        if (count < tempCount) answer = nums[nums.length - 1];

        return answer;
    }
}
```

---

## 🌱 풀이 개선

먼저 2가지의 방법을 소개한다.

1번째 방법은 정렬도 필요하지 않고 오직 O(N)의 복잡도로 풀이되는 방법이다.

```java
class Solution {
    public int majorityElement(int[] nums) {
        int count = 0;
        int majority = 0;

        for (int i = 0; i < nums.length; i++) {
            if (count == 0 && majority != nums[i]) {
                majority = nums[i];
                count = 1;
            } else if (majority == nums[i]) {
                count++;
            } else {
                count--;
            }
        }

        return majority;
    }
}
```

2번째 방법은 너무 간단해서 처음에 보았을 때 당황했다.
```java
import java.util.*;

class Solution {
    public int majorityElement(int[] nums) {
        Arrays.sort(nums);
        return nums[nums.length/2];
    }
}
```

처음에는 잘 이해되지 않았지만 문제를 다시 읽어보았다.

> 최댓값을 가지는 값은 n / 2회 이상 나타난다.

문제에서 주어진 이 부분을 간과했다. (앞으로 문제르 잘 읽어야겠다..!) 이로 인해서 더욱 간결하게 풀수 있는 방법이 2가지 정도 생각나게 되었다.

1번에서 풀이한 접근은 이렇다. 최댓값을 가지는 요소는 항상 모든 값들 중 절반 이상을 가지고 있는 값이다. 그렇기 때문에 최댓값이 되는 값을 제외한 
모든 값의 개수를 합해도 최댓값이 되는 값의 개수가 더 크다는 것이다. 이를 활용하여 count하는 방식이다.

2번째 방식은 정렬을 하는 방식이다. 장렬을 하게 되면 모든 값은 같은 값끼리 인접하게 될 것이다. 이 때, 최댓값의 값은 항상 절반이 넘기 때문에 
배열의 중간 index는 항상 최대값이다. 이를 활용하여 풀이한 방법이다.

각각 첫번째 방법의 시간복잡도는 O(N)으로 더욱 빠르게 처리 가능하며, 두번째 방법은 O(NlogN)의 시간복잡도를 가지게 된다. 공간복잡도로는 둘다 
추가적인 자료구조 생성이 없기 때문에 O(1)의 공간복잡도를 가진다.
