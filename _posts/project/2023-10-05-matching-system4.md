---
layout: post
title: "[Matching System #4] Redis Distributed Lock (feat. 동시성 이슈)"
subtitle: "Distributed Lock"
categories: project
tags: project distributed lock
---

> Redis를 사용한 분산락으로 동시성 이슈를 해결한 사례입니다.

<!--more-->

## 🌱 분산락이 필요한 기능 소개

Matching-System 프로젝트에는 현재 클라이언트의 위치 정보(위도, 경도)에서 반경 5km 떨어져있는 Online인 사람들 중 한명과 매칭되는 기능을 가지고 있습니다. 
예를 들어, 앱으로 택시를 콜할 때 현재 나의 지점에서 반경 Xkm(몇 분 이내에 도착할 수 있는)까지 있는 택시 기사들 중 한명과 매칭된다는 것과 비슷한 개념입니다.

<img src="https://i.ibb.co/PxpmyT5/matching-system8.png" alt="matching-system8" width="850" />

---

## 🌱 현재 구현되어 있는 Flow

구현되어 있는 코드는 다음과 같은 Flow를 가집니다.

1. 위치 정보를 제공하는 회원들이 있으며, 이들의 위치는 Redis에 Geo API를 통해 갱신되고 있습니다.
2. 매칭을 시도하는 Member가 있습니다. 현재 Member의 위치를 기준으로 반경 xkm의 회원들을 모두 조회합니다.
3. 2번에서 조회한 매칭될 회원들 중, 이미 매칭되어 있는 회원들을 제외한 다른 회원들 중 한명과 매칭이 되어야 합니다.
4. 매칭에 성공하면 Routing Table에 자신의 회원 아이디와 매칭된 회원 아이디를 저장합니다.

현재 이러한 Flow에서 주의해야할 점은 3번 사항입니다.

위치 정보를 제공하고 있는 회원이 이미 다른 회원과 매칭이 되어 있는 상황이라면 그 회원이 매칭되어 있는 회원이라는 것을 어딘가에 저장해야합니다. 이를 해결하기 위해 
Redis를 활용한 분산락 개념을 응용하려합니다.

...

#### ❗️ MySQL DB로는 해결할 수 없을까?
현재 회원 정보를 MySQL Database에 저장하고 있기 때문에 회원 정보를 MySQL에 저장하고 조회하면 괜찮지 않을까하는 고민이 생겨납니다. 당연히 해결할 수는 있지만 구현이 복잡해지고 Disk DB의 특성상 
빠르게 처리되지 않는다는 단점이 있습니다.

MySQL로 해당 문제를 해결하기 위해서는 Lock이라는 개념을 활용해야합니다. 다양한 lock이 존재하지만 Record lock이라는 개념을 활용하여 정보를 수정할 때 다른 insert, update, delete 같은 쿼리들이 수행되지 않고 
lock이 해제될 때까지 대기하게 됩니다.

Java/Spring 서버는 병렬적으로 Request를 처리하기 때문에 몇십명의 회원들이 동시에 같은 주변의 회원들을 조회할 수 있습니다. 회원 한명 한명 이미 매칭되어 있는지 조회하고 매칭되어 있지 않다면 매칭 상태로 변경하는 과정에서 
다른 쿼리 요청이 수행될 수 있다는 위험을 가지고 있습니다.

<img src="https://i.ibb.co/Db8gX7k/matching-system9.png" alt="matching-system9" width="850" />

이 부분은 MySQL의 lock과 트랜젝션을 스터디하면 더욱 자세하게 알 수 있습니다.

##### 참고 레퍼런스 - [트랜잭션과 잠금](https://hoing.io/archives/4182)

-----

### 🟤 Redis로 해결하기

MySQL로 처리하기 어려운 이유는 2가지입니다.

1. 조회 및 상태 변경 쿼리가 Atomic하지 않는다는 점
2. 쿼리 요청이 병렬적으로 처리되기 때문에 1)조회하고 2)상태를 변경하는 이 과정 중에 다른 트랜잭션이 끼어들 수 있다는 점

즉 이 문제를 해결할 수 있는 방법은 병렬적으로 처리되지 않고 Atomic(원자적)하게 처리하는 방법입니다.

...

Redis는 싱글 스레드 기반으로 동작하는 툴입니다. 여러 요청이 들어오게 되면 병렬적으로 처리하지 않고 모든 요청을 동기적으로 처리합니다. 또한 여러 처리를 하나의 단위로 처리하는 Atomic한 API도 지원합니다. 
이것을 통해 MySQL의 락, 트랜잭션 메커니즘보다 더욱 효율적이고 안정적으로 구현할 수 있습니다.

<img src="https://i.ibb.co/QQBcWzN/matching-system10.png" alt="matching-system10" width="850" />

----

## 🌱 코드로 확인하기

분산락을 구현한 코드를 확인합니다.

```java
 private Long dispatchMember(List<Long> memberIds) {
     for (Long memberId : memberIds) {

         Boolean check = redisDistributedLockTemplate.opsForValue()
             .setIfAbsent(memberId.toString(), memberId.toString());

         if (Boolean.TRUE.equals(check)) {
             return memberId;
         }
     }

     throw new RuntimeException("매칭된 회원이 존재하지 않습니다.");
 }
```

1. 파라미터를 통해 주변에 있는 회원들의 ID를 조회합니다.
2. 각 회원들 아이디가 분산락으로 사용하는 Redis에 `setIfAbsent`로 조회 및 저장합니다.
   - setIfAbsent는 원자적으로 동작하는 메소드입니다. 만일 이미 회원 아이디가 있다면 이미 매칭된 회원이며, 회원 아이디가 존재하지 않다면 바로 해당 아이디를 분산락에 저장합니다.
   - 저장하게 되면 true를 이미 아이디가 있어서 저장하지 않았다면 false를 반환합니다. true면 회원 조회를 멈추고 매칭이 성사되어 값을 반환하고 false면 조회한 아이디를 모두 조회합니다.
3. 매칭 성사된 회원 아이디를 Routing Table에 저장합니다.

------
