---
layout: post
title: "Werecord project -v2👨🏻‍💻"
subtitle: "Wecode 멘티를 포커싱한 근태 프로젝트 -v2"
categories: project
tags: project werecord React 근태
image:
  path: /assets/img/project/2021-08-17-werecord-v2/cover.jpg
---

> Werecord -v2 project 후기 및 피드백입니다.

<!--more-->

- 목차
  - [We Record -v2?](#-we-record--v2)
  - [각 페이지 소개](#-각-페이지-소개)
    - [Initail Setting](#-initail-setting)
    - [Global Component](#-global-component)
    - [Rending Page](#-rending-page)
    - [Main Page](#-main-page)
      - [1. 실시간 타이머 기능](#1-실시간-타이머-기능)
      - [2. 스크린 캡쳐 기능](#2-스크린-캡쳐-기능)
    - [My Page](#-my-page)
    - [Mate Page](#-mate-page)
    - [Responsive Web(반응형 웹)](#-responsive-web-반응형웹)
    - [Error & Fix](#-error--fix)
  - [마치며...](#-마치며)

## 📌 We Record -v2?!?!

---

[💁🏻‍♂️ 버전1 - WeRecord 후기](https://ksy4568.github.io/project/project-review-werecord.html) <br />
[💁🏻‍♂️ 버전1 - WeRecord 영상](https://youtu.be/TtY8FGlqDuw)

팀원들과 함께 한 We Record project가 마무리 된지 어느덧 한달이 지나고 있는 것 같다..(시간이 진짜 빠르다..🚗) 함께했던 팀원들도 각자도생..?의 느낌으로 흩어져 자신만의 스타일, 방식대로 취업을 준비하는 시간을 보내게 될 것 같았다.
<br />

빠듯한 취업 준비로 We Record의 프로젝트는 우리의 코딩 성장의 양분이 되어 기억 속에 잊혀질 것만 같았지만,, 사람 일은 모르는 것!!⭐️ 그저 코딩이 재미있고, 배우고 싶은 기술 스택도 많았고, 무엇보다 서비스를 목적으로 처음 작업했던 프로젝트를 더욱 완성도 있는 서비스를 만드는 것에 마음이 갔다.
이런 나를 팀원분들과 멘토님께서 배려해주셔서 혼자 We Record -v2를 작업할 기회가 생기게 되었다!! 🔥 (두둔)<br />

약 2주 정도가 지난 지금 프로젝트는 생각보다 재미있었고 다른 동기들이 짰던 코드들을 보면서 어떻게 구현해야하는지 몰랐던 로직도 알게 되고, 내가 짰던 코드와 동기 코드를 비교하면서 어떤게 더 효율적일까 고민도 하면서 재밌고 알찬 시간을 보낸 것 같아 기분이 좋았다.😊
<br />

이제는 개인 프로젝트로 바뀌었지만 컨벤션과 trello를 활용하여 할일을 체크하고 기록을 남겨두어 스스로 피드백을 하고 싶었다. 컨벤션은 최대한 이전 프로젝트를 따라가고 trello를 활용하여 blocker와 백앤드분께 추가적으로 요청해야될 부분들을 티켓으로 만들어 보관했다.

<img alt="trello" src="/assets/img/project/2021-08-17-werecord-v2/trelloImg.png" width="600px">

## 📌 각 페이지 소개

---

### 🌱 Initail Setting

- 개인 프로젝트로 바뀌면서 제일 먼저 손봤던 것은 초기세팅이었다. 반응형을 시작으로 폰트, 색상들이 필요할 때마다 master가 merge되다가 지쳐 결국 각자 컴포넌트에서 일일히 따로 선언했던 기억이 떠올랐기 때문이다.
- 1차 프로젝트 당시에 사용했던 초기세팅의 기억을 되살려 폰트 색상, 반응형 breakpoint 등등을 다시 작업했다.
- 초기 세팅에서 제일 신경썼던 기능은 background였다!! 페이지 뒤에 보이는 산같은 실루엣들이 옆으로 계속 움직이는 것과 시간에 따라 색상이 변하는 기능이었다.(와우!!)

<img alt="morning" src="/assets/img/project/2021-08-17-werecord-v2/morning.png" width="250px"> 아침
<img alt="afternoon" src="/assets/img/project/2021-08-17-werecord-v2/afternoon.png" width="250px"> 점심
<img alt="evening" src="/assets/img/project/2021-08-17-werecord-v2/evening.png" width="250px"> 저녁
<img alt="dawn" src="/assets/img/project/2021-08-17-werecord-v2/dawn.png" width="250px"> 새벽

- 어떻게 구현할까 고민하다가 global css에 dayjs를 import시키고 시간 값에 따라서 background를 지정해주어 간단하게 끝냈다! (styled component 짱이다!! 👍🏼)

- 뒤에 동산같이 생긴 곡선이 옆으로 움직이는 로직은 동산 이미지에 animation을 주어 옆으로 무한히 움직이도록 코드를 짰는데.. 생각보다 과부하가 걸리는 듯한 느낌이라 추후에 request animation frame으로 구현해볼까 한다.

---

### 🌱 Global Component

- GNB와 footer도 조금 다시 손봐야했다. GNB와 footer의 양 옆 margin이 page마다 일정해야했기 때문이다.
- 또한 기존에 해당 페이지에 있으면 그 버튼이 안보이게 되었던 것과 다르게, 바뀐 GNB는 각 페이지에 있을 때 폰트 색상이 흰색으로 바뀌어야했다.
- location.pathname을 사용해 조건부 랜더링으로 되어있던 코드를 styled component에 넘겨주어 유동적으로 바뀌도록 코드를 짰다.

<img alt="GNB" src="/assets/img/project/2021-08-17-werecord-v2/GNB.png" width="250px" title="desktop"> desktop
<img alt="GNB" src="/assets/img/project/2021-08-17-werecord-v2/GNB2.png" width="250px"> mobile

- 기존의 modal도 다시 작업했다! 이제는 start를 눌렀을 때 또한 모달창이 떠야했고 모달 크기는 일정해야하지만 내부 컨텐츠 크기가 제각각이어서 내부에 padding을 주기보단 width와 height를 직접 설정해주는 방법으로 변경했다.

---

### 🌱 Rending Page

- 수정하는데 조금 복잡했던 랜딩페이지..! 소셜 로그인을 구현안해본 나로서는 멀 건드리면 안되고 되는지.. 하나하나 손볼 때마다 조심스럽게 다뤘던 것 같다.
- 포인트는 -v1에서는 SignIn이 모달을 통해 이루어졌는데 -v2에서는 한 페이지 내에서 구현을 해야했다..!!
- 하나로 합치는 것까지는 어렵지 않았지만 처음보는 소셜로그인 코드를 보고 JWT를 방식에 대해 공부하면서 작업해서 시간이 조금 소요되었다.(나중에 JWT 포스팅해야지~)

<img alt="rending" src="/assets/img/project/2021-08-17-werecord-v2/rending.gif"  width="500px">

---

### 🌱 Main Page

- 드디에 시간이 제일 많이 걸린 Main Page다..
- 딱 봐도 새로운 기능이 추가되었는데 바로 실시간 타이머기능, 스크린샷 기능이다.

#### 1. 실시간 타이머 기능

<img alt="timer" src="/assets/img/project/2021-08-17-werecord-v2/timer.gif" width="500px">

- 기존에 없던 기능이라 고민을 많이 했다. 마이 페이지에도 실시간으로 근태 시간이 반영되는 기능이 있지만 그것은 시간만 따로 빼내어 랜더링 시킨 것이라면, 이 기능은 flipClock기능도 추가시켜야 했다.
- npm을 구석구석 찾다가 페이지 컨셉과 잘 맞을 것 같은 라이브러리를 찾았다. react-simple-flipClock이 바로 그것이지만 문제점이 하나 있었는데... 시간이 0으로 가는 타이머 기능만 있었고 정교한 스타일링을 지원을 하지 않았고,, pause를 눌러 멈추고 다시 가게하는 기능이 없었던 것.. 너무나 pure한 타이머였다. 👶🏻

- 결국 라이브러리를 뜯어 커스텀하기 시작했는데.. 생각보다 재미있네..?? 🤭

---

- 라이브러리의 동작 방식을 살펴보니 초를 넘겨주면 내부에서 알아서 1초마다 transform이 일어나도록 코드가 짜여져 있었다. 그리고 0에 도달하면 타이머가 멈추는 방식이었다.

- 이것을 이용하여 벡앤드분께 지금까지 진행한 total time과 last stop time을 요청하여 코드를 짰다.
  > start를 눌렀을 때는 0부터 serinterval로 1씩 증가
  > restart나 pause를 눌렀을 때는 백엔드로부터 total time을 초로 받아서 로직에 적용
  > start를 누른 상태에서 페이지를 나갔다가 다시 들어왔을 때도 계속 변하는 값을 기억하는 방법은 total시간에 (가장 나중에 누른 start시간 - 현재 시간)으로 계산했다.
  > 문제점이 하나 있는데 백엔드 시간 데이터와 프론트 시간 데이터 간의 차이가 간혹 발생한다.. 이로 인해 1~2초 정도 오차가 나는 경우가 종종 발생하는데.. 시간 데이터는 한쪽에서 계산하는 것이 좀 더 안정적으로 보인다..ㅠ

#### 2. 스크린 캡쳐 기능

<img alt="screenCapture" src="/assets/img/project/2021-08-17-werecord-v2/screenCapture.gif" width="500px">

- 이것도 새로운 기능이라 신기하고 재미있었다 :)
- html2canvas 라이브러리를 이용하여 적용했는데 처음에는 desktop영역을 지정하려 했었지만, desktop영역에는 없는 요소들도 추가되거나 desktop에는 있지만 screenCapture에는 보이지 않아야하는 요소들이 있었다.
- 결국 ScreenModal을 새로 만들어 그 modal를 capture하는 식으로 계획을 변경했다! 😃

---

### 🌱 My Page

<img alt="myPage" src="/assets/img/project/2021-08-17-werecord-v2/myPage.png" width="500px">

- 마이페이지는 전체적인 UI를 수정하는 것이 주 작업이었다.
- 앞서 소개한 랜딩 페이지와 마이 페이지를 수정하면서 느낀 것이 있다라면,, 다른 사람 코드도 내 코드처럼 보는 능력을 기르는 것!! 굉장히 중요한 능력인 것 같다고 생각되었다!🥸
- 기존의 highChart의 내부 코드를 수정해야하는 작업, 랜더링 시켜야하는 시간 데이터 코드 수정 작업을 제외하면 빠르게 작업한 페이지였던 것 같다. :)

---

### 🌱 Mate Page

<img alt="matePage" src="/assets/img/project/2021-08-17-werecord-v2/matePage.png" width="500px">

- 기수페이지 또한 빠르게 작업한 페이지였다.
- 전체적인 UI부분을 바꾸는 것이 주 작업이었다.
- 기수페이지는 나 혼자 작업하기에는 한계가 있었고 백엔드분께 추가적인 데이터를 요청드려야했다! (공손공손)

---

### 🌱 Responsive Web (반응형 웹)

<img alt="responsiveWeb" src="/assets/img/project/2021-08-17-werecord-v2/responsive.gif" width="500px">

- 반응형은 최대 1440px, 모바일 1024px로 지정하여 적용하였다.
- styled에 media를 적용하여 해당 breakPoint별로 변화가 있다면 각 component에 선언해주어야 하는 방식으로 코드를 짰다.
- 하지만 이 방법이 가독성이나 효율적인 코드인지는 다시 검토해보고 적용시킬 예정이다.

---

### 🌱 Error & Fix

- 나름 꼼꼼하게 코드를 짰다고 하더라도 초기에는 에러와 함께하는 것 같다...ㅠㅠ(디자인이나 문구도 조금씩 바뀌는 것들도 함께 고려..!) 생각지도 못했던 에러들을 접하면서 코드의 질을 높일 수 있는 고민을 해보는 시간을 가져야겠다!!

> 🔥 멘토님께서 확인해주신 Error & Fix & Design 수정
>
> 1. window.innerheight를 사용하여 화면 크기가 달라도 한 화면에 보여질 수 있도록
> 2. 화면이 커져도 요소들이 창 가운데 있도록
> 3. 요소들 dragEvent시 포인트 색상으로 변경
> 4. Screen Capture Modal 위, 아래에 페이지 이름, 넣기
> 5. 다른 기수 페이지에 접근하기 위해 url을 바꾸면 에러가 나는 버그 수정
> 6. 반응형 모바일일 때 ScreenCapture Modal이 desktop화면과 동일하도록 수정
> 7. 반응형 모바일일 때 기수페이지 기수 등수를 누르면 z-index가 앞으로 나오도록 하는 이벤트 걸기 <br />
>
> (생각보다 버그보다는 디자인이 고칠 부분이 많은 것 같아서 괜히 뿌듯했다 :) )

---

## 📌 마치며..

---

사실 아직 아쉬운 부분들도 남아있다. Redux를 사용해서 전역으로 state를 관리해보고 싶었고, typeScript를 사용해서 트렌드에 맞는 코드를 경험해보고 싶었다. 이런 바램과 달리 오늘 멘토님과 프로젝트 에러픽스를 마치면 바로 배포 진행해서 실제 서비스할 예정이라고 말씀해주셔서 왠지 모르게 신기했다. 실제 코딩을 통해 '개발'을 했다라는 것이 기분이 묘해졌다. 앞으로도 개발과 함께 삶을 살아간다는 것 또한 신기했다.👨🏻‍💻 <br />

우여곡절이 많았던 WeRecord 프로젝트! 실제 서비스할 수 있도록 끝까지 함께했던 시간을 통해 많은 것들을 알아갔다. 마냥 코딩이 재미있어서, 새로운 알고리즘으로 문제를 해결하는 통쾌, 새로운 기술 스택으로 안정성 있는 코드를 구현하는 즐거움... 이런 것들이 전부였던 나에게(당연히 저 요소들도 중요하겠지만 🤗) '내 프로젝트, 내 코드'라는 애착이 생겼다. 더 효율적으로, 다른 사람들이 봐도 가독성있게, 무엇보다 실제 서비스를 받는 클라이언트를 위한 쾌적한 서비스환경을 만들고 싶은 마음이 생겼다. <br />

가치를 만들고자 하는 가치관에서 '어떤 가치를 만들고 싶은지'에 대해 약간 가닥이 잡혀가는 것 같다. 특정 분야가 아니라 어느 분야에서도 실제 서비스를 받는 클라이언트가 만족할 수 있는 '개발'을 하고 싶다. 그런 개발자가 될 수 있도록 앞으로도 화이팅이다!! 💪🏻

> 🎈 [We-Record Github repo](https://github.com/wecode-bootcamp-korea/werecord-frontend)
