---
layout: post
title: "[Matching System #2] WebSocket & STOMP를 활용한 위치 데이터 저장"
subtitle: "WebSocket & STOMP"
categories: project
tags: project websocket stomp
---

> WebSocket & STOMP 통신 기법 및 Redis geo API 사용

<!--more-->

## 🌱 WebSocket & STOMP를 활용한 API 호출

WebSocket는 하나의 통신 기법으로 네트워크 커넥션을 유지하며 데이터를 전송시키는 기술입니다. WebSocket은 통신 기법일 뿐이며, 데이터를 전송시킬 수 있는 
간편한 폼을 제공하는 것이 STOMP입니다. STOMP는 텍스트 기반의 메시지 전송 폼을 제공합니다.

기술에 대한 자세한 설명은 정리를 잘해주신 타 블로그를 대신합니다.

> ❗️참고 레퍼런스 - [Spring Websocket & STOMP](https://brunch.co.kr/@springboot/695)

----

### 🟤 프로젝트에서 활용한 사례 분석

Matching System 토이 프로젝트는 서버에 등록한 회원의 위치를 지속적으로 트래킹하는 기능을 가지고 있습니다. 마치 카*오 모빌리티의 기능 중 하나인 택시를 잡는 상황을 생각할 수 있습니다. 
회원(ex. 택시 기사)은 서버에 자신이 위치한 지리 정보를 갱신합니다. 이 때, 서버와 회원은 WebSocket으로 커넥션을 맺고 지속적 통신을 하게 됩니다.

<img src="https://i.ibb.co/f1FbStv/matching-system1.png" alt="matching-system1" width="850" />

...

하지만 일반적인 TCP를 사용한 HTTP 통신을 하게 되는 경우는 이러한 Flow를 가지게 됩니다.

<img src="https://i.ibb.co/1rLxK66/matching-system2.png" alt="matching-system2" width="850" />

이 두가지 Flow의 장단점을 비교해보고자 합니다.

- TCP 통신의 장점
  - 보편적으로 사용되는 통신 방식이기 때문에 새로운 아키텍쳐를 구성할 오버헤드가 줄어든다.
  - 코드 레벨에서도 일반적인 API를 구현하는 것과 동일하기 때문에 추가적인 코드 이해가 불필요하다. (접근성이 용이하다.)

- TCP 통신의 단점
  - 단방향으로 한번의 요청을 보내게 되면 그것으로 통신은 끝이다. (header 옵션인 Keep-Alive는 예외)
  - 한번 통신할 때마다 TCP 3 way handshake를 거쳐야하기 때문에 이에 대한 비용이 추가적으로 발생한다.

...

- WebSocket 통신의 장점
  - 한번 커넥션을 맺은 이후로는 3 way handshake 같은 과정없이 실시간으로 양방향 통신할 수 있다.

- WebSocket 통신의 단점
  - WebSocket은 계속 커넥션을 맺고 있기 때문에 서버와 클라이언트 간 커넥션이 많아지게 되면 쉽게 부하가 걸린다.
  - 새로운 형태의 아키텍쳐와 코드가 추가되기 때문에 새롭게 익혀 적용하는 비용이 발생한다.

이 정도로 구분할 수 있을 것 같습니다.

> ❗️ 3 way handshake라는 오버헤드를 줄이기 위한 다른 방안들
> 
> 1. TCP 통신할 때의 Header 옵션인 Keep-Alive를 활성화한다. <br />
> 
> - 현 프로젝트 한정으로 제일 유력한 대체 기술이라고 생각합니다. TCP 통신 그대로 API를 만들면 되기 때문에 어렵지 않게 구현할 수 있으며, 
> 일정 시간 동안 커넥션을 유지하기 때문에 3 way handshake에 대한 오버헤드를 줄일 수 있습니다.
> 
> => WebSocket 학습 프로젝트가 아니었다면 Keep-Alive를 활성화하는 방법으로 프로젝트를 구현했을 것 같습니다.
> 
> --------
> 2. UDP 통신으로 3 way handshake 과정을 생략한다. <br />
> - UDP는 3 way handshake 과정을 생략하고 그저 데이터를 전송합니다. 따라서 TCP 통신보다 빠르다는 장점을 가지고 있습니다. 
> 하지만 데이터의 순서가 보장되지 않고, 데이터의 신뢰도가 낮다는 단점을 가지고 있습니다.
> 
> => 위치를 서버에 갱신하는 기능이기 때문에 UDP 통신 기법을 사용해서 해당 프로젝트를 구현하는 것도 좋았겠다는 생각을 합니다.

-----

### 🟤 코드로 보는 사례

WebSocket 또한 처음 커넥션을 맺기 위해서는 WebSocket handshake 과정을 거쳐야합니다. 처음에 커넥션을 맺을 때는 HTTP 프로토콜로 커넥션을 맺게 되면 Response로 
101 status code를 받게 됩니다. 이 이후로 클라이언트와 서버는 ws 프로토콜을 사용해서 양방향 통신을 할 수 있게 됩니다.

<img src="https://i.ibb.co/s6F3Kfq/matching-system3.webp" alt="matching-system3" width="850" />

###### 출처 : https://adequatica.medium.com/how-to-manually-test-websocket-apis-855393911d1a

------

#### 1. WebSocket Config
먼저 WebSocket 통신을 활성화하기 위해서는 Config class를 구현해야합니다.

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
	
  @Override
  public void configureMessageBroker(MessageBrokerRegistry registry) {
    registry.enableSimpleBroker("/sub");
    registry.setApplicationDestinationPrefixes("/pub");
  }

  @Override
  public void registerStompEndpoints(StompEndpointRegistry registry) {
    registry.addEndpoint("/ws")
      .setAllowedOrigins("*");
  }
}
```

간단하게 Config를 구현했습니다. 이번 프로젝트에서 WebSocket은 STOMP라는 하위 프로토콜을 사용하여 메시지들을 전송합니다. 
STOMP는 pub/sub 이라는 발행-구독 같은 메시지 브로커 형태로 동작합니다. /sub/* 이라는 URL을 클라이언트가 구독하게 되면 pub을 통해 
메시지를 보낼 수 있게 됩니다.

Config 작성을 마치게 되면 `ws://localhost:8080/ws`로 HTTP 커넥션을 맺을 수 있습니다.

----

#### 2. WebSocket API

커넥션을 맺으면 다음과 같이 Controller에서 API를 호출할 수 있습니다.

```java
/**
 * WebSocket Connection 이후 지속적으로 위치 데이터 전송 API
 */
@MessageMapping("/location")
public void reflectPositionOfMember(@Payload GeoLocation geoLocation) {
      // ...
}
```

@MessageMapping은 WebSocket을 통해 클라이언트가 메시지를 보내는 목적지를 지정하는 등의 기능을 가지고 있습니다. 마치 PostMapping, GetMapping 등과 같이 사용됩니다.

@Payload 또한 @MessageMapping으로부터 받은 메시지를 GeoLocation으로 deserialization하여 받은 메시지를 객체화합니다. 이 객체를 통해 메시지 전달한 회원 ID, 위도, 경도 등의 데이터를 받습니다.

...

<img src="https://i.ibb.co/G90szty/matching-system4.png" alt="matching-system4" width="850" />

Apic이라는 툴을 사용해 WebSocket & STOMP 으로 구현된 API를 테스팅합니다.

1. ws://localhost:8080/ws 라는 URL에 먼저 커넥션을 합니다.
2. Subscription URL에 구독하고자 하는 URL을 입력합니다.
3. Destination Queue에 발행할 API 주소를 입력하고, 보낼 메시지를 입력합니다.

#### 3. 호출 결과

위치 정보를 저장하는 방법으로는 여러가지가 있지만 Redis가 Geo 데이터를 활용한 여러 API를 지원하는 것을 보고 Redis를 활용하기로 결정하였습니다.

Apic 툴을 사용해 memberId마다 위치 데이터를 갱신하면 Redis에 저장되는 값들도 함께 갱신됩니다.

<img src="https://i.ibb.co/W52MBnB/matching-system5.png" alt="matching-system5" width="850" />
