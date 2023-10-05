---
layout: post
title: "[Matching System #3] WebSocket 서버의 Scale out"
subtitle: "WebSocket 서버 확장"
categories: project
tags: project websocket scale-out
---

> WebSocket 서버의 scale out을 고려한 아키텍쳐 설계

<!--more-->

## 🌱 WebSocket 서버의 한계

WebSocket을 통해 클라이언트와 서버는 계속 커넥션을 유지하면서 양방향 통신할 수 있게 됩니다. 하지만 커넥션을 계속 유지하기 위해서는 서버의 트래픽과 스레드 등 
여러 자원을 활용해야하기 때문에 쉽게 컴퓨팅 리소스가 고갈될 수 있습니다.

대표적으로 WAS의 스레드 풀이 고갈될 수 있습니다. 서버로 유입되는 클라이언트 요청을 병렬적으로 처리할 수 있게 스레드를 사용하는데, 스레드를 무차별적으로 
생성하여 요청을 처리하게 되면 스레드를 생성하기 위해 필요한 메모리가 고갈되어 OOME 같은 에러가 발생할 수 있습니다.

> 수많은 클라이언트와 서버 간의 WebSocket 커넥션을 원활하게 하기 위해서는 Scale out이 필수적입니다.

----

## 🌱 WebSocket 서버의 Scale out과 고려해야할 사항

WebSocket 서버는 클라이언트와 한번 커넥션을 맺으면 지속적으로 통신합니다. Nginx나 HA proxy, API Gateway 같은 reverse proxy에서 한번 
scale out된 서버와 커넥션을 맺으면 이후 ws 프로토콜로 메시지를 주고받기 때문에 처음 커넥션 맺을 때 로드벨런싱이 되어 적절한 서버와 커넥션을 맺게 됩니다.

서버에 클라이언트 A의 위치 데이터만 저장한다면 복잡한 Scale out은 필요없을 것 입니다. 하지만 해당 기능은 클라이언트 A가 클라이언트 B의 위치를 조회합니다. 이 문제는 
WebSocket 서버를 어떻게 안전하게 Scale out 할 것인지에 대한 고민을 안겨줍니다.

<img src="https://i.ibb.co/vj7M5Yb/matching-system6.png" alt="matching-system6" width="850" />

이 때는 Member A의 위치 정보가 WebSocket 서버 한대에 저장되어 있기 때문에 Member B는 Member A의 정보를 WebSocket 서버 한대에서 조회할 수 있습니다. 
Member B의 주변 사람들 중 Member A가 있다면 서버에 Member A와 Member B가 매칭되었다는 정보를 저장한 후 Member B는 WebSocket 서버만으로 Member A의 정보를 
조회할 수 있게 됩니다.

하지만 위에서 언급된 것처럼 WebSocket 트래픽이 많아지게 되면 Scale out은 불가피합니다.

...

<img src="https://i.ibb.co/B3H6LxQ/matching-system7.png" alt="matching-system7" width="850" />

위의 그림처럼 WebSoket 서버를 확장하게 되면 생기는 문제점입니다. Member A의 위치는 Redis 저장소에 저장합니다. 그 다음으로 Member B가 
Member A와 매칭되어 서버 A에 매칭 정보를 저장하고 통신하게 된다면, Member B가 LB에 의해 서버 B에 매칭된 회원의 정보를 요청했을 경우는 에러가 나게 됩니다.

이와 같은 문제를 해결하기 위해 매칭자 - 피매칭자 간의 정보를 저장할 수 있는 Routing Table이 필요하게 됩니다.

----

## 🌱 Routing Table로 구현

여러가지 이유로 인해 Routing Table을 Redis로 관리하도록 하였습니다. 

1. 스토리지 접근 속도가 빨라야합니다. 
   - Redis는 In-memory 기반의 형태의 메모리를 제공합니다. 일반적으로 Disk에 데이터를 저장하는 DB보다 더욱 빠르게 조회할 수 있습니다.

2. key-value 형태의 데이터를 저장합니다.
   - key값을 MemberB의 회원아이디로, value를 MemberA의 회원아이디 형태로 저장하여 목적에 맞게 사용할 수 있습니다.


```java
@Configuration
public class RedisRoutingTableConfig {

	@Value("${spring.redis.routing-table.host}")
	private String host;

	@Value("${spring.redis.routing-table.port}")
	private Integer port;

	@Bean
	public RedisConnectionFactory redisRoutingTableConnectionFactory() {
		return new LettuceConnectionFactory(host, port);
	}

	@Bean
	@Qualifier(value = "redisRoutingTableTemplate")
	public RedisTemplate<String, String> redisRoutingTableTemplate() {
		RedisTemplate<String, String> redisTemplate = new StringRedisTemplate();
		redisTemplate.setConnectionFactory(redisRoutingTableConnectionFactory());

		return redisTemplate;
	}
}
```

key-value(memberB id - memberA id)로 key-value 전략을 두어 위와 같은 Config를 구성하였습니다.


----

## 🌱 한계점

Redis같은 기술들을 사용하면 언제나 트레이드 오프가 있기 마련입니다.

Redis를 사용한다는 것은 스토리지 서버를 새로이 구축한다는 의미입니다. 이것은 또다른 SPOF 문제를 야기하게 됩니다. Routing-Table 기능을 하는 Redis 서버에 장애가 발생하면 
장애를 failover해야하는 아키텍쳐를 구축해야한다는 의미이기도 합니다.

Redis Cluster 구성을 통해 쉽게 다중 서버로 구축이 가능하지만 아키텍쳐를 고려해야하여 설계해야하는 시간적 비용, 서버가 운영되는 재정적 비용 등의 상황에 맞닥뜨릴 수 있습니다.

사실 문제를 해결하는데 정답은 없지만 `해답`은 있다고 생각합니다. 제품에 새로운 기능을 구현할 때는 시간적 비용과 재정적 비용 등의 상황을 고려하여 최선의 것을 선택할 수 있도록 해야할 것입니다.  
