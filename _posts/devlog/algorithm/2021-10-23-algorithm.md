---
layout: post
title: "[Algorithm] 문자 숫자열과 영단어"
subtitle: "카카오 코딩 테스트 문제"
categories: devlog
tags: algorithm 재귀함수 split
---

> 카카오 코딩 테스트 문제 '문자 숫자열과 영단어'입니다.

<!---more--->

## 📌 문제

---

> 네오와 프로도가 숫자놀이를 하고 있습니다. 네오가 프로도에게 숫자를 건넬 때 일부 자릿수를 영단어로 바꾼 카드를 건네주면 프로도는 원래 숫자를 찾는 게임입니다.
> 다음은 숫자의 일부 자릿수를 영단어로 바꾸는 예시입니다.
>
> 1478 → "one4seveneight"
> 10203 → "1zerotwozero3"
> 234567 → "23four5six7"
>
> 이렇게 숫자의 일부 자릿수가 영단어로 바뀌어졌거나, 혹은 바뀌지 않고 그대로인 문자열 s가 매개변수로 주어집니다. s가 의미하는 원래 숫자를 return 하도록 solution 함수를 완성해주세요.

## 📌 나의 풀이

---

```
function solution(s) {
    let answer = s;
    let numObj = {
        zero: 0,
        one: 1,
        two: 2,
        three: 3,
        four: 4,
        five: 5,
        six: 6,
        seven: 7,
        eight: 8,
        nine: 9,
    };

    function DFS() {
        for(let i in numObj) {
            let answer2 = answer;
            answer2 = answer2.replace(i, numObj[i]);

            if(answer2 !== answer) {
                answer = answer2;
                DFS();
            }
        }
    }

    DFS();

    return parseInt(answer);
}
```

- 처음에는 `replaceAll`을 사용하여 문제를 풀 마음이었지만 프로그래머스는 replaceAll을 지원하지 않는다고...ㅠ
- 어떻게 할까 고민하다가 replace를 상황에 따라 계속 해주면 되는거 아닌가? 싶은 생각이 들었다.
- 상황이라함은 중복되는 숫자가 한 문자열에 존재할 경우였다.
- 기존 수를 변수로 할당하고 replace하여 기존 값과 replace하여 변하는 값이 있다면 다시 검사하는 방식으로 로직을 구현했다.
- 어떤 방법으로 다시 검사할까 고민하다가 최근에 배운 DFS (재귀함수)를 사용하여 구현해보았다.

=> 이렇게 하니 모든 테스트케이스에는 통과했지만, 속도는 한 테스트케이스당 0.2s ~ 0.3s정도 나오는 것 같았다.

---

## 📌 다른 풀이

---

```
function solution(s) {
  let num = ['zero', 'one', 'two', 'three', 'four', 'five', 'six', 'seven', 'eight', 'nine'];

  let answer = s;

  for(let i = 0; i < num.length; i++) {
    let arr = answer.split(num[i]);
    answer = arr.join(i);
  }

  return parseInt(answer);
}
```

- 풀이 중에서 제일 좋아요? 수를 많이 받는 코드이다.
- 잘 살펴보면 num 배열을 for문으로 돌리는 것까지는 내 코드와 비슷하지만, 필자는 DFS를 사용함으로서 이중 삼중으로 반복 로직을 실행시켰다.
- 하지만 이 로직에서는 for하나로 split를 사용함으로서 전체 string을 for가 돌때마다 한번씩만 호출하면 된다.
- 역시나 테스트 코드는 모두 통과임과 함께 속도도 하나의 테스트 코드당 0.06s ~ 0.07s가 걸린다.
- 시간복잡도 측면에서 훨씬 좋은 코드인 것이다..!!ㅠㅠ

## 📌 마치며

---

- 역시나 알고리즘은 재밌다!!!
- 여러가지 알고리즘 자료구조를 활용해 최적의 코드를 구현하는 방법! 더 연구하고 많이 사용해보아야겠다.
- 하지만 나도 과거에 쩔쩔매던 재귀함수를 활용해서 스스로 코드를 구현한 것에 보람을 느낀다 :)
