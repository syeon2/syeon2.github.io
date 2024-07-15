---
layout: post
title: "📈 Stalk Project를 소개합니다."
subtitle: "Stalk 프로젝트를 시작하며"
categories: devlog
tags: msa
---

> Stalk 프로젝트를 소개하고, 프로젝트의 이슈들을 모아 정리한 글입니다. 

<!--more-->

- 목차
  - [🌱 개요](#-개요)
  - [🌱 1차 프로젝트 구성안](#-1차-프로젝트-구성안)
    - [🥕 프로젝트 예상 설계](#-프로젝트-예상-설계)
  - [🌱 정리](#-정리)

- [🧑🏻‍💻 Stalk 프로젝트의 다양한 이야기](#-stalk-프로젝트의-다양한-이야기)


## 🌱 개요

> 여러 증권사의 종목 탐색 및 최대 5년간의 개장가, 종가, 저가, 고가 등 세부 정보 제공과 투자자 간 인사이트 공유가 가능한 커뮤니티 프로그램입니다.

Stalk 프로젝트는 MSA 환경을 기반으로 기획된 프로젝트입니다.

기존 Tosstock 프로젝트에서 MSA로 마이그레이션 및 추가 기능을 구현하고자 시작되었습니다. 사실 Tosstock 프로젝트는 User, Post, Newsfeed, Stock 정도의 큰 도메인을 가지고 있어 MSA로 
확장할 사이즈의 프로젝트는 아닙니다.

하지만 Stalk 프로젝트에는 앞으로 기획하고 있는 `각 증권마다 실시간으로 사람들과 정보를 주고 받는 단체 채팅` 같은 굵직한 기능들을 지속적으로 개발할 예정입니다. 개인 프로젝트이기에 트래픽 걱정은 없지만, 
도메인별로 각각 특징과 기술을 잘 보여줄 수 있는 개별적인 프로젝트가 된다는 목적으로 MSA를 적용해보게 되었습니다. 

Stalk 프로젝트의 1차 목표는 다음과 같습니다.

- Main Branch Merge마다 개인 서버에서 바로 실행할 수 있는 CI/CD 파이프라인 구축
- MSA 환경을 구성하여 User, Post, Newsfeed, Stock 도메인 단위로 분리
- 실제 서비스되고 있는 여러 비즈니스 구성을 기반으로 API 제공
- 각종 모니터링, MSA 로깅 등의 툴 사용
- Java -> Kotlin 마이그레이션

가장 큰 목표는 여기서 개발한 모든 지식들을 제가 충분히 고민하고 적용해보는 것입니다. 개발에는 절대적인 기술, `은탄환`이 없다는 것을 기억하고 깊게 생각하면서 프로젝트를 만들어가고 싶습니다. 💪🏻

-------------

## 🌱 1차 프로젝트 구성안

현업에서는 무작정 프로젝트 만들자!! 가 아니라 동료들과 충분한 협의 끝에 작업의 우선순위를 두고 진행해야합니다. 하나의 프로젝트 안에서 고민해야할 것들이 인프라, 환경 등을 함께 고려하면서 우선순위를 두고 
초기 계획 및 구성안을 세워 프로젝트를 시작하고자 합니다. (초기에 세원던 계획이 이후에 완성된 프로젝트의 구성과 얼마나 차이점이 있는지 비교 분석해보고 스스로를 보완하고자 합니다.)

이미 마이그레이션을 위해 만들어진 프로젝트도 있어, 초기에 어떤 세팅과 작업을 우선할지가 고민이었습니다. 특히, MSA를 도입하게 되면서 기존에 바로 모놀리식으로 만들었던 것보다 더 많은 생각을 했던 것 같습니다.

생각 끝에 다음과 같은 큰 주제들을 순서로 개발을 진행하기로 결정했습니다.

1. CI/CD 파이프라인 구성
   - 로컬에서만 테스트할 것이 아니라 구현한 서비스들이 클라우드 등에서 정상적으로 동작하는지 주기적인 확인이 필요하다고 판단
   - 비즈니스 로직이 거대해지만 그만큼 CI/CD 파이프라인에서도 고민한게 많아지게 되는데, 나중에 서비스들이 얽혀서 낭패를 보게 될 수도 있겠다는 생각
   - 지금은 하나의 서버에서 Docker network를 구성하여 컨테이너 간 IP 통신을 해서 서버 비용이 들지는 않지만, 이후 CI/CD 툴마다 클라우드 서버를 두고 동작시키게 될 때 발생하게 될 서버 비용이
     단점


2. MSA 환경 구성
   - CI/CD 구성과 비슷한 이유로 2번째에 작업 진행


3. 각 도메인 및 비즈니스 로직 구현

이런 큰 틀을 놓고 개발 계획을 세웠습니다.

### 🥕 프로젝트 예상 설계

<img src="https://i.ibb.co/cvxDpnx/Tosstock-12.jpg" alt="Tosstock-12" border="0">

- CI / CD

  - Jenkins는 모든 Spring 서버의 CI/CD 작동 현황을 한눈에 파악하기 위해 사용됩니다.
  - Ansible는 지정된 각각 다중화된 도메인 서버(docker container)에 배포 작업을 자동화하는 목적으로 사용됩니다.

CI/CD 파이프라인을 구성하는 툴들로 Jenkins와 Ansible을 사용하려 합니다. Jenkins는 CI(Continuous Integration)과 CD(Continuous Delivery)를 담당할 예정이며, Ansible가 CD(Continuous Deploy)를 담당할 예정입니다.

- Server Architecture

  - API Gateway는 분산된 도메인 API 요청을 하나로 모으는 역할과 인가를 담당합니다.
  - Eureka는 배포된 서버 IP와 Port를 확인하고 간단한 로드밸런싱을 위해 사용됩니다.
  - Config는 각 Spring 서비스의 설정파일을 관리하고 은닉화합니다.

서버 아키텍처로는 API Gateway, Eureka, Config를 사용하여 MSA 아키텍처 구성을 최대한 단순화하였습니다.

- Infra

  - MySQL은 기본적인 RDB로 사용되어질 예정입니다.
  - Redis는 로그인 단계에서 Session / Token 스토리지로 사용되어질 예정입니다.
  - RabbitMQ는 Spring Config에서 설정 파일 정보가 변경되어 업데이트가 필요할 때, RabbitMQ의 broadcast 방식으로 모든 Spring 서비스에 업데이트된 설정 파일을 자동으로 업데이트 하도록 사용될 예정입니다.
  - Kafka는 뉴스피드 시스템, 도메인 서비스 메시지 브로커 등에 사용되어질 예정입니다.

---

## 🌱 정리

많은 기술이 들어간만큼 기본기를 소홀히 하지 않도록 해야할 것 같습니다. 각 서비스들을 구현해나가면서 어떻게 구현하게 되었는지, 왜 구현했는지, 트러블 슈팅 등의 사례들을 소개할 생각에 괜히 설레는 하루입니다.

<br />

###### 🧑🏻‍💻 Stalk 프로젝트의 다양한 이야기

- [아기자기한 라즈베리파이 서버 구축](https://syeon2.github.io/devlog/tosstock-server.html)
- [Jenkins & Ansible을 활용한 CI/CD 구축기](https://syeon2.github.io/devlog/stalk-ci-cd.html)
- [Spring Cloud Config로 분산된 마이크로 서비스 설정 중앙 관리](https://syeon2.github.io/devlog/stalk-msa-config.html)
- [Service Discovery로 마이크로 서비스 주소 관리하기](https://syeon2.github.io/devlog/stalk-service-discovery.html)


- [JPA의 N + 1 Query issue 성능 개선 사례](https://syeon2.github.io/devlog/tosstock-query-n+1.html)
- [@Async을 활용한 이메일 수신 API 개선 방안](https://syeon2.github.io/devlog/tosstock-mail-sender.html)
- [JPA 변경감지 vs QueryDSL : 회원 정보 수정 최적화 방안](https://syeon2.github.io/devlog/tosstock-improve-updatequery.html)