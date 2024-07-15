---
layout: post
title: "Service Discovery로 마이크로 서비스 주소 관리하기"
subtitle: "Service Discovery를 구축하면서 고민했던 사례들을 소개합니다."
categories: devlog
tags: msa service-discovery eureka
---

> Service Discovery를 구축하면서 고민했던 사례들을 소개합니다.

<!--more-->

- 목차
  - [🌱 여는 글](#-여는-글)
  - [🌱 Spring Cloud Netflix Eureka](#-spring-cloud-netflix-eureka)
    - [🥕 Client Side Discovery vs Server Side Discovery](#-client-side-discovery-vs-server-side-discovery)
    - [🥕 Stalk Project의 Service Discovery](#-stalk-project의-service-discovery)
  


## 🌱 여는 글

AWS 같은 클라우드 환경에서는 트래픽에 따라 자동적으로 서버를 증설 및 삭제하는 AutoScaling 기능을 가지고 있습니다. 이 의미는 서버마다 IP 및 Port가 동적으로 생성되어 기존의 값과 달라진 경우가 발생합니다.

Service Discovery는 사용하고 있는 서비스들의 IP 및 Port를 등록하고 관리할 수 있도록 돕는 역할을 합니다.

- Spring Cloud Netflix Eureka
  - 분산된 각 마이크로 서비스의 주소를 등록하여 API Gateway에 제공
  - 각 서비스의 위치를 중앙에서 관리

이와 같은 장점들을 가지고 있기에 Eureka를 활용하여 MSA 환경을 구축해보았습니다.

<br />

## 🌱 Spring Cloud Netflix Eureka

### 🥕 Client-Side Discovery vs Server-Side Discovery

<img src="https://i.ibb.co/VCk2VJK/service-discovery.webp" alt="service-discovery" border="0">

Service Discovery는 크게 2가지 패턴으로 나눌 수 있습니다.

Client-Side Discovery는 클라이언트 측에서 Service Discovery에 직접 접근하여 라우팅된 마이크로 서비스에 접근하는 방식입니다. 즉, 위의 그림에서 Load Balancer는 존재하지 않고 클라이언트가 직접 Discovery Service에 접근 및 라우팅된 주소에 
직접 요청합니다.

> - 구현이 간단 & 클라이언트가 적절한 서비스 인스턴스를 선택하고 요청을 보내는 로드벨런싱이 가능
> - 클라이언트와 서비스 레지스트리가 종속적

<br />

Server-Side Discovery는 Client에서 먼저 Load Balancer에 접근하면 Load Balancer가 요청 쿼리를 분석 후 Service Discovery에서 라우팅된 주소를 찾아 마이크로 서비스에 접근하는 방식입니다.

> - 클라이언트와 서비스 레지스트리가 분리
> - 클라이언트는 직접적으로 Service Discovery에 접근하지 않고, 대신 로드 밸런서가 이 역할을 수행
> - 로드 밸런서는 Service Discovery에서 적절한 마이크로 서비스의 주소를 찾아 요청을 라우팅하는 자동화된 프로세스를 수행
> - 로드벨런서라는 추가적인 운영 요소가 추가되는 리소스 발생

<br />
<br />


### 🥕 Stalk Project의 Service Discovery

Stalk Project는 Eureka와 Spring Cloud Gateway를 채용한 Server-Side Discovery 패턴을 적용하였습니다. 클라이언트가 서버의 각 라우팅된 주소 및 로드벨런싱을 관리하는 것보다 서버 측에서 관리하는 것이 더 효율적이라는 판단 하에 내린 결정이었습니다.

<img src="https://i.ibb.co/dmYP9sV/API-Gateway.jpg" alt="API-Gateway" border="0" width="800">

API Gateway로 활용된 Spring Cloud Gateway가 상단의 로드벨런서의 역할을 합니다. API Gateway는 클라이언트로부터 요청을 직접 받고, Eureka로부터 라우팅된 주소를 찾아 적절한 마이크로 서비스에 요청을 전달합니다.

API Gateway라는 추가적인 관리 비용이 발생했지만, 마이크로 서비스의 인증/인가, 로드 밸런싱 등 다양한 처리 등을 리버스 프록시 패턴으로 해결하고자 하는 니즈가 있어서 API Gateway는 필수적으로 구현해야하는 사항이었습니다.

때문에 Eureka와 Spring Cloud Gateway는 현 상황의 문제를 해결할 수 있는 좋은 선택지라고 판단하여 적용하였습니다.

<br />
<br />

###### 📚 참고자료
[Spring Cloud Gateway + Eureka](https://medium.com/@im_zero/spring-cloud-gateway-eureka-25567532cfcd) <br />
[Service Discovery DR 구성 1부 - Eureka 서버를 지역 분산시켜 안정성을 높이자](https://11st-tech.github.io/2022/12/30/eureka-disaster-recovery-1/)

----------------
