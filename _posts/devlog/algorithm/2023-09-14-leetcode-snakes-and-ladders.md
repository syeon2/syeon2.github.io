---
layout: post
title: "[LeetCode] 909. Snakes and Ladders"
subtitle: "algorithm"
categories: devlog
tags: algorithm graph
---

> LeetCode Top Interview 150의 909번 문제입니다.

<!--more-->

📚 목차
- [🌱 Snakes and Ladders](#-snakes-and-ladders)
  - [🟤 문제 정의 - Definition](#-문제-정의-definition)
  - [🟤 문제 풀이 전략 추상화 - Abstraction & 문제 풀이 (Algorithm)](#-문제-풀이-전략-추상화-abstraction--문제-풀이-algorithm)

----

## 🌱 [Snakes and Ladders](https://leetcode.com/problems/snakes-and-ladders/?envType=study-plan-v2&envId=top-interview-150)

### 🟤 문제 정의 (Definition)

- 파라미터로 int형의 2차원 배열(board)가 주어진다.

- 문제를 간략하게 요약하자면 이렇다. 주어진 2차원 배열을 게임 보드판이라고 가정한다. 이 보드판은 (board.length - 1)<sup>2</sup>의 크기를 가지고 
총 (board.length - 1)<sup>2</sup> 만큼의 발판(위치값)을 가지고 있다.
- x = 0, y = board.length - 1에서부터 위치값 1로 시작하여 왼쪽방향으로 이동한다. 위치값도 1씩 증가한다.
- x가 board.length - 1에 다다르면 y축으로 -1하여 이동하고, 다시 x가 0이 되는 방향으로 (오른쪽으로) 위치값이 1씩 증가하면서 이동한다.
- 위의 2가지 이동방식을 가지고 발판(위치값)이 (board.length - 1)<sup>2</sup> 값인 마지막 발판까지 도달하기 위한 최소 이동 수를 구하는 문제이다.

- 이동에 필요한 조건으로는 주사위를 굴려 주사위에서 나온 수만큼 이동할 수 있다. 즉, 1 ~ 6만큼 이동할 수 있다.
- 무조건 6만큼 움직이는 것이 좋은 것은 아니다. 왜냐하면 보드판 중간 중간에 사다리와 뱀이라는 구조물이 있는데, 이 구조물은 2개의 발판이 연결되었으며, 
하나의 발판에 위치하면 연결된 다른 발판으로 이동할 수 있다.
- 이 구조물은 주사위를 굴려 한번 이동할 때마다 한번만 사용 가능하다. 예를 들어 1-6, 6-10 이렇게 사다리가 존재한다면 1에서 바로 10으로 갈 수 있는 것이 아니라 
6으로 한번 이동하고 그 다음 10으로 이동 가능하다는 것이다.

- 위와 같은 조건으로 문제를 풀어야 한다.


- 조건
  - n == board.length == board[i].length
  - 2 <= n <= 20
  - board[i][j] is either -1 or in the range [1, n2].
  - The squares labeled 1 and n2 do not have any ladders or snakes.

---

### 🟤 문제 풀이 전략 추상화 (Abstraction) & 문제 풀이 (Algorithm)

해당 문제는 특이한 형태라고 생각되지만, 전형적인 BFS를 사용한 최단 거리 탐색 문제이다. 
이 문제의 특이점은 이동할 수 있는 거리가 1에서 6으로 랜덤하게 정할 수 있고, 1이라는 첫 시작점에서 n * n이라는 도착점까지 도달할 수 있는 최소 거리를 
구하는 문제이다.

주의해야할 부분이라고 생각하는 포인트는 위의 관점으로 문제를 풀어야하는 것이다. 2차원 배열이 주어졌다고 해서 2차원 배열을 캐시 메모리로 사용하는 것보다 
1부터 N * N까지의 거리를 1차원 배열로 캐싱 메모리를 구하는 것이 쉽고 효율적이다.

코드를 보면서 이해해보자.


```java
class Solution {
  public int snakesAndLadders(int[][] board) {
    // n은 전체 보드판 중 하나의 row에 몇칸 있는지를 구한 값이다.
    int n = board.length;

    // BFS 탐색을 위한 Queue 자료구조이다. Queue 자료구조는 한번 위치한 발판에서 1-6까지의 주사위를 통해 이동할 수 있는 다음 발판을 미리 저장하여 
    // 수평 탐색을 가능하도록 한다.
    Queue<Integer> queue = new LinkedList<>();
	
	// 첫 시작점인 1을 queue에 넣는다.
    queue.add(1);

	// 각 방문했던 발판을 캐싱하는 boolean 타입 배열 자료구조이다.
    boolean[] isVisit = new boolean[n * n + 1];

	// 한번 이동할 때마다 1씩 증가하는 move 변수이다.
    int move = 0;
	
	// queue에 저장된 값이 없어질 때까지 무한히 동작한다.
    while (!queue.isEmpty()) {
		
      // 현재 queue 사이즈를 한번 i에 할당하여 i가 0이 될때까지 다음 동작을 수행한다.
      // 현재 queue 사이즈는 이전에 위치했던 발판에서 주사위를 굴려 다음에 위치할 수 있는 발판(위치값)을 저장한 개수이다.
      // 즉, 이 시점에서 queue가 담고 있는 발판값은 move + 1만큼의 이동으로 위치할 수 있는 것이다.
      for (int i = queue.size(); i > 0; i--) {
		  
        // 현재 위치를 queue에서 가져온다.
        // 현재 위치가 이미 방문했던 위치라면 이미 해당 위치에서 1-6까지 이동하는 수들이 queue에 저장되었기 때문에 continue를 사용하여 다음 값으로 넘긴다.
        // 방문하지 않은 발판이라면 true로 해당 위치를 캐싱하여 다음에 다른 루트로 해당 발판을 방문했을 때 같은 반복을 추가로 하는 일이 없도록 한다.
        int cur = queue.remove();

        if (isVisit[cur]) continue;

        isVisit[cur] = true;

		// 이 부분에서 문제에서 요구하는 값을 반환한다. 각 발판에 위치할 때마다 현재 위치값에서 1-6만큼의 값을 다시 queue에 넣는다.
        // 이런 동작을 반복하다보면 도착지점 값이 queue에 저장되는 순간이 존재한다. 이 때, return을 하게 되면 현재까지 최소로 더한 move한 값을 
        // 구할 수 있다.
        if (cur == n * n) return move;

		// 현재 위치에서 1-6만큼 주사위를 굴려 queue에 저장하는 코드이다.
        // 이 코드에서는 총 2가지의 작업을 수행한다.
        // 1. 다음으로 갈 위치값을 구하여 방문하지 않은 발판값만 queue에 저장한다.
        // 2. 해당 발판이 사다리로 이어져있는 발판인지 확인해야하는 동작이 필요하다. 주어진 2차원 배열은 기본적으로 각 요소들이 -1값을 가지지만 
        // 사다리나 뱀으로 이어져있는 값들은 이어져있는 발판으로 저장되어 있다.
        
        // 이동하는 경로는 숫자 상으로는 1부터 n * n까지지만, 해당 보드판은 각 숫자가 왼쪽에서 오른쪽으로 출발하여 위를 향해 지그제그 형태도 되어 있다.
        // 이 특징을 가지고 다음 이동하는 경로를 2차원 배열에서 구하여 사다리, 뱀의 유무를 확인하는 함수(getPosition)를 구현한다.
        for (int k = 1; k <= 6 && cur + k <= n * n; k++) {
          int next = cur + k;
          int value = getPosition(board, next);

          if (value > 0) next = value;
          if (!isVisit[next]) queue.add(next);
        }
      }

	  // 한번 스냅샷?한 queue size만큼 동작을 반복하면 그것이 한번 이동할 때 발생하는 동작이기 때문에 for 문이 완료되면 move를 1증가시켜 
      // 다음 이동할 때의 값들을 비교할 수 있도록 한다.
      move++;
    }

    return -1;
  }

  // 보드는 (1 - n), (n + n - n), (n + n + 1, n + n + n)... 이런 지그재그 방식으로 이동할 수 있기 때문에 이러한 특징을 고려하여 
  // 다음에 이동할 발판에 사다리나 뱀이 있는지 없는지 탐색한다. 사다리나 뱀이 있다면 연결되어 있는 다음 발판 값을 반환하고 없다면 -1을 반환하여 
  // 본래 주사위로 이동하는 코드에서 다음에 이동할 수 있는 경로를 queue에 저장하도록 한다.
  private int getPosition(int[][] board, int num) {
    int n = board.length;

    int r = (num - 1) / n;
    int x = n - 1 - r;
    int y = r % 2 == 0 ? num - 1 - r * n : n + r * n - num;

    return board[x][y];
  }
}
```

