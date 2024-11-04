---
layout: post
title: "[Project] 예약 상품 주문 요청 속도 개선 사례를 소개합니다. "
subtitle: "Redis의 Rua Script를 활용하여 동시성 이슈 및 속도를 개선한 사례입니다."
categories: devlog
tags: project
---

> Redis의 Rua Script를 활용하여 동시성 이슈 및 속도를 개선한 사례입니다.

<!--more-->

#### 목차

- [여는 글](#-여는-글)
- [상품 재고 처리 속도 개선 트러블 슈팅](#-상품-재고-처리-속도-개선-트러블-슈팅)
  - [원인 분석 및 문제 파악](#-원인-분석-및-문제-파악)
  - [Redis를 통한 재고 관리로 요청 처리량 이슈를 해결](#-redis를-통한-재고-관리로-요청-처리량-이슈를-해결)
- [결론](#-결론)

---

## 🌱 여는 글

[이전 글 (Pessimistic Lock으로 동시성 이슈 트러블슈팅)](https://syeon2.github.io/devlog/premarket-concurrency.html)에서 주문 시 상품 재고가 
정상적으로 처리되지 않는 이슈를 해결해보았습니다.

하지만 Pessimistic Lock으로는 동시성 문제를 해결했지만, 기획에서 요구하는 요청 속도는 현저히 미치지 못하고 있다는 것을 발견하게 되었습니다.

이번 글에서는 이전 동시성 이슈와 함께 속도 또한 개선해본 사례를 소개합니다.


## 🌱 상품 재고 처리 속도 개선 트러블 슈팅

### 🥕 원인 분석 및 문제 파악

결론부터 언급하자면, MySQL의 특징으로 인한 한계로 인해 발생한 문제입니다. 

MySQL은 디스크에 데이터를 저장하고 불러오는 특징을 가지고 있습니다. 어플리케이션(프로세스)에서 디스크에 접근하여 데이터를 불러오는 속도는 생각보다 많이 느리다는 점이 이 
문제의 포인트입니다.

<img src="https://i.ibb.co/YcMhTzs/diskandmemory.png" alt="diskandmemory" border="0">

위의 표는 각 메모리마다 연산 속도의 차이를 보여주는 표입니다. HDD는 SSD보다 500배 빠르지만, RAM(메인 메모리)는 SSD보다 1,000배 더 빠릅니다.

즉, SSD나 HDD에 데이터를 저장하는 MySQL 대신 RAM을 사용하는 방법을 사용한다면 위의 요구사항에 대한 이슈를 해결할 수 있을 것입니다.

#### ⚙️ 주문 속도 테스트

- 묙표 Thread(유저 수): 10,000명
- 요청 총 시간: 10초 (1초에 약 1000명의 요청)
- 초기 상품 수량 : 10,000개
- 예상 결과 : 10초 뒤 해당 상품의 수량 0


- 테스팅 컴퓨터 : 맥북 M1 프로 맥스
- 디스크 : SSD를 기본으로 사용

위의 상황에서 Jmeter로 테스트한 결과는 다음과 같습니다.

<img src="https://i.ibb.co/zfH1ZS3/order-spped1.png" alt="order-spped1" border="0">
<img src="https://i.ibb.co/3CV3hFc/order-speed2.png" alt="order-speed2" border="0">

10초동안 요청했던 10,000개의 요청, 1초에 1,000개의 요청 처리를 해야하는 해당 로직은 총 30초가 걸려서야 마무리가 되었습니다. 게다가 10,000개 모두 
처리한 양도 아닌 약 1,500개 정도 요청이 누락된 결과값이 위의 결과입니다.

그렇다면 현재 코드에서 1초에 최대 몇 요청을 처리할 수 있을지 테스트해보았습니다.

<img src="https://i.ibb.co/Bwn7sVS/order-speed3.png" alt="order-speed3" border="0">
<img src="https://i.ibb.co/HKrnZmd/order-speed4.png" alt="order-speed4" border="0">

10초에 최대 3,000 ~ 3,500개 사이로 요청을 처리할 수 있다는 것을 파악하였습니다.

<br />

#### ⚙️ 테스트 결과

해당 요청은 예약 구매 API만을 테스트한 결과이지만, 실제 서비스 시에는 이 API뿐만 아니라 다른 요청도 처리해야하기 때문에 더욱 많은 요청 처리량을 확보해야했습니다.

### 🥕 Redis를 통한 재고 관리로 요청 처리량 이슈를 해결

최소 2배의 요청 처리량을 더 확보해야하는 현 상황에서 해결할 수 있는 방법 중 Redis를 통한 재고 관리를 통해 문제를 해결하였습니다.

Redis는 디스크를 사용하는 MySQL 등의 DB와 다르게 메인 메모리를 사용하는 인메모리 DB입니다. 순수 디스크 처리 속도만으로는 약 1,000배 가량 높은 퍼포먼스를 
보여줄 수 있어 해당 이슈를 처리하기에 최적의 방법이라고 판단하였습니다.

#### ❗️ Redis의 동시성 이슈

Redis라고 해서 여러 스레드의 동시성 이슈를 피해갈 수는 없었습니다. Redis는 MySQL처럼 Pessimistic Lock을 지원하지 않기 때문에 다른 방법을 통해 동시성 이슈를 
해결해야했습니다.

Redis를 통한 동시성 제어 방법으로 setnx를 사용하는 방법이 있습니다.

```java
public void lock(Long itemId) {
	String lockKey = "lock:" + itemId;
	
	return redisTemplate.opsForValue().setIfAbsent(lockKey, "lock", Duration.ofMillis(3000L));
}

public void unlock(Long itemId) {
	String lockKey = "lock:" + itemId;

	redisTemplate.delete(lockKey);
}

public void deductItemQuantity(Long itemId, Integer quantity) {
    while (!lock(itemId)) {
        try {
            Thread.sleep(50);
        } catch (InterruptedException e) {
            //...
        }
    }
    
    try {
        Integer q = Integer.parseInt((String)redisTemplate.opsForValue().get(generatedKey(itemId)));
    
        if (q - quantity < 0) {
            throw new OutOfStockException("Out of stock");
        }
		
        redisTemplate.opsForValue().set(generatedKey(itemId), String.valueOf(q - quantity));
    } finally {
        unlock(itemId);
    }
}
```

해당 로직은 먼저 특정 상품 아이디에 대한 lock을 Redis에 먼저 생성하고 이후 재고 감소 로직을 처리합니다. Lock을 걸고 난 이후 재고 차감 로직을 실행하는데, lock을 먼저 취득하지 못한 
요청(스레드)는 lock을 취득할 때까지 while문을 반복합니다.

<img src="https://i.ibb.co/Y29r4ZG/request-thread.jpg" alt="request-thread" border="0">

해당 로직은 정상적으로 동작하는 것처럼 보이지만, Request 개수를 증가시키면 Thread 생성 및 CPU 가용률이 늘어나는 이슈를 동반합니다. lock을 취득하지 못한 
다른 request들은 unlock이 되었는지 계속 조회해야하는 매커니즘(Spin Lock)을 가지고 있습니다.

#### ⚙️ Redis Lock을 통한 구현 테스트
- 묙표 Thread(유저 수): 10,000명
- 요청 총 시간: 10초 (1초에 약 1000명의 요청)
- 초기 상품 수량 : 10,000개
- 예상 결과 : 10초 뒤 해당 상품의 수량 0

<img src="https://i.ibb.co/gwHxf5P/order-speed5.png" alt="order-speed5" border="0">

위의 표를 보면 thread수가 200까지 30초 안에 생성된 것을 볼 수 있습니다. thread가 계속 생성되고 이후에 수거되지 않는다면 Out of memory 가 발생하여 서버가 다운될 
수 있습니다. (Thread Pool을 설정하지 않았을 경우)

<br />

서버의 다운은 서비스에 매우 치명적인 이슈를 동반할 수 있습니다. 그렇다고 Thread Pool을 제한한다면 요청이 누락되어 좋은 품질의 서비스를 제공하지 못하게 될 것입니다.

따라서 lock이 아닌 다른 방법을 통해 이슈를 해결해야했습니다.

#### ❗️ Lua Script를 활용하여 Redis의 원자성 보장

문제를 다시 짚어보자면, 재고의 `조회`와 `수정`이 각각 독립적인 요청으로 동작하는데 있습니다. A 트랜잭션이 조회 후 수정 전에 B 트랜잭션이 수정 전의 재고를 조회하는 상황이 
동시성 이슈를 유발하는 근본적인 원인입니다.

<img src="https://i.ibb.co/h1VGsJT/order-speed9.jpg" alt="order-speed9" border="0">

만약 `조회`와 `수정`이 하나의 원자적 요청으로 수행된다면 Redis를 통한 동시성 제어와 메모리를 사용한 빠른 데이터 처리가 가능해져서 주어진 요건을 충족할 수 있을 것입니다.

Redis의 Lua Script는 Redis가 스크립트를 통해 명령을 수행하도록 하는 기능입니다. Lua Script로 작성된 처리는 하나의 원자적 요청으로 처리되어 현 상황에 적합한 도구라 판단하고 적용하였습니다.

```java
private static final String DECREMENT_STOCK_SCRIPT =
    "local currentStock = tonumber(redis.call('get', KEYS[1])) "
        + "local orderQuantity = tonumber(ARGV[1]) "
        + "if currentStock == nil then return -1 "
        + "end "
        + "if currentStock < orderQuantity then return 0 "
        + "end "
        + "redis.call('decrby', KEYS[1], orderQuantity) "
        + "return currentStock - orderQuantity ";

public void deductItemQuantity(Long itemId, Integer quantity) {
    DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>();
    redisScript.setScriptText(DECREMENT_STOCK_SCRIPT);
    redisScript.setResultType(Long.class);

    Long result = redisTemplate.execute(
        redisScript,
        Collections.singletonList(generatedKey(itemId)),
        String.valueOf(quantity));

    if (result == null) {
        throw new RuntimeException("redis execution error");
    } else if (result == -1) {
        throw new IllegalArgumentException("상품 정보가 존재하지 않습니다.");
    } else if (result == 0) {
        throw new OutOfStockException("재고가 부족합니다.");
    }
}
```

위의 코드는 Lua Script를 사용한 상품 재고 처리 로직입니다. 위의 코드를 통해 아래와 같은 결과를 얻을 수 있었습니다.

#### ⚙️ Redis Lua Script를 통한 구현 테스트
- 묙표 Thread(유저 수): 10,000명
- 요청 총 시간: 10초 (1초에 약 1000명의 요청)
- 초기 상품 수량 : 10,000개
- 예상 결과 : 10초 뒤 해당 상품의 수량 0

<img src="https://i.ibb.co/Jntg18H/order-speed7.png" alt="order-speed7" border="0">
<img src="https://i.ibb.co/qRXxSh7/order-speed8.png" alt="order-speed8" border="0">

Lua Script로 수정한 재고 차감 로직은 1초에 1000개의 요청을 거뜬히 수핼할 수 있게 개선되었습니다.

<img src="https://i.ibb.co/r7RsxBQ/order-speed6.png" alt="order-speed6" border="0">

또한, Redis에 lock 개념을 사용해서 200개 정도 생성되었던 상황과 달리 해당 로직은 thread가 약 60개 정도만 생성되어 서버 스레드로 인한 메모리 이슈를 
해결할 수 있었습니다.

## 🌱 결론

> Redis의 In-Memory 특징을 통해 I/O 시간을 단축시킬 수 있었으며, Lua Script를 통해 상품 재고 처리를 원자적 처리로 보장할 수 있게 되었습니다.

이 해결 방안으로 2가지의 이슈를 해결할 수 있었습니다.

- 재고 차감 동시성 이슈 해결
- TPS 200 -> TPS 1,000으로 개선

#### ❗️ 트래이드 오프 고려

기획 요구조건에 만족할만한 개선을 이루었지만, 다음과 같은 비용이 추가로 발생되었습니다.

- 예약 상품 재고를 위한 Redis
  - Redis를 지속적으로 모니터링하고 관리해야하는 비용이 추가되었습니다.
  - Redis는 In-Memory DB이기 때문에 데이터가 휘발된다는 특징이 있습니다. 이에 대비하여 Redis를 클러스터링 등 Redis 운영 안정성을 확보하는 등의 추가 운영 요소가 발생하였습니다.


- 기존 RDB에서 사용하던 재고 정보
  - 상품 리스트 조회 시 상품 재고도 함께 반환해야한다면 상품 리스트 개수만큼 Redis에서 추가로 재고 조회를 해야하는 점


등 예상하지 못한 트레이드 오프 이슈들이 발생할 수 있습니다. 결국 서버 아키텍처가 비대해지고 관리 비용이 증가하여 개발의 지속성이 감소할 수 있는 
상황이 발생할 수 있기 때문에 팀원과 적절한 협의를 통해 선택하는 과정이 중요하다고 판단되어집니다.