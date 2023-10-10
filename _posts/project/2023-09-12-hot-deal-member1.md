---
layout: post
title: "[Hot-Deal #1] QueryDSL update 쿼리 인터페이스화"
subtitle: "QueryDSL을 활용하여 update 쿼리를 인터페이스화 합니다."
categories: project
tags: project querydsl
---

> QueryDSL을 활용하여 update 쿼리를 인터페이스화 합니다.

<!--more-->

-------

## 🌱 QueryDSL로 update 쿼리 인터페이스화

Hot-Deal 프로젝트는 DB에 쿼리를 요청하는 기술로 JPA를 기본으로 사용하는 프로젝트입니다. JPA의 특징은 이미 JpaRepository 인터페이스를 extends하여 
기본적으로 제공하는 API를 통해 쉽게 쿼리 요청할 수 있다는 장점을 가지고 있습니다. 하지만 JPA는 해당 Entity를 통해 엔티티 속성인 멤버 변수의 값을 변경해주기만 해도 
해당 엔티티의 레코드에 update 쿼리가 요청되는 편리함?을 가지고 있습니다.

이러한 편리함은 어떻게 update 쿼리가 요청되는지를 코드 레벨에서 바로 확인할 수 없으며, 해당 도메인의 Repository Layer에서 어떤 update 쿼리들을 사용하고 있는지 
알 수 없는 상황이 생길 수 있습니다.

update 쿼리를 QueryDSL로 관리하게 되면 update에 대한 다양한 구현체들을 설계할 수 있다는 장점을 생각해보았습니다. 
JPA를 사용하면서 인터페이스로 어떤 update 쿼리 요청 기능을 명세할 수 있도록 QueryDSL을 사용하였습니다.

<img src="https://i.ibb.co/VgyK6q5/hot-deal1-001.jpg" alt="hot-deal1-001" width="850" />

예를들어 JPA나 MyBatis같은 쿼리 요청하는 Repository 뿐만 아니라, 결제라면 다른 PG사들에게 요청하는 메소드 명세 또한 인터페이스로 설계할 수 있다는 장점을 가지고 있습니다. 

QueryDSL은 보통 동적쿼리, 복잡한 쿼리를 ORM 수준에서 보다 쉽게 작성할 수 있고 컴파일 단계에서 미리 쿼리의 문제를 파악할 수 있다는 장점을 가지고 있습니다. 하지만 단점으로는
QueryDSL은 일반적으로 쿼리가 수행되면 JPA의 특징인 영속성 컨텍스트에 해당 엔티티를 캐싱하는 특징을 무시하여 update 쿼리를 요청한 이후 select를 하게 되면 update 되지 않는 
엔티티를 조회할 수 있습니다. 이 점을 주의하여 QueryDSL를 통한 update 쿼리를 명세하고 구현하였습니다.


```java
public interface MemberCustomRepository {

	Long updateNickname(Long memberId, String nickname);

	Long updatePhone(Long memberId, String phone);
}
```

QueryDSL로 구현할 메소드들을 인터페이스에 명세합니다.

...

```java
@Repository
@RequiredArgsConstructor
public class MemberRepositoryImpl implements MemberCustomRepository {

	private final JPAQueryFactory jpaQueryFactory;
	private final QMember member = QMember.member;

	@Transactional
	@Modifying(clearAutomatically = true)
	@Override
	public Long updateNickname(Long memberId, String nickname) {
		return jpaQueryFactory.update(member)
			.set(member.nickname, nickname)
			.set(member.updatedAt, LocalDateTime.now())
			.where(member.id.eq(memberId)).execute();
	}

	@Transactional
	@Modifying(clearAutomatically = true)
	@Override
	public Long updatePhone(Long memberId, String phone) {
		return jpaQueryFactory.update(member)
			.set(member.phone, phone)
			.set(member.updatedAt, LocalDateTime.now())
			.where(member.id.eq(memberId)).execute();
	}
}
```

QueryDSL로 쿼리 요청 메소드를 작성하면서 주의깊게 보아야 할 부분은 `@Modifying(clearAutomatically = true)` 어노테이션입니다. 이 어노테이션은 
앞서 소개한 QueryDSL이 JPA의 영속성 컨텍스트에 캐싱하는 특징을 무시하여 데이터 정합성을 깨뜨릴 수 있는 여지를 개선합니다. 해당 어노테이션을 추가함으로 쿼리 요청 이후 
해당 엔티티를 영속성 컨텍스트에서 flush합니다. 해당 어노테이션과 옵션이 적용되어 있는 코드를 실행하고 다시 해당 엔티티를 조회하면 영속성 컨텍스트에서 값을 가지고 오는 것이 아니라 
DB에 select를 직접하여 새로운 값을 캐싱하고 가지고 옵니다.


#### 🥕 마무리

개인적으로 서버 개발은 서버 어플리케이션과 DB를 모두 신경써야하는 개발이라고 생각합니다. 그렇기 때문에 2가지를 모두 숙지하고 있어야하지만 사 내에서의 개발작업은 모든 사람들이 
처음부터 끝까지 함께하는 것이 아닐 것입니다. 도중에 더 좋은 회사로 이직하시는 분도 계실 것이고, 도중에 새로운 개발자분께서 투입될 수도 있습니다.

그렇기 때문에 코드는 최대한 명시적으로 구현하고자 하는 것을 보여주어야 한다고 생각합니다. DB의 각 column들의 타입을 DB까지 타고 들어가 확인하는 것이 아니라 
서버 어플리케이션에서 @Column(columnDefinition) 같은 어노테이션으로 최대한 보여줄 수 있는 것들을 보여주고, Service Layer에서 사용하고 있는 Repository Layer의 
메소드들을 쉽게 파악하기 위해 QueryDSL을 사용하여 명시적으로 update 쿼리를 인터페이스화하는 것이 위의 상황을 잘 보여줄 수 있는 방법이라고 생각됩니다.

함께 협업하는 과정에서 최대한 시너지를 낼 수 있도록 여러 방안들을 모색하는 것이 좋은 팀 관계를 형성하는데 도움이 되기에 이러한 협업의 방법들을 더욱 많이 
고민해볼 필요를 느낍니다.

------
