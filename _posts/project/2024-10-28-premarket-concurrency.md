---
layout: post
title: "[Project] 상품 주문 시 재고 차감 동시성 이슈 트러블슈팅"
subtitle: "Pessimistic Lock을 사용하여 동시성 이슈를 개선한 사례입니다."
categories: devlog
tags: project
---

> Pessimistic Lock을 사용하여 동시성 이슈를 개선한 사례입니다.

<!--more-->

#### 목차

- [여는 글](#-여는-글)
- [동시성 이슈 트러블 슈팅](#-동시성-이슈-트러블-슈팅)
  - [문제 파악](#-문제-파악)
  - [원인 분석](#-원인-분석)
  - [Pessimistic Lock을 활용하여 동시성 이슈를 해결](#-pessimistic-lock을-활용하여-동시성-이슈를-해결)
- [한계점](#-한계점)

---

## 🌱 여는 글

Pre market 서비스는 예약 구매 기능을 제공하여, 판매자가 원하는 시간에 상품을 구매할 수 있도록 설정할 수 있습니다. 설정한 시간이 되면 선착순으로 구매가 가능하며, 재고 설정과 함께 한 번에 하나의 예약 구매 상품만 선택하여 구매할 수 있습니다. 수량은 구매자의 재량에 따라 결정됩니다.

현재 Pre market 서비스의 DAU(일일 활성 사용자 수)는 2,000명 수준이지만, 인기 상품이 예약 구매 기능을 통해 제공될 경우 갑작스러운 트래픽 상승이 발생할 수 있습니다.

기획 상 DAU 2,000명은 시간당 평균 약 100명이 접속하는 수준이지만, 회원 수 증가와 예약 구매 기능의 중요성을 고려해 TPS(초당 처리량) 최소 500~1000을 안전하게 확보할 필요가 있습니다.

...

### ❗️ 상품 주문에 필요한 트러블슈팅 리스트

- 동시성 제어
- TPS 500 ~ 1000 확보


## 🌱 동시성 이슈 트러블 슈팅

### 🥕 문제 파악

예약 구매 기능은 `한정된 수량`을 정해진 시간에 `선착숙으로 구매`할 수 있는 서비스입니다. 즉, 여러 사용자가 동시에 하나의 상품 데이터에 접근하여 재고를 차감하는 것으로 
실시간으로 반영된 상품 재고량을 파악 및 Out of Stock같은 예외를 처리할 수 있습니다.

<img src="https://i.ibb.co/Ybb7TkN/order1.jpg" alt="order1" border="0">

현재는 위와 같은 매커니즘으로 예약 상품 주문이 동작합니다. 이 과정에서 상품 재고를 처리하는 과정에서는 다음과 같은 매커니즘으로 동작하게 됩니다.

1. 현재 상품 재고 조회
2. 현재 상품 재고와 회원이 주문한 수량 비교
   - 2번에서 (현재 상품 재고 - 주문한 수량 >= 0)일 경우 정상 동작
   - 2번에서 (현재 상품 재고 - 주문한 수량 < 0)일 경우 Out of Stock 예외 발생
3. 2번에서 정상 동작 이후 상품 주문 데이터 생성

위의 매커니즘을 반영하여 코드를 작성해보았습니다.

```java
@Transactional
public String addPreOrderItemWithOrderDetail(String memberId, AddAddressDto addAddressDto, AddOrderItemDto orderItem) {
    // 상품 재고를 조회
    ItemEntity itemEntity = validateItemForPreOrder(orderItem);
    
    // 상품 재고 차감
    itemEntity.deductQuantity(orderItem.getQuantity());
    
    // 상품 주문 생성 및 저장
    OrderEntity savedOrderEntity = orderRepository.save(
        orderMapper.toEntity(initializeOrderDto(memberId, addAddressDto))
    );
    
    //...
}

private ItemEntity validateItemForPreOrder(AddOrderItemDto orderItem) {
    ItemEntity itemEntity = itemRepository.findItemEntityForQuantityUpdate(orderItem.getItemId())
        .orElseThrow(() -> new IllegalArgumentException("잘못된 상품 아이디입니다."));
    
    if (itemEntity.getQuantity() - orderItem.getQuantity() < 0) {
        throw new OutOfStockException("재고가 부족합니다.");
    }
    
    return itemEntity;
}
```

보기에는 별 문제 없어보이는 코드입니다. 위의 코드를 Jmeter를 통해 동시에 요청해보았습니다.

#### ⚙️ 예약 주문 테스트

- 생성 Thread(유저 수): 2,000명
- 요청 총 시간: 10초 (1초에 약 200개의 요청)
- 초기 상품 수량 : 2,000개
- 예상 결과 : 10초 뒤 해당 상품의 수량 0

```json
// - Order Request Body
{
    "order_items": {
        "item_id": 1,
        "quantity": 1
    }
}
```

<img src="https://i.ibb.co/7XRkm6V/concurrency1.png" alt="concurrency1" border="0">

위의 결과를 보면 총 2,000개의 요청을 10초에 걸쳐 주문 API를 호출하였습니다. 1초에 약 200개의 요청을 전송하였습니다. 에러율도 0%로 2,000개의 요청이 
모두 성공적으로 주문을 생성했다는 것을 볼 수 있습니다.

하지만, 위의 코드는 반전의 결과를 가지고 옵니다.

<img src="https://i.ibb.co/m9nNfn2/concurrency2.png" alt="concurrency2" border="0">

예상한 결과 0이 아닌 555의 재고가 아직 남아있다는 것을 볼 수 있습니다. 즉, 2,000개의 요청 중 1,545개의 재고 차감 요청만 반영되고 나머지는 반영되지 않았습니다.

### 🥕 원인 분석

다시, 예약 주문 동작 순서를 생각해보겠습니다.

1. 현재 상품 재고 조회
2. 현재 상품 재고와 회원이 주문한 수량 비교
   - 2번에서 (현재 상품 재고 - 주문한 수량 >= 0)일 경우 정상 동작
   - 2번에서 (현재 상품 재고 - 주문한 수량 < 0)일 경우 Out of Stock 예외 발생
3. 2번에서 정상 동작 이후 상품 주문 데이터 생성

여기서 위와 같은 문제가 발생한 원인은 1번과 2번 과정을 처리하는 중에 발생할 수 있겠다는 것을 생각해볼 수 있습니다.

<img src="https://i.ibb.co/SJFM7Fs/concurrency3.jpg" alt="concurrency3" border="0">

User 1번이 Item의 재고를 조회하고 차감하기 전에 User 2번이 같은 Item의 재고를 조회하여 20이라는 값을 받아 처리하기 때문에 위와 같은 이슈가 발생한 것입니다.

이러한 동시성 이슈를 해결하기 위한 다양한 방법이 존재합니다.

1. Application Synchronized 예약어 사용 [🔗 Deep Dive](https://medium.com/@gsy4568/java-synchronized-deep-dive-9a764568d27c)
2. DB Lock (Pessimistic Lock, Optimistic Lock) [🔗 Deep Dive](https://medium.com/@gsy4568/pessimistic-locking-deep-dive-feat-mysql-7fcf90f259f0)

그 중에서 2번 DB Lock 개념을 활용하여 1차적으로 동시성 이슈를 해결해보고자 합니다.

> 1번 Application 레벨에서 메서드 자체에 lock을 거는 방법도 존재합니다.
> 
> `synchronized public String addPreOrderItemWithOrderDetail()...` 이 방법으로 Spring 서버에 들어오는 request(각각 개별적인 스레드로 동작)들을 
> 동기적으로 실행시킵니다.
> 
> 하지만, 서버를 다중화할 시 서버 어플리케이션이 여러 개로 나뉘기 때문에 동기적으로 실행할 목적으로 선언한 synchronized는 무용지물이 될 수 있습니다.

### 🥕 Pessimistic Lock을 활용하여 동시성 이슈를 해결

DB에서 제공하는 Lock은 동시성 이슈를 해결하는 방법 중 가장 간단합니다. 간단히 표현하자면, user 1번이 재고를 조회한 시점부터 재고를 업데이트할 때까지 다른 
트랜잭션이 해당 Item 데이터에 접근하지 못하게 `Lock`을 거는 방식으로 동작합니다.

DB를 통한 Lock 매커니즘은 크게 Pessimistic Lock, Optimistic Lock으로 나눌 수 있으며, 여기서는 상품 재고에 확정적으로 요청끼리 충돌하기 때문에 Pessimistic Lock을 
사용하여 문제를 해결하였습니다.

```java
ItemEntity itemEntity = itemRepository.findItemEntityForQuantityUpdate(orderItem.getItemId())
        .orElseThrow(() -> new IllegalArgumentException("잘못된 상품 아이디입니다."));


@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select i from ItemEntity i where i.id = :id")
Optional<ItemEntity> findItemEntityForQuantityUpdate(Long id);
```

JPA를 사용할 때 Pessimistic Lock을 사용하는 것은 생각보다 간단합니다. 상품 재고를 조회하는 Repository의 조회 메서드 위에 `@Lock(LockModeType.PESSIMISTIC_WRITE)`을 
추가함으로써 해당 메서드가 호출되는 시기로부터 다른 트랜잭션 요청이 접근할 수 없도록 합니다.

코드를 수정해서 다시 실행해보면 다음과 같인 정상적으로 처리된 결과를 얻을 수 있습니다.

<img src="https://i.ibb.co/5xNWZwm/concurrency-end1.png" alt="concurrency-end1" border="0">
<img src="https://i.ibb.co/RS8gsQG/concurrency-end2.png" alt="concurrency-end2" border="0">

똑같은 테스트를 진행해보았을 때 이제는 정상적으로 모든 요청에 맞춰 재고가 소진된 것을 확인할 수 있습니다.

## 🌱 한계점

Pessimistic Lock은 간편하게 적용할 수 있다는 이점이 있습니다. 이것으로 서버를 다중화시킬 상황이 발생하더라도 동시성 이슈를 DB 레벨에서 제어할 수 있게 되었습니다.

하지만 Lock을 사용함과 동시에 큰 한계점에 맞닥뜨리게 되었습니다.

### ❗️ 병목점 발생

Lock을 사용하는 것은 논블로킹으로 처리할 수 있었던 요청들을 블로킹으로 처리된다는 것을 의미합니다. 즉, 한꺼번에 처리할 수 있었던 요청을 순서대로 줄지어 차례대로 처리하도록 
변경하였기 때문에 속도의 측면에서 좋지 않은 퍼포먼스를 가지고 올 수 있습니다.

또한, 조금 더 근본적으로 디스크에 데이터를 저장하고 조회하는 MySQL의 속도는 초기에 목표한 TPS 500 ~ 1,000의 요구량을 충족시키기 어려웠습니다. (프로젝트를 가동시키고 있는 컴퓨터 기준)
다음 글에서는 어떻게 재고 차감 및 주문 생성 속도를 향상시킬 수 있었는지에 대한 사례를 소개합니다.