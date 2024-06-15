---
layout: post
title: "[Troubleshooing] JPA 변경감지 vs QueryDSL : 회원 정보 수정 최적화 방안"
subtitle: "Tosstock 프로젝트에 적용된 회원 정보 수정 최적화 방안에 대해 소개합니다."
categories: devlog
tags: troubleshooting jpa querydsl
---

> QueryDSL을 사용하여 member update 기능을 개선시킨 사례를 소개합니다.

<!--more-->

- 목차

## 🌱 문제 인식

Tosstock 프로젝트는 JPA에서 DB에 저장되어 있는 데이터를 변경하는 방법 중 하나인 '변경 감지'를 통해 회원의 정보를 변경하고 있었습니다.

이 방법은 간단하게 조회한 엔티티의 필드값을 바꿔주면 이후 해당 로직의 트랜젝션이 끝나고 커밋이 이루어질 때 변경된 필드를 감지하고 update하는 편리함을 제공합니다.

하지만 필드를 변경하기 위해 무조건적으로 select문으로 쿼리 요청을 하는 방식이 비효율적이지 않을까 판단했습니다.

- <strong>회원은 로그인 한 이후 자신의 개인정보를 알고 있는 상황에서 update를 위한 select문을 요청하는 것은 불필요</strong>
- <strong>AWS RDS를 사용하고 있기 때문에 네트워크 I/O를 줄이는 것은 비용절감에 기여할 수 있을 것</strong>

select문 하나 요청하는 것이 큰 리소스를 차지하지 않다고 생각할 수 있지만, update 쿼리 하나만 요청하게 된다면 수치적으로 약 50%의 성능 개선이라고 감히 표현할 수 있을 것 같습니다.

----

## 🌱 트러블슈팅

### 🥕 기존 코드
```java
@Service
@RequiredArgsConstructor
public class MemberService {
	
    public final MemberRepository memberRepository;
    
    @Transactional
    public boolean changeMemberInfo(Long memberId, UpdateMemberDto updateMemberDto) {
        // select query 발생
        memberRepository.findById(memberId)
        
        // update query 발생
            .ifPresent(s -> s.set...(updateMemberDto...));
        
        return true;
    }
}
```
- 조회한 엔티티의 필드값을 변경하면 transaction이 커밋될 시점에 update 쿼리를 요청하는 매커니즘으로 동작

#### 🥕 실행 결과

<a href="https://ibb.co/93fTMJt"><img src="https://i.ibb.co/0VWMwkt/Screenshot-2024-06-15-at-14-08-40.png" alt="Screenshot-2024-06-15-at-14-08-40" border="0"></a>

---

### 🌱 개선 방안

JPA의 변경감지 대신 update를 할 수 있는 방법 중 QueryDSL을 통해 update 쿼리요청하도록 수정하였습니다.

> QueryDSL은 JDBC, JPQL, Native Query 등의 복잡한 로직을 개선합니다.
> 
> 또한 개발자의 쿼리문 실수로 인한 런타임 에러가 발생할 여부를 Java 코드로 쿼리문을 작성하기 때문에 컴파일 에러로 사전에 방지합니다.

Tosstock 프로젝트의 회원이 개인정보를 수정할 때 다음과 같은 룰이 존재합니다.

- 이메일과 전화번호는 변경 불가
- 이름은 빈칸을 허용X (NotBlank)
- 자기 소개는 null을 허용X (NotNull)
- 프로필 사진 URL은 null을 허용

1차적으로는 Controller의 Validation을 통해 각 필드마다 유효성 검사를 진행합니다. 유효성 검사 이후 DTO의 필드 중 null인 데이터는 update에 반영하지 않고(프로필 이미지는 예외) 
필드에 존재하는 값만 update에 반영하기 위해 동적으로 쿼리를 작성해야합니다.

QueryDSL은 이러한 상황에서 보다 쉽게 쿼리를 작성할 수 있다는 장점을 가지고 있습니다.


#### 🥕 개선된 코드
```java
@RequiredArgsConstructor
public class MemberRepositoryImpl {

    private final JPAQueryFactory queryFactory;

    @Transactional
    @Modifying(clearAutomatically = true)
    @Override
    public void updateInfo(Long memberId, UpdateMemberDto updateMemberDto) {
        JPAUpdateClause update = queryFactory.update(memberEntity);
        
        setNotNullFields(update, memberEntity.username, updateMemberDto.getUsername());
        setNotNullFields(update, memberEntity.introduce, updateMemberDto.getIntroduce());
        
        update
            .set(memberEntity.profileImageUrl, updateMemberDto.getProfileImageUrl())
            .where(memberEntity.id.eq(memberId))
            .execute();
    }

    private void setNotNullFields(JPAUpdateClause update, StringPath field, String value) {
        if (value != null) {
            update.set(field, value);
        }
    }
}
```

- queryFactory.update(memberEntity)를 통해 어떤 엔티티를 업데이트할 것인지 초기화합니다.
- setNotNullFields 메서드를 만들어 null이 아닌 필드를 set합니다. (프로필 이미지 URL은 예외)
- 어떤 회원 아이디를 가진 엔티티를 update할 것인지 where을 통해 선언합니다.

#### 🥕 실행 결과

<a href="https://ibb.co/6BCzPdx"><img src="https://i.ibb.co/nCJ9cWv/Screenshot-2024-06-15-at-14-37-52.png" alt="Screenshot-2024-06-15-at-14-37-52" border="0"></a>

---

> 💡 어노테이션 Modifying(clearAutomatically = true)는?
> 
> QueryDSL과 JDBC, JPQL 등의 기술은 JPA를 사용하지 않고 DB에 쿼리문을 요청합니다. 즉, JPA의 영속성 컨텍스트에 수정된 필드가 반영되지 않고 쿼리가 요청되기 때문에 
> 하나의 트랜잭션 안에서 update 이후 update된 필드를 다시 불러온다면 수정되지 않은 필드값을 얻게 됩니다.
> 
> 이러한 상황을 미리 방지하고자, update 쿼리가 완료되면 영속성 컨텍스트에 있는 해당 엔티티의 정보를 flush합니다. 이는 update 이후에 엔티티 필드에 접근하고자 하면 비어있는 영속성 
> 컨텍스트로 인해 다시 select문으로 해당 엔티티를 조회하여 값을 가지고 올 수 있게 합니다.
> 
> 즉, 서버 안에서 데이터의 정합성을 위해 설정한 어노테이션입니다.

---

## 🌱 결론

JPA의 변경감지의 필요성에 대해 고민해보았을 때 '회원 정보 수정'이라는 단발성 수정 요청이 아닌 보다 복잡한 요청이 들어왔을 경우에는 유용하게 사용할 수 있을거라 생각합니다. (ex. 하나의 API로 많은 엔티티의 데이터들이 수정되어야 하는 경우)

회원 정보를 업데이트하고 난 이후 추가적으로 업데이트된 필드 값을 사용하는 비즈니스 로직이 있다면 JPA의 변경감지를 통해 업데이트하는 것이 효율적이지 않았을까 생각합니다.

이렇듯 다양한 기술들을 사용할 때는 '무조건 옳다'는 기능은 없는 것 같습니다. 여러 상황에 맞게 필요한 기술을 선택하고 적용해서 최적의 답을 얻어내는 스킬이 개발자에게 필요하지 않을까 생각해봅니다.