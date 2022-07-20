---
layout: post
title: "[Algorithm] 프로그래머스 신고결과받기"
subtitle: "algorithm"
categories: devlog
tags: algorithm
---

> 프로그래머스 신고결과받기(카카오)

<!---more--->

- 목차
  - [문제](#-문제)
  - [나의 풀이](#-나의-풀이)

## 📌 문제

- [문제 링크](#https://school.programmers.co.kr/learn/courses/30/lessons/92334)

## 📌 나의 풀이

```
    public int[] solution(String[] id_list, String[] report, int k) {

        HashMap<String, ArrayList<String>> notice = new HashMap<>();
        HashMap<String, Integer> personState = new HashMap<>();

        for (int i = 0; i < id_list.length; i++) {
            ArrayList<String> init = new ArrayList<>();
            notice.put(id_list[i], init);

            personState.put(id_list[i], 0);
        }


        for(int i = 0; i < report.length; i++) {
            StringTokenizer st = new StringTokenizer(report[i], " ");

            String reporter = st.nextToken();
            String reported = st.nextToken();

            if(notice.get(reporter).contains(reported)) {
                continue;
            }

            personState.put(reported, personState.get(reported) + 1);

            notice.get(reporter).add(reported);
        }

        int[] answer = new int[id_list.length];
        for(int i = 0; i < id_list.length; i++) {
            ArrayList<String> last = notice.get(id_list[i]);

            for (int x = 0; x < last.size(); x++) {
                if  (personState.get(last.get(x)) >= k) {
                    answer[i]++;
                }
            }
        }

        return answer;
    }
```

> 📌 피드백 포인트

- 백준 풀이 방식에 익숙해져있다보니 StringTokenizer를 자주 쓰게 되는 것 같다. (split도 있으니 사용해보기)
- for문 index를 정직하게 모두 사용하는 경우는 for(String i : list) 문법을 사용하여 좀 더 간결하게 만드는 것도 좋을 것 같다.

- 계속 틀렸던 부분이 contains로 해당 값이 있나 없나 검증하는 코드였다.
- 원래 처음에는 report를 sorting하고 한 reporter가 같은 reported 사람을 신고하면 1번만 처리되게하는 코드였는데 sorting으로 데이터를 나열하여 같은 값을 찾는건 위험하다는 것을 알게 된 것 같다. (ex. neo, neos일 경우)

- 다른 사람의 풀이를 보니 User라는 class를 만들고 report관련된 데이터를 class instance에 저장하는 방법을 사용한 것 같은데 신박했다. 새로운 api를 사용하고 method를 사용하는 것도 신기했지만, 객체지향 관점을 알고리즘에 적용하여 풀어내는 부분이 인상깊었다. 도전해봐야겠다!
