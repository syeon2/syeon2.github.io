---
layout: post
title: "[Algorithm] 프로그래머스 Level 1 - 모의고사"
subtitle: "모의고사"
categories: devlog
tags: algorithm filter
---

> 프로그래머스 Level 1 모의고사 문제입니다.

<!---more--->

- 목차
  - [문제](#-문제)
  - [나의 풀이](#-나의-풀이)
  - [다른 사람 코드를 참고한 리펙터링](#-다른-사람-코드를-참고한-리펙터링)

## 📌 문제

> 수포자는 수학을 포기한 사람의 준말입니다. 수포자 삼인방은 모의고사에 수학 문제를 전부 찍으려 합니다. 수포자는 1번 문제부터 마지막 문제까지 다음과 같이 찍습니다.
>
> 1번 수포자가 찍는 방식: 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, ...
> 2번 수포자가 찍는 방식: 2, 1, 2, 3, 2, 4, 2, 5, 2, 1, 2, 3, 2, 4, 2, 5, ...
> 3번 수포자가 찍는 방식: 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, ...
>
> 1번 문제부터 마지막 문제까지의 정답이 순서대로 들은 배열 answers가 주어졌을 때, 가장 많은 문제를 맞힌 사람이 누구인지 배열에 담아 return 하도록 solution 함수를 작성해주세요.
>
> 제한 조건
> 시험은 최대 10,000 문제로 구성되어있습니다.
> 문제의 정답은 1, 2, 3, 4, 5중 하나입니다.
> 가장 높은 점수를 받은 사람이 여럿일 경우, return하는 값을 오름차순 정렬해주세요.

## 📌 나의 풀이

```
function solution(answers) {
    let correct = {
        1: 0,
        2: 0,
        3: 0
    }

    let one = [1, 2, 3, 4, 5];
    let two = [2, 1, 2, 3, 2, 4, 2, 5];
    let three = [3, 3, 1, 1, 2, 2, 4, 4, 5, 5];

    for(let i = 0; i < answers.length; i++) {
        if(answers[i] === one[i % 5]) {
            correct[1] += 1;
        }
        if(answers[i] === two[i % 8]) {
            correct[2] += 1;
        }
        if(answers[i] === three[i % 10]) {
            correct[3] += 1;
        }
    }

    let answer = Object.entries(correct).sort((a, b) => b[1] - a[1]);
    let return1 = [];

    function DFS() {
      for(let i = 0; i < answer.length; i++) {
        if(i === answer.length - 1) {
            return return1.push(parseInt(answer[i][0]));
        }
        if(answer[i][1] > answer[i + 1][1]) {
          return return1.push(parseInt(answer[i][0]));
        }
        if(answer[i][1] === answer[i + 1][1]) {
          return1.push(parseInt(answer[i][0]));
        }
      }
    }
    DFS();
    return return1.sort((a, b) => a - b);
}
```

- 참.... 봤을 때부터 드럽다...
- 어찌 어찌 풀기는 했지만... 메소드를 어거지로 붙여서 완성시킨 느낌이 강하게 든다...ㅠ
- 이 코드로 의도한 바는 다음과 같다.
- 먼저 사람1, 사람2, 사람3에 대한 답안 패턴을 array에 담는다.
- 문제의 정답이 담겨져 있는 array의 index는 문제의 번호라고 할 수 있을 것 같다. 각 문제의 번호는 각 사람들이 찍은 정답 패턴의 index와 동일할 것이다.
- 사람들이 찍은 정답 패턴은 0 ~ x까지 정해졌기 때문에 문제 번호가 x를 넘어가면 다시 index 0부터 시작된다.
- 이것을 통해 사람들이 찍은 정답 패턴의 나머지를 반복시켜 각 문제 번호의 정답과 비교할 수 있다.
- 정답을 비교하면서 문제의 정답과 사람들의 정답이 일치하면 사람들 정답에 해당되는 correct의 키값을 +1 시켜준다.
- 키값을 비교하기 위해 Object.entries를 사용하여 array로 만들고 DFS라는 함수를 통해 각 사람들의 숫자를 비교해준다.
- 비교한 값들을 return1 배열에 push하고 return한다...

- 참...너무 복잡하다.. 부끄럽지만 DFS라고 해놓고 재귀함수도 안썼다..(쓸 것 같았는데...)

## 📌 다른 사람 코드를 참고한 리펙터링

```
function solution (answers) {
  let answer = [];

  let one = [1, 2, 3, 4, 5];
  let two = [2, 1, 2, 3, 2, 4, 2, 5];
  let three = [3, 3, 1, 1, 2, 2, 4, 4, 5, 5];

  let oneCorrect = answers.filter((a, i) => a === one[i % one.length]).length;
  let twoCorrect = answers.filter((a, i) => a === two[i % two.length]).length;
  let threeCorrect = answers.filter((a, i) => a === three[i % three.length]).length;

  let max = Math.max(oneCorrect, twoCorrect, threeCorrect);

  max === oneCorrect && answer.push(1);
  max === twoCorrect && answer.push(2);
  max === threeCorrect && answer.push(3);

  return answer;
}
```

- 와우... 완전 깔끔하다..!
- 코드가 무엇을 하는건지 명확하게 파악할 수 있어서 더욱 보기 좋은 것 같다.
- 첫번째로 one, two, three를 통해 각 사람들의 정답 패턴을 지정해준다.
- 각 사람들의 정답패턴과 문제 정답을 filter 메소드를 통해서 걸러준다..! (filter로 return된 array의 length는 당연히 각 사람들의 정답을 맞춘 개수일 것..!)
- 각 사람들 중에서 제일 높은 수를 찾아준다..! Math.max가 3개의 수도 가능한 것인지 이제 알았다..!(알았으면 뻘짓안하고 진작 사용했을텐데..)
- 제일 높은 수와 각 사람들의 정답 개수가 일치하면 answer에 push해준다! (순서대로 1, 2, 3선언해주었기 때문에 굳이 sort가 필요없다.)

## 📌 마치며...

- 아직은 알고리즘을 많이 풀어보지 않았으니 다양한 메소드를 활용하는 능력이 부족할 수 있을 것 같다.
- 문제를 어떻게 하면 해결할 수 있는지.. 효율적으로, 가독성 있게.. 작성하는 코드는 무작정 머릿속에서 어떻게든 해결하려는 단계를 넘어서 고민하고 생각해봐야하는 문제인 것 같다.
- 더 좋은 코드를 위해 오늘도 열심히!!
