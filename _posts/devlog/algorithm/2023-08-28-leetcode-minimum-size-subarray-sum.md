---
layout: post
title: "[LeetCode] 209. Minimum Size Subarray Sum"
subtitle: "algorithm"
categories: devlog
tags: algorithm array sliding-window
---

> LeetCode Top Interview 150의 209번 문제입니다.

<!--more-->

📚 목차
- [🌱 Minimum Size Subarray Sum](#-minimum-size-subarray-sum)
  - [🟤 문제 정의 - Definition](#-문제-요약-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction](#-문제-풀이-전략-추상화-abstraction)
  - [🟤 문제 풀이 - Algorithm](#-문제-풀이-algorithm)

- [🌱 풀이 개선](#-풀이-개선)

----

## 🌱 [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/description/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 1차원 숫자 배열(nums)과 목표값(target)이 주어진다.

- 배열의 요소 중 인접한 값들의 합이 목표값과 같거나 클 경우 인접한 값들의 개수가 가장 작은 경우를 반환하는 문제이다.
- 만약, 위의 조건에 모든 경우가 만족하지 않는다면 0을 반환해야한다.


- 조건
  - 1 <= target <= 10<sup>9</sup>
  - 1 <= nums.length <= 10<sup>5</sup>
  - 1 <= nums[i] <= 10<sup>4</sup>


- 추가조건
  - Follow up: If you have figured out the O(n) solution, try coding another solution of which the time complexity is O(n log(n)).

---

### 🟤 문제 풀이 전략 추상화 (Abstraction)

이번 문제는 투포인터와 슬라이딩 윈도우를 함께 사용하여 문제를 풀이해보려 한다.

문제의 핵심은 슬라이딩 윈도우의 사이즈를 조절하면서 조금씩 index를 오른쪽으로 옮겨가는 것이다. 이것이 무슨 의미인지 조금 더 자세히 설명해보려 한다.


문제에서 요구하는 것은 인접한 요소들의 합이 목표값과 같거나 클 경우들을 모두 비교하여 가장 작은 요소를 가지고 있는 경우의 요소 개수를 반환하는 문제이다. 
먼저 포인터 a, b를 2개 두어 윈도우의 크기를 설정한다. 처음에는 두 포인터 모두 0부터 시작한다. 0값을 sum에 저장한 이후 sum과 target을 비교한다. 
비교하여 sum이 target보다 작다면 윈도우의 크기를 늘리기 위해 b를 증가시키고, sum이 target보다 같거나 크다면 a를 순차적으로 증가시켜 윈도우의 크기를 줄임에도 target과의 조건에 만족하는지 확인한다.

이러한 방법을 반복하면 target에 만족하는 최소 윈도우 크기를 구할 수 있다.

> 🥕 시간복잡도는 2개의 포인터를 사용하여 각 값이 포인터의 변화에 따라 어떻게 변하는지를 트래킹하는 코드이다. 따라서 O(2N) => O(N)의 시간복잡도를 가진다.
> 
> 🥕 공간복잡도는 추가적으로 만든 배열 및 가변공간이 없기 때문에 O(1)의 공간복잡도를 가진다.

---

### 🟤 문제 풀이 (Algorithm)

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
		
		int a = 0;
		int b = 0;
		
		int min = Integer.MAX_VALUE;
		int sum = 0;
		
		while (b < nums.length) {
			sum += nums[b];
			
			if (sum >= target) {
				while (sum >= target) {
					min = Math.min(min, b - a + 1);
					sum -= nums[a++];
                }
            }
			
			b++;
        }
		
		if (min == Integer.MAX_VALUE) return 0;
		else return min;
    }
}
```

---

## 🌱 풀이 개선

아무리 생각해봐도 O(N logN)으로 풀 수 있는 방법은 떠오르지가 않았다. 물혼 시간복잡도 측면에서는 O(N)이 항상 빠르겠지만 추가 구현사항으로 제시되어 있어 도전해보아야겠다고 생각했다.

처음에는 divide and conquer 방법을 생각해보았다. 하지만 divide and conquer은 해당 문제를 풀이하기에는 맞지 않은 방법이다. 왜냐하면 pivot으로 하나씩 나누게 되면 나누어지는 index의 
인접한 index끼리 더하여 비교할 수 없기 때문이다..!

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int answer = divideConquer(nums, 0, nums.length - 1, target);

        if (answer == Integer.MAX_VALUE) return 0;
		else return answer;
    }

    public int divideConquer(int[] list, int start, int end, int target) {

		if (start == end) {
			if (list[start] >= target) return 1;
			else return Integer.MAX_VALUE;
		}

		int pivot = (start + end) / 2;

		int temp = 0;
		for (int i = start; i <= end; i++) {
			temp += list[i];
		}

		int windowSize = end - start;
		if (temp < target) windowSize = Integer.MAX_VALUE;

		windowSize = Math.min(windowSize, divideConquer(list, start, pivot, target));
		windowSize = Math.min(windowSize, divideConquer(list, pivot + 1, end, target));

		return windowSize;
	}
}
```


위와 같이 분할정복 알고리즘으로는 해당 문제를 풀 수 없다는 것을 몸소 알게 되어 어떤 방법을 사용해야하나 고민했다. 풀이를 공유한 사람들 코드를 보니 이진 탐색을 사용하여 문제를 풀었다.

```java
class Solution {
    public int minSubArrayLen(int target, int[] nums) {
        int low = 1;
        int high = nums.length + 1;
        boolean flag = false;
        while (low < high){
            int mid = low + (high - low) / 2;
            if(fixsw(mid, nums, target)) {
                flag = true;
                high = mid;
            }
            else
                low = mid + 1;
        }
        if(!flag)
            return 0;
        return low;
    }
    public boolean fixsw(int size,int[] nums,int target) {
        int i = 0;
        int j = 0;
        int sum = 0;
        int max = Integer.MIN_VALUE;
		
        while(j < nums.length) {
            sum += nums[j];
			
            if(j - i + 1 == size) {
                max = Math.max(max, sum);
                sum -= nums[i];
                i++;
            }
			
            j++;
        }
		
        if (max >= target) return true;
        else return false;
    }
}
```

위의 방법의 컨셉을 설명하자면 이렇다. fissw 함수는 주어진 size에 맞추어 투포인터를 사용하여 size에 따른 요소들의 합을 구한다. 구한 합이 목표값보다 작다면 false를,
크다면 true를 반환한다.

만약 해당 윈도우 사이즈에 맞추어 target과 조건에 만족하는 값이 있다면 window를 반절로 줄여 다시 비교해본다. 즉, 윈도우 사이즈에 맞추어 배열의 요소들을 더해 target과 비교하는 시간복잡도로는 O(N)이다. 
하지만, 윈도우 사이즈를 이진 탐색을 통해 해당 사이즈에 만족하는 값이 있는지를 조회하기 때문에 O(logN)의 시간복잡도를 가진다. 모든 윈도우 사이즈에 맞추어 이진탐색을 해야하기 때문에 결국 O(NlogN)의 시간복잡도를 
가지게 되는 것이다.

