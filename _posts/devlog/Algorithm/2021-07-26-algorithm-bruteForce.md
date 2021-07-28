---
layout: post
title: "Algorithm - Brute Force"
subtitle: "완전탐색 브루트 포스"
categories: devlog
tags: algorithm BruteForce 완전탐색
---

> 알고리즘 완전 탐색 브루트 포스의 개념과 원리, 간단한 사용 예제입니다.

<!---more--->

- 목차

## Brute Force란?

---

> - Brute Force의 Brute는 짐승, 난폭한,,이란 뜻과 Force가 만나 난폭한 힘.. 이런 느낌인 자료탐색방법인데 간단히 말하면 "너가 멀 좋아할지 몰라서 일단 다 준비해봤어.."이거다. 🤪
> - Brute Force의 자료구조의 종류로는 2가지(선형구조 - 배열, 연결리스트 등, 비선형구조 - DFS, BFS)가 있다!! <br>(다음엔 자료구조도 공부해보아야겠다!!)
> - 주어진 값을 구하기 위해서 모든 경우를 탐색하기 때문에 시간복잡도는 경우가 늘어날 때마다 기하급수적으로 늘어난다. <br> 일반적으로 컴퓨터는 1초에 평균 1억번 정도의 연산이 가능하기 때문에 1000억번의 연산을 수행한다면 <b>17분</b>이 소요된다./
> - 시간적인 측면에서는 비효율적이지만, 사용하기 쉽고 모든 경우를 탐색하기 때문에 <b>100%</b> 답을 찾는 것이 가능하다.

## 사용 예제 (JavaScript)

---

- 간단하게 소수를 찾는 문제로 Brute Force를 알아보자! <br> (자료구조는 배열을 사용하여 진행!)

> ```
> const question = arr => {
>    let answer = []
>
>    for(let i of arr) {     --- (1)
>        let s = []
>        for(let z = 1; z <= i; z++) {    --- (2)
>            if(i % z === 0) s.push(z);
>        }
>        if(s.length === 2) answer.push(s[s.length - 1])
>    }
>    return answer
> }
>
> question([3, 2, 4, 6, 9, 11, 129, 127])
> ```

- 효율적인 코드인지는 잘 모르겠지만.. 간단하게 살펴보면!
-
