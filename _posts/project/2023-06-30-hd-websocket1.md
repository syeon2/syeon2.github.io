---
layout: post
title: "[Hot-Dealicious #2] WebSocket 통신"
subtitle: "라이더와 WebSocket 통신"
categories: project
tags: project Network WebSocket
---

> 라이더와 지속적인 위치 정보 Request를 위한 기술 스터디

<!--more-->

## 🌱 기능 구현을 위한 탐색
이번 프로젝트에서 핵심적인 기능을 소개하자면 라이더의 위치 정보를 지속적으로 Request 받아 소비자 클라이언트에게 전송하여 실시간 위치 정보를 제공하는 
기능을 꼽을 수 있을 것 같다. (WOW!!)

이 기능을 구현하기 위해 다양한 환경을 고려해야했다. 제일 먼저 고려했던.. 가장 중요한 것은 서버 어플리케이션의 다중화였다. WebSocket으로 클라이언트와 
요청을 주고받기 위해서는 반드시 처음에 연결된 서버와 지속적으로 통신해야한다.(아래에 더 자세하게!) 하지만 이 프로젝트의 가장 큰 특징 중 하나는 
서버 어플리케이션을 여러개 두어 로드벨런서로 분산 처리를 하는 다중화를 고려하여 구현하는 것이다. 그러므로 클라이언트가 하나의 서버와 지속적인 통신을 
주고받기 위해서는 특별한 장치가 마련되어야 했다.

다음으로 Redis를 적절하게 사용하는 것이 포인트이다. 이미 로그인 & 세션 파트를 구현하면서 Redis Session을 사용했었지만 이번 기능을 통해 
Redis에 Geo 데이터를 저장하는 용도, 서버와 클라이언트의 WebSocket 통신을 지속적으로 가능하게 해주는 Routing Table 등을 구현하면서 
Redis에 대한 지식을 확장하는 기회였다.

---

## 🌱 기능을 구현하기 위한 설계
### 🟤 어떤 기술이 필요할까?
클라이언트와 효율적으로 통신하기 위해서 WebSocket 통신 방식을 사용하기로 했다. Http는 Stateless Protocal이므로 서버 혹은 클라이언트에서 요청/응답에 대한 
상태값이 변경되면 다시 Http 통신을 하여 Refresh를 시켜주어야 하는 단점이 있었다. 이런 단점을 해소하기 위해 클라이언트에서 서버에 주기적으로 데이터를 
요청하는 Polling 방식이나 서버에서 주기적으로 클라이언트에게 Push하는 Server-Sent Events가 있다.

하지만 이러한 통신 방식 또한 한계점을 가지고 있다. 첫번째로는 클라이언트 혹은 서버에서 주기적으로 Refresh하는 요청/응답을 하더라도 조금의 
딜레이는 존재하게 된다. 즉, 실시간 데이터가 아니라는 한계점이다. 두번째로는 클라이언트와 서버가 통신을 할때마다 4-way handshake같은 네트워크 처리 
과정이 필요해진다. 이런 부분은 어떻게 보면 클라이언트, 서버 양쪽 모두 자원을 불필요하게 낭비하는 과정이 발생하게 된다.

WebSocket을 사용하게 되면 위의 요소들을 해소할 수 있게 된다. 클라이언트와 서버가 한번 연결되면 이후 추가적인 커넥션 과정을 거치지 않고 바로 데이터를 요청/응답할 수 
있어서 네트워크 통신에 필요한 과정을 단축시킬 수 있다. 이는 양측 모두 실시간으로 데이터를 반영할 수 있게된다는 의미이기도 하다.

당연히 장점이 있다면 단점도 있기 마련..! 단점으로는 역시 유지보수/관리해야하는 비용이 추가된다는 것이다. 아무래도 단순한 Http 요청이 아니기 때문에 WebSocket 통신을 
구현하기 위해 추가적으로 필요한 장치들이 있을 것이다. 이러한 요소들을 관리해야하는 비용적인 측면이 존재한다. 그리고, 여러 장치가 추가된다는 것은 코드와 아키택처에 대한 
복잡성이 커진다는 것이다. 잠깐 위에서 언급했지만, 분산 환경에서 처리하고 있는 서버 중 하나와 클라이언트가 지속적으로 커넥션을 유지하며 통신하기 위해서는 코드와 아키택쳐 방면으로 
추가적인 기능이 필요하다고 판단할 수 있다. 그리고 서버와 클라이언트 간의 커넥션을 끊지 않고 계속 연결해야하기 때문에 커넥션에 대한 메모리도 조심해서 관리해야할 것이다.

이러한 특징들을 가지고 있기 때문에 초기에 기획과 설계를 하는 과정에서 충분한 비교를 해봐야할 것이다. 유지보수할 수 있는 인력은 충분한지, 현재 구현되어 있는 어플리케이션에 당장 
적용할 수 있는 기능인지, 없다면 무엇이 필요하고 기능 구현에 필요한 시간 등 다양한 요소들을 고려해봐야할 것이다. <br />
여기서는 프로젝트이기 때문에 WebSocket 스터디를 하고 싶어 적용하게 되었다.

---

### 🟤 필요한 기술 다음에 고려할 것은 어떻게 현 아키택처에 적용할지 고민하는 것!
<img src="https://wkblog-images.s3.ap-northeast-2.amazonaws.com/project/hot-dealicious/webSocekt-architecture.jpeg" />

여러가지 고민을 하면서 설계해보았다. 현재 서버의 상황은 이렇다.
1. WebSocket을 처리할 서버는 분산 처리를 위한 다중화가 되어 있다.
2. 로드벨런서로 HA Proxy를 선택했다. (Nginx도 선택지에 있었지만, Nginx는 Health Check 기능이 유료인점..🥲 HA proxy는 무료로 제공된다.)

그림을 보기만해도 가장 큰 특징은 분산처리를 위한 서버 다중화이다. 서버를 다중화하고 부하 분산 장치인 로그벨런서를 두어 대용량 트레픽에 견딜 수 있도록 설계한 것이 중점이다. 
이러한 환경에서 WebSocket이 안정적으로 관리되기 위해서는 별도의 장치가 필요하다. 바로 Routing Table이다.

라이더는 처음에 서버와 WebSocket 통신을 맺을 때 Routing Table에서 WebSocket 서버의 URL을 조회한다. 만약 Routing Table에 URL이 있다면 바로 클라이언트는 
특정 서버의 URL에 위치 정보를 Request 보낸다. 하지만 Routing Table에 URL이 없을 경우 클라이언트는 로드벨러서에 요청하고 로드벨런서가 적절한 WebSocket 서버로 
요청하여 이후 요청부터는 WebSocket 서버와 클라이언트가 다이렉트로 통신하도록 한다.

클라리언트로부터 요청받는 Geo 데이터는 공통 Redis 서버를 두어 분산 환경에서도 데이터를 공유 가능하도록 설계하였다.

###### [토스 증권의 실시간 시세 적용기](https://www.youtube.com/watch?v=WKYE-QtzO6g&list=PL1DJtS1Hv1PiGXmgruP1_gM2TSvQiOsFL&index=39) 영상을 많이 참고하였다.

---

## 🌱 실제 코드로 구현해보자!🧑🏻‍💻
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfigurer implements WebSocketMessageBrokerConfigurer {

	@Override
	public void configureMessageBroker(MessageBrokerRegistry registry) {
		registry.setApplicationDestinationPrefixes("/rider");
	}

	@Override
	public void registerStompEndpoints(StompEndpointRegistry registry) {
		registry.addEndpoint("/ws")
			.setAllowedOrigins("*");
	}
}
```

Spring 기반의 서버이기 때문에 @Configuration으로 DI 컨테이너에 등록한다. 여기서 Broker라는 개념이 등장한다. Spring에서 구현하는 Stomp의 Broker는 `/pub/*`으로 요청이 오는 
Request들을 인메모리에 각각 저장하여 해당 URL을 구독하는 Subcriber에게 메시지를 전달하는 역할을 한다.

### ❗️ Stomp와 Broker
Stomp는 WebSocket 위에서 동작하는 프로토콜이다. 단순히 WebSocket을 사용하는 것보다 많은 이점을 가지고 있는데 
이 부분은 타 [블로그](http://minjoon.com/spring-stomp)를 첨부한다.

###### [STOMP - Web Socket](http://minjoon.com/spring-stomp)

---

```java
/**
 * Redis Routing Table이 있는 서버에서 실행되는 API
 * 메인 서버 어플리케이션에서 실행되지 않는다. (단, 여기에서는 프로젝트이기 때문에 하나의 서버로 동작한다.)
 */

@RestController
@RequestMapping("/api/v1/rider")
@RequiredArgsConstructor
public class WebSocketRoutingTableController {

	private final RedisTemplate<String, String> redisRoutingTableTemplate;

	@GetMapping("/{id}")
	public ApiResult<String> getConnectedWebSocketUri(@PathVariable String id, HttpServletRequest request) {
		String str = redisRoutingTableTemplate.opsForValue().get(id);

		return ApiResult.onSuccess(str);
	}
}
```

이 부분은 웹 소켓 서버에서 실행되는 컨트롤러는 아니다. 라이더가 WebSocket 통신을 할 때 이미 WebSocket과 커넥션되어 있는 서버 URL이 있는지 확인하는 
API이다. 이 컨트롤러와 커넥션되어 있는 Redis 라우터와 함께 하나의 라우팅 테이블이 있는 서버로 배포되어야한다. 지금은 프로젝트라 하나의 어플리케이션 코드에서 구현한다고 
이렇게 구현하게 되었다.

---

```java
@MessageMapping("/commute")
public ApiResult<Long> checkCommute(RiderGeoDto riderGeoDto) {
	
	Point point = new Point(riderGeoDto.getData().getX(), riderGeoDto.getData().getY());
	redisGeoTemplate.opsForGeo().add("myMap", point, riderGeoDto.getRiderId());

	if (riderGeoDto.getWorkStatus() == WorkStatus.INIT) {
    	connectWebSocketServerToRider(riderGeoDto.getRiderId());
	}

	return ApiResult.onSuccess(riderGeoDto.getRiderId());
}
```

이 메소드가 WebSocket 통신할 때 호출되는 메소드이다. 여기에서는 Request받는 파라미터 타입을 적절히 활용하여 Redis에 Geo 데이터를 저장하도록 했다.

---

## 🌱 마무리
처음에 WebSocket에 관심을 가졌을 때는 안드로이드 개발자로 현업에 있을 때다. 그 당시에는 WebRTC 기술을 활용하여 화상 미팅 기능을 지원하는 
SendBird API를 사용하면서 네트워크 통신의 심오함에 관심을 가지게 되었다.(네트워크.. 너무 깊고 넓다..🌊)

이번 프로젝트에서 WebSocket을 활용하는 기능을 구현하게 되어 처음에는 새로운 개념에 머릿속이 복잡하기도 했고 어떻게 풀어가지? 고민도 되었다. 완벽주의 성향이 있어서 
 완벽한 프로젝트가 되지 않으면 레포지토리에 반영하거나 블로그 글을 작성하기 꺼렸다.

최근에 토스의 개발자 컨퍼런스 영상을 보면서 인사이트를 얻은 문장이 있었다. "정답은 없다."라는 문구이다. 현업에 있을 때도 느꼈지만 정말 기능을 구현할 때는 어떤 방법을 사용하든 
어떻게 구현하든 사람들마다 다른 방향성을 가지고 접근했었다. 여러 선택지 중에서 베스트 케이스를 판단하는 것이 중요하다고 느꼈다. 베스트 케이스를 선택하는 기준은 그 기능을 
논리적으로 뒷받침해줄 CS 지식과 다양한 사례들을 기준으로 선택하는 것..! 결국 지금 당장 가지고 있는 지식은 부족할 수 있지만, 앞으로 성장하면서 얻게될 지식들을 기반으로 꾸준히 
보완하며 개선해나가는 것이 프로그래밍을 업으로 하는 사람들의 마음가짐이 아닐까 한다.

이 프로젝트도 나와 함께 성장하는 어플리케이션이 되었으면 하는 바람이다.🙏🏻
