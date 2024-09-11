---
layout: post
title: "[Project] 상품 주문 쿼리 최적화한 사례"
subtitle: "where in / bulk insert를 활용하여 쿼리 성능을 개선한 사례를 소개합니다."
categories: devlog
tags: project
---

> where in / bulk insert를 활용하여 쿼리 성능을 개선한 사례를 소개합니다.

<!--more-->

#### 목차

- [여는 글](#-여는-글)

---

## 🌱 여는 글

Hot-Deal의 일반 상품 주문 기능은 여러 상품을 한번에 주문할 수 있습니다. 실시간으로 재고를 처리해야했던 [예약 상품 주문](https://syeon2.github.io/devlog/hotdeal-order-speed.html) 기능과 다르게 일반 상품은 상품 판매자(관리자)가 
직접 재고 상태를 파악하여 주문을 확정/취소 할 수 있어, 재고 처리에 대한 이슈는 발생하지 않습니다.

다만, 여러 개의 상품을 한번에 주문하는 기능이니만큼 추가로 최적화해야할 이슈는 발생하게 되는데 바로 `쿼리 최적화`입니다.

기획상 목표 DAU는 최대 10,000명이며, 1시간에 약 400명의 유저가 방문하는 것입니다. 기존의 성능으로도 충분히 커버는 가능했지만, 유저 유입이 높아지고 다양한 쿼리를 처리할 때 DB가 받는 부하를 
줄이기 위한 개선은 필요하다고 판단하여 트러블슈팅을 진행하였습니다.

---

## 🌱 주문 쿼리 최적화 트러블슈팅

### 🥕 문제 파악

현재 주문하기 기능은 다음과 같은 순서로 동작합니다.

1. 주문할 상품 리스트 조회
2. 상품 가격과 할인 금액 검증
3. 각 주문 상품들에 대한 세부 내역 DTO 생성
4. 3번에서 만들어진 세부 내역 정보들을 저장
5. 주문 데이터 저장

이와 같은 순서로 동작하는 것은 현재 구현되어 있는 주문, 주문 상세 테이블이 나뉘어 저장되고 관리되기 때문입니다.

<img src="https://i.ibb.co/z5Dr9K2/hotdeal-order.png" alt="hotdeal-order" border="0">

주문 테이블은 하나의 주문에 대한 정볼르, 주문 리스트 테이블은 주문에 맵핑되는 각 상품들의 세부 정보를 관리합니다. 테이블 구성으로 인해 주문하기 기능을 사용할 때마다 `주문한 상품 정보 조회(N) + 1(주문 정보) + N(주문 리스트 - 각 상품 정보)개`, 총 2N + 1개의 쿼리들이 요청되어집니다.

이 문제는 작동하는데는 아무 이상 없지만, 다음과 같은 문제를 야기할 수 있습니다.


<br />

#### ❗️ 데이터베이스 커넥션 풀 고갈

Spring에서는 Database Connection Pool이란 자원을 가지고 있습니다. 이것은 서버 어플리케이션에서 데이터베이스에 접근할 때마다 많은 네트워크 비용이 발생하는 3 handshake 등의 절차를 간소화하기 위해 초기 지정된 개수만큼 미리 커넥션을 맺고 연결을 유지하는 기능입니다.

이 기능은 많은 요청으로 인해 생성되는 커넥션으로 인해 OOME가 발생하는 것을 막아주는 등 부가적인 이슈도 방지할 수 있습니다.

쿼리가 요청되는 것은 바로 이 커넥션 풀에 생성되어 있는 커넥션을 사용하는 것입니다. 만일, 커넥션 풀에서 커넥션을 사용하고 반환하는 시간보다 처리해야하는 쿼리 수가 더 많아진다면 병목점이 될 수 있고, 심지어 설정값에 따라 쿼리 요청이 취소될 수도 있습니다.

> 현재의 문제는 하나의 주문에 많은 쿼리들이 생성되어 전체적은 처리 속도는 물론이거니와 커넥션 풀을 고갈시킬 가능성을 가지고 있습니다.
> 
> 이에 대한 최적화가 꼭 필요하다고 판단하여 트러블슈팅을 진행하였습니다.

---

### 🥕 원인 분석

트러블슈팅 전 Hot Deal의 일반 상품 주문 기능은 아래와 같은 코드로 구현되어 있었습니다.

```java
@Transactional
public String addNormalOrderWithOrderDetail(
    String memberId,
    AddAddressDto addAddressDto,
    List<AddOrderItemDto> orderItems
) {
    List<OrderDetailDto> orderDetails = createOrderDetails(orderItems);

    OrderEntity savedOrderEntity = orderRepository.save(
        orderMapper.toEntity(initializeOrderDto(memberId, addAddressDto))
    );

    orderDetails.forEach(orderDetail -> {
        savedOrderEntity.addOrderDetail(orderDetailMapper.toEntity(orderDetail));
    });

    return savedOrderEntity.getUuid();
}

//-----------


private List<OrderDetailDto> createOrderDetails(List<AddOrderItemDto> orderItems) {
  Map<Long, OrderDetailDto> orderDetails = new HashMap<>();

  for (AddOrderItemDto orderItem : orderItems) {
    ItemEntity itemEntity = itemRepository.findById(orderItem.getItemId())
            .orElseThrow(() -> new IllegalArgumentException("잘못된 상품 아이디를 압력하였습니다."));

    //...

    OrderDetailDto orderDetail = OrderDetailDto.from(
            orderItem,
            itemEntity.getPrice(),
            itemEntity.getQuantity()
    );

    orderDetails.putIfAbsent(itemEntity.getId(), orderDetail);
  }

  return orderDetails.values().stream()
          .toList();
}

//-------------

@Entity
@Table(name = "orders")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class OrderEntity extends BaseEntity {
	
    @OneToMany(
        mappedBy = "order",
        cascade = CascadeType.ALL,
        orphanRemoval = true,
        fetch = FetchType.LAZY)
    private List<OrderDetailEntity> orderDetail = new ArrayList<>();
}
```

주문과 주문 세부 정보의 관계성에 따라 cascade, orphanRemoval를 설정하였고 이 기능으로 상품 세부 정보를 저장하는 로직입니다. 또한, 클라이언트로부터 결제액, 할인액의 조작의 가능성도 있어 클라이언트에서 전달받는 결제액, 할인액에 
의존하지 않고 DB에 저장되어 있는 상품 정보를 통해 정확한 처리를 하기 위해서 각 상품마다 정보를 조회하는 쿼리(createOrderDetails 메서드)도 포함되어 있습니다.

이 로직으로 실행한 주문하기 기능은 하나의 주문에 10개의 상품을 함께 주문하면 총 21개의 쿼리(상품 조회 쿼리 10개 + 주문 데이터 저장(1) + 주문 상세 데이터 저장(10))가 발생합니다.

###### ⚙️ 2개의 상품을 주문한 테스트
<img src="https://i.ibb.co/bdGw82N/order-query-issue1.png" alt="order-query-issue1" border="0">

---

### 🥕 쿼리 최적화를 통한 커넥션 풀 확보 전략

주문하는 상품이 많으면 많을수록 요청하는 쿼리 개수가 증가하기 때문에, 이 상황을 타개할 수 있는 방법은 `상품 조회 쿼리`와 `상품 세부 정보 저장 쿼리`를 최적화하는 것이 필요하다고 판단하였습니다.

#### ❗️ Where In 절을 활용한 조회 쿼리 최적화

필요한 상품 데이터를 한번에 조회하는 방법은 Where In 절을 사용하는 것입니다.

다행히(?) Spring Data JPA에서는 where in을 통해 쿼리를 요청할 수 있도록 findAllById라는 메서드를 지원하고 있습니다.

```java
private List<OrderDetailDto> createOrderDetails(List<AddOrderItemDto> orderItems) {
    Map<Long, OrderDetailDto> orderDetails = new HashMap<>();
    
    List<Long> orderedItemIds = orderItems.stream()
        .map(AddOrderItemDto::getItemId)
        .toList();
    
    Map<Long, ItemEntity> itemMap = itemRepository.findAllById(orderedItemIds).stream()
        .collect(Collectors.toMap(ItemEntity::getId, itemEntity -> itemEntity));
    
    for (AddOrderItemDto orderItem : orderItems) {
        ItemEntity findItem = itemMap.get(orderItem.getItemId());
    
        //...
    
        orderDetails.putIfAbsent(findItem.getId(), orderDetail);
    }
    
    return orderDetails.values().stream().toList();
}
```

###### ⚙️ 쿼리 로그를 확인하면 추가된 where in을 볼 수 있습니다.
<img src="https://i.ibb.co/w4XdYk1/order-query-issue2.png" alt="order-query-issue2" border="0">

---

#### ❗️ Bulk Insert를 활용한 저장 쿼리 최적화

JPA는 batch size 옵션을 조절함으로 커넥션이 한번에 처리할 수 있는 쿼리의 양을 설정할 수 있습니다.

```yaml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 100
```

batch size를 설정하는 것은 서버 어플리케이션 측면에서는 하나의 커넥션으로 여러 쿼리들을 처리할 수 있기 때문에 성능이 최적화될 수 있습니다. 하지만, 데이터베이스 입장에서는 각각 쿼리들이 개별적으로 처리되는 것은 변하지 않아 모니터링을 통한 
데이터베이스 부하 상태를 체크해야합니다.

또한, 가장 치명적인 문제가 있습니다. 바로 MySQL의 Id 생성 전략인 IDENTITY를 사용하고 있다면 해당 옵션은 무시됩니다.

#### 💡 MySQL Id 생성 전략

데이터베이스에 데이터를 저장할 때 `AUTO INCREMENT` 옵션이 있습니다. 이 옵션은 현재 테이블에서 마지막으로 저장된 레코드의 id값을 기준으로 새로운 id를 `자동으로` 생성하여 저장하는 방식입니다.

그 중, MySQL은 IDENTITY 방식으로 새로운 Id값을 생성합니다. Identity는 DB 메모리에 현재 생성된 마지막 아이디값을 저장하고, 데이터가 생성될 때마다 갱신하며 Id를 부여합니다.

즉, 서버에서 하나의 트랜잭션이 끝나기도 전에 insert문의 쿼리가 DB에 전달되고 처리되는 매커니즘을 가지고 있습니다. 이 매커니즘이 batch size 옵션을 무시하고 동작하게 되는 원인을 제공합니다.

<br />...

안타깝게도(?) JPA 자체에서 Identity 설정에 대비한 bulk insert는 지원하고 있지 않습니다. 그렇기 때문에 해당 프로젝트에서는 JdbcTemplate를 활용한 bulk insert를 적용하였습니다.

```java
@Repository
@RequiredArgsConstructor
public class OrderDetailRepository {

    private final JdbcTemplate jdbcTemplate;
    
    public void saveAll(List<OrderDetailDto> orderDetails, String orderId) {    
        String sql = "INSERT INTO hotdeal.order_detail "
            + "(quantity, total_discount, total_price, item_id, order_id, created_at, updated_at) "
            + "VALUES (?, ?, ?, ?, ?, now(), now())";
    
        jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                OrderDetailDto orderDetail = orderDetails.get(i);
    
                ps.setInt(1, orderDetail.getQuantity());
                ps.setInt(2, orderDetail.getTotalDiscount());
                ps.setInt(3, orderDetail.getTotalPrice());
                ps.setLong(4, orderDetail.getItemId());
                ps.setString(5, orderId);
            }
    
            @Override
            public int getBatchSize() {
                return orderDetails.size();
            }
        });
    }
}
```

위의 로직으로 처리하는 쿼리는 다음과 같이 변경되었습니다.

<img src="https://i.ibb.co/9tXZPXj/order-query-issue3.png" alt="order-query-issue3" border="0">

## 🌱 마무리

where in 절과 JdbcTemplate를 활용한 bulk insert를 사용하여 주문 쿼리를 최적화한 결과는 다음과 같습니다.

> 이전 : 2N + 1 (N은 상품 개수) 개의 쿼리
> 
> 이후 : => 3개 (상품 조회 where in 쿼리, 주문 insert 쿼리, 주문 세부 정보 bulk insert 쿼리)

이미 생성되어 있는 커넥션을 사용해서 그런지 단일 요청으로는 명확하게 성능이 개선되었는지 파악이 잘 되지 않았습니다. 그래서 Jmeter를 사용하여 초(s)단위로 테스트를 진행해보았습니다.

#### ⚙️ Jmeter 부하 테스트

- 묙표 Thread(유저 수): 500 per second
- 요청 총 시간: 10초 (1초에 약 500명의 요청)
- 저장되어 있는 상품 수량 : 약 30,000개
- 한번에 주문하는 상품 개수 : 10개
  - 개선 전 로직에서 발생하는 쿼리 개수 : 21개
  - 개선 후 로직에서 발생하는 쿼리 개수 : 3개

###### 💡 개선 전

<img src="https://i.ibb.co/Z60JBd0/order-query-test00.png" alt="order-query-test00" border="0">
<img src="https://i.ibb.co/cxX5FY3/order-query-test7.png" alt="order-query-test7" border="0">
<img src="https://i.ibb.co/ZJYhyVC/order-query-test8.png" alt="order-query-test8" border="0">

확인 결과 개선 전 주문 기능의 TPS는 약 150입니다. 이 상태 그대로 500 TPS에 맞춰 요청을 발생시키면 아래와 같은 결과를 볼 수 있습니다.

<img src="https://i.ibb.co/yy7bdPZ/order-query-test3.png" alt="order-query-test3" border="0">
<img src="https://i.ibb.co/8PnktsJ/order-query-test4.png" alt="order-query-test4" border="0">

목표한 5000개의 요청을 모두 처리하는 과정을 볼 수 있는 테스트 결과입니다. 에러율은 발생하지 않았지만 총 42초가 소요되었습니다.


###### 💡 개선 후

트러블 슈팅으로 개선된 코드를 통한 테스트 결과입니다.

<img src="https://i.ibb.co/KWZB0tm/order-query-test01.png" alt="order-query-test01" border="0">

<img src="https://i.ibb.co/PGsHGRN/order-query-test1.png" alt="order-query-test1" border="0">
<img src="https://i.ibb.co/t2zZBFt/order-query-test2.png" alt="order-query-test2" border="0">

개선된 코드는 TPS 500을 안정적으로 확보하였습니다. 단일 요청으로 볼 수 있는 테스트는 그 효과가 미비해보였지만, 부하테스트를 통한 성능 개선은 약 3.3배의 성능 개선을 이루었습니다.