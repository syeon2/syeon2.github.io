---
layout: post
title: "[Project] Pre Market 프로젝트를 소개합니다."
subtitle: "다양한 주문 서비스를 제공하는 Pre Market 프로젝트를 소개합니다."
categories: devlog
tags: project
---

> 다양한 주문 서비스를 제공하는 Pre Market 프로젝트를 소개합니다.

<!--more-->

#### 목차

- [여는 글](#-여는-글)
- [사용된 기술](#-사용된-기술)
- [시스템 아키텍처](#-시스템-아키텍처)
- [ERD](#-erd)
- [문제 및 트러블슈팅](#-문제-및-트러블슈팅)

---

## 🌱 여는 글

Pre Market 프로젝트는 다양한 주문 서비스를 통해 사용자에게 폭넓은 경험을 제공합니다. 일반 상품의 여러 건을 한 번에 주문할 수 있는 기능과 더불어, 공연이나 스포츠 경기 티켓처럼 선착순 예약 구매 기능 등 빠르고 신뢰성 있는 기능을 구현하였습니다.  
특히 급격히 증가하는 트래픽을 안정적으로 관리해야했던 예약 구매 기능이 중요한 과제였습니다. 이를 해결하기 위한 아키텍처와 시스템 설계 등을 고민하고, 모든 트러블슈팅 과정을 CS, 데이터 등을 기반으로 근거 있는 해결 과정을 소개합니다.

Pre Market 프로젝트에서 고민한 핵심 기술과 이를 통해 얻은 인사이트를 소개하고자 합니다:

- Pre Market 소프프웨어 아키텍처 (Layered & DDD)
- JWT의 Access Token & Refresh Token 매커니즘을 활용한 인증/인가
- 스파크성 트래픽을 안정적으로 처리하는 예약 주문 기능 설계

이 프로젝트의 고민과 트러블슈팅 과정을 바탕으로, 안정적인 서비스를 제공하기 위한 접근 방식을 공유하려 합니다.

---

## 🌱 사용된 기술

#### Java 17, Spring Boot 3.x.x, MySQL, Redis, JPA, QueryDSL, Kafka, AWS EC2, Docker, Jenkins

---

## 🌱 시스템 아키텍처

<img src="https://i.ibb.co/ZYV61xK/premarket-architecture.jpg" alt="premarket-architecture" border="0">

---

## 🌱 ERD

<img src="https://i.ibb.co/mtNQpPL/premarket-erd.png" alt="premarket-erd" border="0">

---

## 🌱 문제 및 트러블슈팅

- [Pre Market 소프프웨어 아키텍처 (Layered & DDD)](https://syeon2.github.io/devlog/pre-market-architecture.html)
- [많은 트래픽을 안정적으로 처리하는 예약 주문 기능 설계](https://syeon2.github.io/devlog/premarket-concurrency.html)
- [상품 주문 쿼리 최적화 사례](https://syeon2.github.io/devlog/premarket-order-query.html)
- 