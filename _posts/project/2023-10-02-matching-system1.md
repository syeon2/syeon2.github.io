---
layout: post
title: "[Matching System #1] 프로젝트 개요 및 Architecture & 기능 Flow"
subtitle: "Matching System 전반적인 소개"
categories: project
tags: project websocket stomp
---

> Matching System 프로젝트를 소개합니다.

<!--more-->

## 🌱 Matching System 프로젝트 개요

Matching System은 특정 회원 반경에 매칭할 수 있는 회원이 있다면 매칭되어 위치 데이터를 지속적으로 전달할 수 있도록 하는 토이 프로젝트입니다.

배달 플렛폼, 위치 추적 어플 등에서 볼 수 있는 기능을 참고하여 WebSocket & STOMP, Redis의 다양한 활용등을 학습하기 위한 프로젝트입니다. 


### 🟤 사용된 기술

- for Application : Java 11, Spring boot 2.7.xx,
- for Database : MySQL, Redis
- for network : WebSocket, STOMP


### 🟤 프로젝트 핵심 기능 및 트러블슈팅

- WebSocket & STOMP 통신 기법을 활용한 지속적인 통신
- 서버의 Scale out을 고려하여 Redis에 GEO 데이터 저장 및 API 활용

----

## 🌱 핵심 기능 Flow

<img src="https://i.ibb.co/cw0YfHx/matchingflow.jpg" alt="matchingflow" width="850" />

이 프로젝트의 핵심기능 Flow는 아주 간단?합니다.

1. MemberA는 서버와 WebSocket Connect하여 지속적으로 위치 데이터를 전송 및 서버에 저장합니다.
2. MemberB는 자신의 위치에서 반경 X 범위 내에 있는 MemberA들 중 하나를 조회하고, 해당 MemberA의 위치를 Polling 방식을 사용하여 위치 데이터를 받습니다.

크게 이렇게 2가지의 Flow를 가지고 있습니다.

MemberA의 위치를 지속적으로 서버에 저장하기 위해 사용한 기술로 WebSocket을 선택하였습니다.

MemberA가 active 상태가 되어 자신의 위치를 서버에 제공하기 시작하면, MemberA 클라이언트가 종료하기 전까지 위치 데이터를 빠르게 전송하는 것을 목표로 하였습니다. 
일반적인 Http API를 사용하면 위치 데이터 하나를 보낼 때마다 3 handshake, 4 handshake 등 네트워크를 연결하기 위한 추가적인 동작이 수행됩니다. 
이러한 컴퓨팅 자원을 줄여 성능을 높이고자 WebSocket을 활용하였습니다.

------

## 🌱 프로젝트 Architecture

<img src="https://i.ibb.co/km407sZ/matching-architecture-001.jpg" alt="matching-architecture-001" width="850" />

해당 프로젝트는 유독 Redis를 많이 사용되는데 각각 Redis 서버마다 다른 역할을 담당합니다. Redis의 GEO API를 활용하여 위치 데이터를 저장 및 반경 범위 내의 
데이터를 조회할 수 있는 기능 등을 지원합니다.

또한, MemberB가 계속 매칭된 MemberA의 위치 데이터를 조회하기 위해서는 MemberA의 아이디값이 필요합니다. 이 때, 2가지의 방법을 생각해볼 수 있었습니다.

1. MemberB에게 매칭된 MemberA의 아이디를 제공하는 것입니다. 이 방법은 간단하지만 클라이언트 개발에 의존적입니다. 또한, 매칭된 클라이언트 아이디가 
클라이언트 코드에 노출된다는 특징을 가지고 있습니다.


2. MemberA와 MemberB를 맵핑시켜주는 코드가 서버에 필요합니다. 이 때 서버가 단일 서버라면, Map같은 자료구조에 각 맵핑된 정보를 저장할 수 있었겠지만, 서버 개발자는 
언제나 서버의 Scale out, SPOF 등을 고려해야합니다. 그렇기 때문에 다중 서버로 Scale out 한다면 Map 자료구조를 사용한 맵핑 정보는 무용지물이 됩니다. 
이를 해결하기 위해 Client to Client를 할 수 있도록 Routing Table을 Redis로 구현합니다.


-------

## 🌱 마무리

Matching-System은 Hot-Dealicious 프로젝트를 개선하고 조금 더 정리하고자하는 목표를 가집니다. 이를 통해 WebSocket, Redis를 심층적으로 이해하고자 합니다. 
