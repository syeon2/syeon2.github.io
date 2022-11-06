---
layout: post
title: "2차 프로젝트 회고 (CREAM)"
subtitle: "2차 프로젝트를 돌아보며.."
categories: project
tags: project Kream React 경매
image:
  path: https://wkblog-images.s3.ap-northeast-2.amazonaws.com/wecode-cream/2021-07-24-project-review-cream-Cover.png
---

> Wecode 부트캠프에서 진행한 프로젝트 CREAM에 대한 회고록입니다.

<!--more-->

- 목차
  - [근황](#-근황)
  - [Project CREAM](#-project-cream)
  - [기획하는 단계](#1-기획하는-단계)
  - [프로젝트 설명](#2-프로젝트-설명)
  - [마치며...](#-마치며)

## 📌 근황...

---

2차 프로젝트를 마무리하고 약 한달 반이 지났다. 그동안은 Velog를 이용해왔지만 '나만의 블로그를 만들어서 운영하고 싶다..'라는 단순하고도 어리석은(?) 생각으로 github를 이용한 블로깅을 해야지..해야지.. 했던게 지금까지 이어져 왔다.😂

Wecode 부트캠프를 수료하고 난 이후에 개인적으로 공부하는 기술 스택들을 블로깅하고 싶은 마음에 마음 단단히(?) 먹고 몇일동안 고군분토한 결과... 드디어!! 블로그가 만들어졌다!!🎉
이제 부지런히 밀린 블로깅을 하는 일만 남았다..!

## 📌 Project CREAM

---

> 🚌 기간 : 5월 24일 ~ 6월 04일 <br>
> 👥 Front-end : 3명, Back-end : 2명
> 💻 사용된 기술 스택 : React Hooks, React Router, Styled-Component, KAKAO social login, RESTful API
>
> - 팀원들과 함께 신발 경매 사이트 Kream을 모티브로 하여 개발을 진행 <br>
> - 사람들과의 거래 및 경매라는 플로우를 중심으로 한 개발을 목표로 선정

### 1. 기획하는 단계🚶

---

#### 🌱 서비스 파악하기

CREAM의 특징은 물품 구매자와 판매자 모두 서비스를 이용하는 이용자들이 될 수 있다는 것이다. 그렇기 때문에 서비스를 이용하는 회원들은 물품을 판매하게 되면 물품 값을 받아야하고 구매를 하게 되면 물품 값을 내야했다.

프로젝트이기 때문에 실제 화폐가 아닌 팀원끼리 사용할 수 있는 가상 활페로 대체하고, 대략적인 플로우를 생각해보았다.

Kream은 경매로 물품에 대해 입찰만 할 수 있는 것이 아니라 상품에 대한 즉시 구매도 가능했다.
물품에 대한 채결 거래 내역이 실시간으로 갱신되는 것도 chart를 통해서 확인할 수 있었는데, 경매 입찰 거래가와 즉시 구매가, 즉시 판매가에 대한 연결성을 파악해야했다.

얼마 지나지 않아서 즉시 구매가는 판매 입찰로 나온 물품 중 가장 싼 판매 희망가가 즉시 구매가로 책정이 되고, 즉시 판매가는 구매 입찰에 나온 물품 중 가장 비싼 구매 희망가로 책정된다는 것을 발견하였다.

또한 구매 입찰을 진행할 때, 만약 구매 입찰가가 즉시 구매가보다 높다면 자동으로 즉시 구매가로 넘어가주어야 했고, 마찬가지로 판매 입찰을 진행할 때, 만약 판매 입찰가가 즉시 판매가보다 낮으면 자동으로 즉시 판매가로 넘어가주어야 했다.

이러한 거래 플로우를 각 하나의 물품에 대한 여러가지의 사이즈를 고려하여 데이터를 처리해야했기 때문에 생각보다 복잡했고, 팀원들과 의논하며 구현에 집중할 기능들을 하나 둘씩 정리하였다.

#### 🌱 내가 맡은 기능

- 제품 상세 페이지 (chart.js 라이브러리 사용, 무한 스크롤 구현)
- 팀원들과 함께 사용할 공용 Modal 구현
- URLSearchParams를 사용하여 백엔드에서 받는 데이터 필터링 구현

<br>

### 2. 프로젝트 설명🙈

---

#### 🌱 Modal

- WIKEA 프로젝트 때 팀원이셨던 분께서 공용 모달로 만드셨던 기억을 떠올려 스스로 구현해보았다.
- 가장 큰 모달창만 구현을 하고 모달 내부에 들어갈 데이터 요소들은 하위 컴포넌트로 구현하기로 기획하였다.
- 여기서 등장하는 모달들은 내부에 있는 데이터만 다르고 모두 같은 컴포넌트로 구현하였다.

  ![modal](https://wkblog-images.s3.ap-northeast-2.amazonaws.com/wecode-cream/2021-07-24-cream-modal.gif)
  ![modalCode](https://wkblog-images.s3.ap-northeast-2.amazonaws.com/wecode-cream/2021-07-24-cream-modalCode.png)

- 간단해보이는 modal이다. 모달의 on/off를 상위 컴포넌트의 state로 관리해주고, 내부 데이터들을 children으로 받아서 랜더링시켜준다.

<br>

#### 🌱 Chart & Infinity scroll

- 처음 입문한 라이브러리인 Chart.js이다. Chart.js 자체의 기능은 어렵지 않았지만... 설치하는 부분에서 꼬박 하루가 걸렸다..😂 아무리 `npm install chart.js`를 실행해서 chart를 설치해도... 일부 기능들이 동작하지 않았다..

- 원인은 chart.js의 버전이었다. chart.js를 사용하는 다른 사람들의 소스코드를 계속보다가 문득 package.json은 어떻지??라는 생각에 살펴보게 되었는데, 다른 사람들은 2.9.3버전의 chart.js를 사용하고 있었고, 나는 최신 버전으로 보이는 chart.js를 사용하고 있었다. 혹시나 하는 마음에 chart.js를 다운그래이드하여 다시 설치해본 결과...!! 드디어!! chart.js의 option으로 관리되는 기능들이 동작하였다..!! 그때의 감격.. (그 이후 package.json의 존재 이유를 알게 되었다는....)

![chartGif](https://wkblog-images.s3.ap-northeast-2.amazonaws.com/wecode-cream/2021-07-24-cream-chart.gif)
![chartCode](https://wkblog-images.s3.ap-northeast-2.amazonaws.com/wecode-cream/2021-07-24-cream-chartCode.png)

---

- 무한 스크롤은 보통 백엔드와 소통하여 offset과 limit를 사용한 무한 스크롤을 구현하는 경우가 일반적인 것 같다. 하지만 백앤드와 소통해서 구현하기에는 메인 기능이 아니라 여겼기 때문에 백엔드로부터 오는 result값은 다 받고 이벤트가 발생할 때마다 result의 범위를 slice를 통해 넓혀나가는 방법을 택했다. (랜더링 최적화에는 좋지 않은 것 같아 리팩토링이 필요한 로직이라 생각한다.)

![infinityScroll](https://wkblog-images.s3.ap-northeast-2.amazonaws.com/wecode-cream/2021-07-24-cream-infinity1.png)
![infinityScroll](https://wkblog-images.s3.ap-northeast-2.amazonaws.com/wecode-cream/2021-07-24-cream-infinity2.png)

<br>

#### 🌱 Filter

- 입찰 거래 내역에는 가격별, 사이즈별로 오름차순 내림차순 기능이 구현되어 있다.
- 처음에는 데이터 영역을 다루는 것 자체가 백엔드의 역할이라는 회의 결과로 인해 백앤드분께서 일일히 해당 엔드포인트를 만들어주셨다. 그래서 URLSearchParams를 가지고 각 해당 엔드포인드로 이동하게끔 만들었다.
- 하지만 오히려 이것은 버튼을 누를 때마다 불필요한 통신을 실행하기에 오버 엔지니어링이라 느꼈다. 프론트에서 자체적으로 간단하게 필터링할 수 있는 부분인 것 같아 효율적인 운용을 위해 리팩토링해야할 것 같다.

![filter](https://wkblog-images.s3.ap-northeast-2.amazonaws.com/wecode-cream/2021-07-24-cream-filter.gif)
![filterCode](https://wkblog-images.s3.ap-northeast-2.amazonaws.com/wecode-cream/2021-07-24-cream-filterCode.png)

<br>

## 📌 마치며..

---

- 현재 글을 적고 있는 시점은 이미 1달 반이라는 시간이 지났지만 아직까지 팀원들과 프로젝트를 진행했던 기억은 생생하다. 이번 프로젝트의 제일 Blocker는 chart.js의 버전의 차이로 인한 버그인 것 같다.
- 하루종일 '왜 동작을 안하지?' 라는 생각을 수십번도 넘게 했던... 비록 버전 넘버 바꿔주고 npm install하니까 금방 해결될 문제였지만, 현업에서 이런 Blocker를 만나면 어떻게 효율적으로 대처할 것인가에 대해 많이 생각해보았던 것 같다.
- 이 프로젝트를 진행하면서 로직을 통한 문제 해결능력도 중요하지만, 필요한 시점에 적절한 문구를 사용하여 서치하는 능력 또한 중요하다는 것을 느끼는 프로젝트인 것 같다!! :)
