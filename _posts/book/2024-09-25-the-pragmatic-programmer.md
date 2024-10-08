---
layout: post
title: "[Book] The Pragmatic Programmer (실용주의 프로그래머)"
subtitle: "실용주의 프로그래머 서적의 간단 요약 및 저의 생각을 정리한 글입니다."
categories: devlog
tags: book
---

> 실용주의 프로그래머 서적의 간단 요약 및 저의 생각을 정리한 글입니다.

<!--more-->

#### 목차

- [여는 글](#-여는-글)
  - [Chapter 1. 실용주의 철학](#-chapter-1-실용주의-철학)
    - [Topic 1 : 당신의 인생이다.](#1-topic-1--당신의-인생이다)
    - [Topic 2 : 고양이가 내 소스 코드를 삼켰어요.](#2-topic-2--고양이가-내-소스-코드를-삼켰어요)
    - [Topic 3 : 소프트웨어 엔트로피](#3-topic-3--소프트웨어-엔트로피)


## 🌱 여는 글

필자가 생각하는 '좋은 개발자'는 <strong>생각</strong>하는 개발자입니다.

단순하게 기술을 사용하는 것이 아닌, 기술을 선택하고 사용하는데에는 논리적인 판단을 할 수 있는 자신만의 주관을 가지는 것이 필요하다고 느낍니다. 커뮤니케이션 등의 소프트 스킬도 마찬가지입니다.

<strong>실용주의 프로그래머</strong>에서 등장하는 여러 주제들을 읽고, '개발'에 대한 저의 생각을 확장하고 다지는 기회가 될거라 기대하며 이 글을 작성합니다.

---

## 🌱 요점 정리

### 🥕 Chapter 1. 실용주의 철학

#### 1. Topic 1 : 당신의 인생이다.

<strong>행동</strong>하라는 주제입니다.

"자신의 현 상황에서 변화를 피하지 않고, 무엇이든지 적극적으로 개선하라." 라고 말합니다.

> 변화는 '좋은 변화', '나쁜 변화'가 있습니다. 사람들이 '나'로 인한 변화가 '좋고/나쁨'인지를 판단하는 기준은 무엇이 될지 고려해야합니다. 
> 
> 같은 변화라 할지라도 환경, 사람 등으로 인해 좋고/나쁨이 결정되는 상황도 발생합니다. 그렇기 때문에 변화를 꾀하는 사람은 주변 상황을 인식하고 그에 맞는 타당한 근거를 제시해야합니다.
> 
> 변화는 단순한 실행이 아니라, 주변 상황과 사람들의 기대를 고려한 신중한 계획이 필요합니다. 또한, 지속적인 피드백을 통해 변화를 조정하고, 예상치 못한 반응에도 유연하게 대응하는 것이 중요합니다. 변화의 과정은 끊임없이 조정하고 적응하는 과정임을 인식하고, 이를 통해 개인적인 성장과 주변과의 조화를 이끌어내는 것이 핵심입니다.

#### 2. Topic 2 : 고양이가 내 소스 코드를 삼켰어요.

`약점을 보이는 것에 대한 두려움이 가장 큰 약점이다.`

자신의 모든 행동에 대해 <strong>책임을 지는 태도</strong>가 중요하다고 말합니다.

> 결과에 따른 책임을 지는 태도는 그 결과를 이끌어낸 리더에게 꼭 필요하다고 생각합니다.
> 
> 따라서, 리더는 결과를 도출하기 위해 크게 2가지를 고려해야합니다.
> 
> 첫번쨰는 결과를 만들기 위한 다양한 방안을 모색하는 것입니다. 여러가지 방안을 탐색하고 각 방안마다의 트레이드오프를 고려하여 최적의 선택을 하는 것입니다.
> 
> 두번쨰는 결과를 이끌어 내지 못했을 경우를 대비하여 적절한 대안을 모색하는 것입니다.
> 
> 고양이가 소스 코드를 먹었다고 핑계대는 것이 아니라 논리적이고 타당한 방안들을 찾아 팀원들과 공유하고 설득하는 과정이 중요한 덕목인 것 같습니다.

### 3. Topic 3 : 소프트웨어 엔트로피

`소프트웨어 엔트로피 : 기술 부채`

소프트웨어에서 무질서가 발생하는 주요 원인은 ‘형편없는 코드’, ‘잘못된 결정’, ‘나쁜 설계’ 등을 꼽을 수 있습니다. 이러한 문제들은 흔히 <strong>기술 부채</strong>라는 개념으로 표현되며, 시간이 지남에 따라 소프트웨어를 점점 더 망가뜨리게 됩니다. 
결국에는 소프트웨어가 손을 대기 힘들 정도로 악화될 수 있습니다. 이는 마치 ‘깨진 유리창 이론’과 같은 현상입니다.

이러한 상황을 방지하기 위해서는 소프트웨어가 꾸준히 관리되고 있음을 명확히 해야 합니다. 급하게 작성된 코드는 주석을 통해 설명을 덧붙이고, 설계에는 지속적인 피드백을 받아 개선하는 노력이 필요합니다.
이렇게 해야만 소프트웨어가 ‘깨진 유리창’처럼 방치되고 있다는 인상을 주지 않을 수 있습니다.

> 종종 면접에서 등장하는 질문이 있습니다. “마감 일정이 촉박한 상황에서 코드 완성도를 높이는 것과 빠르게 개발하는 것 중 어느 것을 우선시해야 할까?”
> 
> 이 질문에 대한 키포인트는 '고객 중심을'이라고 생각합니다. 고객 입장에서는 '약속된 시간'과 '필요한 기능 및 정확도'가 약속되어 있습니다. 따라서 
> 기한 내에 빠르게 개발하되 고객이 가장 필요로 하는 기능을 우선순위에 맞춰 제공하는 것이 중요합니다.
> 
> 마감 이후에는 내부 회의를 통해 왜 특정 수준의 정확도를 달성했는지에 대한 피드백을 나누고, 이러한 상황이 재발하지 않도록 개선점을 도출하는 것이 필요합니다. 이를 통해 향후에는 더 나은 품질과 일정 관리를 할 수 있을 것입니다.