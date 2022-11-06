---
layout: post
title: "[TIL] React study"
subtitle: "react study"
categories: devlog
tags: front_android
---

> 실행 컨텍스트(Excution context)의 개념입니다.

<!--more-->

- 목차
  - [Search Input과 Btn](#-1-search-input과-btn)
  - [memo, useCallback](#-2-memo-usecallback)
  - [컴포넌트 역할분담(MVC)](#-컴포넌트-역할분담mvc)

## 📌 Search Input과 Btn

---

- searchInput과 Btn이 API Call 등 같은 기능을 수행하는 역할을 맡고 있다면 각각 요소들에게 다른 이벤트를 적용하는 것이 아니라 from으로 묶고 onSubmit 이벤트를 걸어 놓는 방법도 있다.

  (before)
  <img src="https://wkblog-images.s3.ap-northeast-2.amazonaws.com/front-android/2022-02-13-2.png" width="500px">

  <br />

  (after)
  <img src="https://wkblog-images.s3.ap-northeast-2.amazonaws.com/front-android/2022-02-13-1.png" width="500px">

## 📌 memo, useCallback

---

- memo가 컴포넌트에 사용되면 해당 컴포넌트가 받고 있는 prop의 값이 변경되었을 경우에만 리랜더링이 된다.

- useCallback은 useEffect와 비슷하게 의존성 배열에 할당된 값이 변할 때만 재생성되어 랜더링이 일어날 때마다 재생성되는 것을 막는다.
- 하지만 useCallback으로 할당한 변수를 prop으로 내리고 한번 더 내리게되면 useCallback의 성격을 잃어버린다.

## 📌 컴포넌트 역할분담(MVC)

---

- MVC 디자인 패턴을 살짝 적용시키면 모델, 뷰, 컨트롤러를 분리시켜 하나의 컴포넌트가 맡는 역할을 잘게 쪼개어 분담시키는 것이 중요하다. Test 코드를 짤 때나 재사용성, 유지보수하기에 효과적인 기술이다.

- API 통신을 하는 여러가지 비즈니스 로직을 Class로 묶어두면 공통으로 사용할 수 있는 헤더나 매개변수를 일일히 할당하지 않고 사용할 수 있다.

<img src="https://wkblog-images.s3.ap-northeast-2.amazonaws.com/front-android/2022-02-13-3.png" width="500px">
