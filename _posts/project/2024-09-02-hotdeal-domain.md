---
layout: post
title: "[Project] Spring에서의 DTO와 VO 전략"
subtitle: "DTO와 VO 개념을 활용하여 클래스를 재활용하기 위한 전략"
categories: devlog
tags: project
---

> DTO와 VO 개념을 활용하여 객체 인스턴스를 재활용하기 위한 전략

<!--more-->


#### 목차

- [여는 글](#-여는-글)
- [객체 지향적으로 풀어내기 위한 여정](#-객체-지향적으로-풀어내기-위한-여정)
  - [기획서 분석 및 ERD 설계](#-기획서-분석-및-erd-설계)
  - [요청으로 받을 데이터 분석](#-요청으로-받을-데이터-분석)
  - [Service Layer에서 사용될 도메인 객체를 고민해보기](#-service-layer에서-사용될-도메인-객체를-고민해보기)

---

## 🌱 여는 글

Spring을 활용한 서버 어플리케이션은 크게 3가지의 Layer로 나눌 수 있습니다.

1. Presentation Layer
2. Service Layer
3. Persistence Layer

Presentation Layer는 서버 어플리케이션과 외부와의 통신을 담당하는 Layer입니다. 클라이언트에서 요청한 값을 서버에서 처리하여 반환할 수 있도록 열어둔 창구같은 개념이며, 서버끼리 내부적으로 통신할 때도 사용합니다. (MSA 등)

Service Layer는 외부의 요청한 값을 처리하기 위한 비즈니스 로직을 담당하는 Layer이며, Persistence Layer는 DB 등 여러 인프라 요소들과 상호작용을 하기 위한 Layer입니다.

...

그런데 각 레이어마다 요청을 처리하기 위해 필요한 객체는 조금씩 다를 수 있습니다.

예를 들어, 처음 서비스에 가입한 회원은 클라이언트로부터 굳이 회원 등급까지 필드값으로 받을 필요 없이 서버 내부에서 기본값으로 처리하여 저장할 수 있습니다.
이 때, 회원가입을 위한 Request 객체는 회원등급 필드가 필요 없지만, 결국 마지막 persistence Layer에서 DB에 저장할 때는 회원 객체에 회원 등급 필드가 필요한 경우가 그 예시입니다.

...

Hot Deal은 하나의 도메인이 여러 형태의 객체로 표현될 수 있다느 것을 인정하고 이것을 어떻게 객체 지향적으로 풀어낼 수 있을지 고민한 프로젝트입니다.

해당 프로젝트에서 사용된 도메인인 Member, Item, Order 등을 통해 위와 같은 고민을 풀어낸 사례를 소개합니다.

---

## 🌱 객체 지향적으로 풀어내기 위한 여정

위의 고민을 해결하기 위해서 다음과 같은 순서로 접근하였습니다.

1. 기획서 분석
2. ERD 설계
3. 요청으로 받을 데이터 분석
4. Service Layer에서 사용될 도메인 객체를 고민해보기
5. 필요한 기능을 구현하기 위한 로직을 파악 및 구현


### 🥕 기획서 분석 및 ERD 설계

기획서 분석은 모든 기능 구현의 시작점입니다. 상품 구매를 위한 서비스에서 각 회원의 주소를 영구적으로 저장하여 주문 시에 지속적으로 사용하고자 한다면 회원 도메인 객체에는 주소 정보를 담을 필드가 필요합니다. 하지만 굳이 회원의 학력이 필요하지는 않습니다.

이러한 관점에서 기획에서 필요한 요구사항을 분석하고 필요한 정보들을 추려 테이블을 설계하였습니다.

<img src="https://i.ibb.co/4WgjZRS/Screenshot-2024-09-02-at-18-44-58.png" alt="Screenshot-2024-09-02-at-18-44-58" border="0">

<br />
<br />

Hot Deal는 JPA를 사용하기 때문에 위의 테이블을 바탕으로 Entity를 설계하였습니다. Persistence Layer에서는 기본적으로 위의 Entity를 통해 회원 정보를 저장, 조회, 수정합니다.

```java
@Getter
@Entity
@Table(name = "item")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class ItemEntity extends BaseEntity {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "id", columnDefinition = "bigint", nullable = false)
	private Long id;

	@Column(name = "uuid", columnDefinition = "varchar(60)", nullable = false)
	private String uuid;

	@Column(name = "name", columnDefinition = "varchar(60)", nullable = false)
	private String name;

	@Column(name = "price", columnDefinition = "int", nullable = false)
	private Integer price;

	@Column(name = "discount", columnDefinition = "int", nullable = false)
	private Integer discount;

	@Column(name = "quantity", columnDefinition = "int", nullable = false)
	private Integer quantity;

	@Column(name = "introduction", columnDefinition = "varchar(255)", nullable = false)
	private String introduction;

	@Enumerated(EnumType.STRING)
	@Column(name = "type", columnDefinition = "varchar(60)", nullable = false)
	private ItemType type;

	@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss.SSS", timezone = "Asia/Seoul")
	@Column(name = "pre_order_time", columnDefinition = "timestamp", nullable = false)
	private LocalDateTime preOrderTime;

	@Enumerated(EnumType.STRING)
	@Column(name = "status", columnDefinition = "varchar(30)", nullable = false)
	private ItemStatus status;

	@Column(name = "category_id", columnDefinition = "bigint", nullable = false)
	private Long categoryId;

	@Column(name = "member_id", columnDefinition = "varchar(60)", nullable = false)
	private String memberId;

	//...
}
```

### 🥕 요청으로 받을 데이터 분석

상품 테이블은 위와 같이 설계하였습니다. 하지만, 상품을 등록할 시 클라이언트로부터 받을 회원 정보들은 위의 엔티티의 필드와 약간 다릅니다. 기획서를 참고하면,

- 상품마다 고유의 회원 아이디(UUID)가 존재해야한다.
- 클라이언트로 예약 판매 여부 값을 받고, 예약 판매 시 `년, 월, 일, 시간, 분`을 개별적으로 입력받아야한다. 

아리헌 요구사항이 있습니다. 이에 따른 회원을 저장하기 위한 Request 객체는 다음과 같습니다.

```java
@Getter
public class AddItemRequest {

	@NotBlank(message = "이름은 빈칸을 허용하지 않습니다.")
	private String name;

	@JsonProperty("cost")
	@NotNull(message = "금액 정보는 필수입니다.")
	private CostRequest costRequest;

	@NotBlank(message = "상품 설명은 빈칸을 허용하지 않습니다.")
	private String introduction;

	@NotNull(message = "상품 수량은 필수 값입니다. (일반 구매 상품은 0으로 입력합니다.)")
	@Min(value = 0, message = "상품 타입이 예약 구매 시 0이상의 값을 입력해야합니다.")
	private Integer quantity;

	@NotNull(message = "상품 타입은 none, pre_order, normal_order 중 한가지를 택해야합니다.")
	private ItemTypeRequest type;

	@JsonProperty("pre_order_schedule")
	private PreOrderScheduleRequest preOrderSchedule;

	@JsonProperty("category_id")
	@NotNull(message = "카테고리 아이디는 빈칸을 허용하지 않습니다.")
	private Long categoryId;

	//...
}

@Getter
public class CostRequest {

	@NotNull(message = "가격은 필수 값입니다.")
	@Min(value = 0, message = "가격은 0이하가 될 수 없습니다.")
	private Integer price;

	@NotNull(message = "할인 금액은 필수 값입니다.")
	@Min(value = 0, message = "할인 금액은 0이하가 될 수 없습니다.")
	private Integer discount;
	
	//...
}

@Getter
public class PreOrderScheduleRequest {

	@Min(value = 2020, message = "예약 스케줄 연도는 2020 미만이 될 수 없습니다.")
	@Max(value = 2100, message = "예약 스케줄 연도는 2100 초과가 될 수 없습니다.")
	private Integer year;

	@Min(value = 1, message = "예약 스케줄 월은 1 미만이 될 수 없습니다.")
	@Max(value = 12, message = "예약 스케줄 월은 12 초과가 될 수 없습니다.")
	private Integer month;

	@Min(value = 0, message = "예약 스케줄 일은 1 미만이 될 수 없습니다.")
	@Max(value = 31, message = "예약 스케줄 일은 31 초과가 될 수 없습니다.")
	private Integer date;

	@Min(value = 0, message = "예약 스케줄 시간은 0 미만이 될 수 없습니다.")
	@Max(value = 23, message = "예약 스케줄 시간은 24 초과가 될 수 없습니다.")
	private Integer hour;

	@Min(value = 0, message = "예약 스케줄 분은 0 미만이 될 수 없습니다.")
	@Max(value = 59, message = "예약 스케줄 분은 60 초과가 될 수 없습니다.")
	private Integer minute;
	
    //...
}
```

위의 Request 객체는 회원 엔티티와 조금 다르다는 것을 알 수 있습니다.

- id, uuid는 클라이언트로부터 받지 않는다.
- Entity는 하나의 LocalDateTime으로 예약 판매 시간을 저장하지만, 클리이언트로부터 년, 월, 일, 시간, 분을 개별적으로 입력받는다.

이 정도의 차이점을 볼 수 있습니다.

Presentation Layer에서 Persistence Layer로 비즈니스 로직을 걸쳐 저장되기 위해서는 Service Layer에서 사용할 Item DTO를 생성하여 처리해야하지만, `상품 등록만을 위한 DTO`는 객체 지향 프로그래밍을 지원하는 Java를 
사용하는 이점이 없다고 판단하였습니다.

---

### 🥕 Service Layer에서 사용될 도메인 객체를 고민해보기

기획서에 따라 상품 도메인은 크게 3가지로 사용될 수 있습니다.

- 상품 등록
- 상품 리스트 조회
- 상품 세부 조회

위의 3가지 기능은 모두 상품 데이터를 조회가 필요하지만, 비즈니스 로직이나 클라이언트에게 반환할 데이터가 조금씩 다릅니다. 이 때, 반환할 데이터가 다르기 때문에 모든 요청마다 처리할 DTO를 따로 만들기보다, 각 DTO안에 들어갈 공통 필드를 VO로 만들어 
객체지향적으로 풀어나가는 것으로 고민을 해결하고자 하였습니다.

<img src="https://i.ibb.co/hmG7gkw/9-C0-BB25-C-1752-44-D4-8-FDA-2-D664-A622-C83.jpg" alt="9-C0-BB25-C-1752-44-D4-8-FDA-2-D664-A622-C83" border="0">

> 📚 DTO와 VO의 개념
> 
> DTO : 순수하게 데이터를 담아 계층 간으로 전달하는 객체
> 로직을 갖고 있지 않은 순수한 데이터 객체. Getter, Setter만을 갖는다.
> 
> VO : 값 그 자체를 표현하는 객체
> VO는 로직을 포함할 수 있으며, 특정 값 자체를 표현하기 때문에 불변성의 보장을 위해 생성자를 사용하여야 한다.

상품 등록, 상품 리스트 조회, 상품 세부 조회에서는 모두 상품 도메인을 활용합니다. 하지만 위의 그림처럼 각 요청마다 요구하는 필드값은 조금씩 다를 수 있기 때문에 내부 필드의 일관성 및 재사용성을 고민하였고, VO의 개념을 활용하여 각 DTO 객체의 필드로 
초기화하였습니다.

```java
@Getter
public class AddItemServiceDto {

	private ItemId itemId;
	private String name;

	private Cost cost;

	private String introduction;
	private Integer quantity;
	private ItemType type;

	private PreOrderSchedule preOrderSchedule;

	private ItemStatus status;
	private String memberId;
	private Long categoryId;

	//...
}

@Getter
public class RetrieveItemsDto {

	private final Long itemId;
	private final String itemName;
	
	private final Cost cost;
	
	private final Boolean isPreOrderItem;
	private final Integer quantity;
	private final LocalDateTime preOrderTime;
	private final String sellerId;
	private final String sellerName;
	
	//...
}

@Getter
public class ItemDetailDto {

	private Long itemId;
	private String itemName;

	private Cost cost;

	private String introduction;
	private Boolean isPreOrderItem;
	private LocalDateTime preOrderTime;
	private LocalDateTime createdAt;
	private String sellerName;
	private String sellerId;

	//...
}

//--------------------------------------------

@Getter
public class Cost {

	private final Integer price;
	private final Integer discount;

	@Builder
	public Cost(Integer price, Integer discount) {
		validateCost(price, discount);

		this.price = price;
		this.discount = discount;
	}

	public static Cost of(Integer price, Integer discount) {
		return new Cost(price, discount);
	}

	private void validateCost(Integer price, Integer discount) {
		if (discount > price || price < 0 || discount < 0) {
			throw new IllegalArgumentException("Invalid cost");
		}
	}
}

@Getter
public class PreOrderSchedule {

	private Integer year;
	private Integer month;
	private Integer date;
	private Integer hour;
	private Integer minute;

	@Builder
	private PreOrderSchedule(Integer year, Integer month, Integer date, Integer hour, Integer minute) {
		this.year = year;
		this.month = month;
		this.date = date;
		this.hour = hour;
		this.minute = minute;
	}

	public static PreOrderSchedule of(Integer year, Integer month, Integer date, Integer hour, Integer minute) {
		return PreOrderSchedule.builder()
			.year(year)
			.month(month)
			.date(date)
			.hour(hour)
			.minute(minute)
			.build();
	}

	public boolean isBetweenStartTimeAndEndTime(LocalDateTime startTime, LocalDate endTime) {
		LocalDateTime inputTime = LocalDateTime.of(year, month, date, hour, minute);

		return startTime.isBefore(inputTime) && endTime.isAfter(inputTime.toLocalDate());
	}

	public LocalDateTime toLocalDateTime() {
		return LocalDateTime.of(year, month, date, hour, minute);
	}
}
```

AddItemServiceDto, RetrieveItemsDto, ItemDetailDto 는 Item 도메인을 활용하지만 조금씩 요구하는 필드 값이 다르기 때문에 Cost, PreOrderSchedule 같은 VO 객체를 두어 
해당 객체에서 공통된 도메인 로직을 처리할 수 있도록 설계하였습니다.

------