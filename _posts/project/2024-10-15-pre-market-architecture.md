---
layout: post
title: "[Project] Layered Architecture (근데 이제 DDD를 곁들인..)"
subtitle: "Layered Architecture와 도메인 주도 개발 방법을 적용한 프로젝트"
categories: devlog
tags: project layered-architecture ddd
---

> Pre Market 프로젝트의 소프트웨어 아키텍처인 Layered 아키텍처, DDD를 적용한 경험을 소개합니다.

<!--more-->

#### 목차

- [여는 글](#-여는-글)
- [Layered 아키텍처를 선택하게 된 배경](#-layered-아키텍처를-선택하게-된-배경)
  - [Layered Architecture란?](#-layered-architecture란)
  - [Layered Architecture의 각 계층](#-layered-architecture의-각-계층)
- [DDD (Domain-Driven Design)이란?](#-ddd-domain-driven-design이란)
  - [DDD의 핵심 개념](#-ddd의-핵심-개념)
- [Pre Market Architecture](#-pre-market-architecture)
  - [Presentation Layer](#-presentation-layer)
  - [Application Layer](#-application-layer)
  - [Domain Layer](#-domain-layer)
  - [Infrastructure Layer](#-infrastructure-layer)
- [닫는 글](#-닫는-글)
- [Reference](#-reference)

---

## 🌱 여는 글

Pre Market은 티켓 예매 같은 예약 구매를 주로 제공하는 프로젝트입니다.

본격적으로 기능을 개발하기 전, 프로젝트의 유지보수 및 서비스 확장을 대비하여 다양한 소프트웨어 아키텍처를 고민하였습니다. 클린 아키텍처, 핵사고날 아키텍처 등 다양한 아키텍처들이 있지만, 
최종적으로는 Layered 아키텍처와 약간의 DDD의 원칙을 도입하여 아키텍처를 설계하였습니다.

어떤 이유로 Layered 아키텍처와 DDD를 도입하게 되었는지, 세부적으로 어떤 원칙들을 적용하여 프로젝트를 만들어가고 있는지 소개합니다.

----

## 🌱 Layered 아키텍처를 선택하게 된 배경

소프트웨어를 설계할 때 다양한 아키텍처 패턴들이 존재합니다. 최근 개발자들 사이에서 주목받고 있는 '핵사고날 아키텍처'나 '클린 아키텍처'가 그 예입니다.
Pre Market 프로젝트는 이러한 여러 아키텍처 중에서 Layered 아키텍처를 메인으로 선택하고, DDD(Domain-Driven Design) 개념을 부분적으로 도입하여 아키텍처를 설계하였습니다.


### 🥕 Layered Architecture란?

> 각 구성 요소들이 '관심사의 분리(Separation of Concerns)'를 달성하기 위해 '책임'을 가진 계층으로 분리한 아키텍처

Layered Architecture는 시스템의 각 부분을 명확히 구분하여 높은 응집도와 낮은 결합도를 유지하는데 초점을 맞추고 있습니다. 이로 인해 재사용성과 유지보수성이 높아지는 장점이 있습니다.
각 계층은 비즈니스 로직, 데이터 엑세스, 프레젠테이션을 명확히 분리하며, 독립적인 책임을 수행합니다.

Spring Framework는 이러한 구조를 쉽게 구현할 수 있도록 돕는 어노테이션들을 제공하며, 이들은 각 계층에 맞는 역할을 표현하는데 유용합니다.


### 🥕 Layered Architecture의 각 계층

<img src="https://i.ibb.co/02mwWB9/architecture4.png" alt="architecture4" border="0">
	
1. Controller Layer
- 클라이언트의 요청을 직접적으로 받는 계층입니다. 이 계층은 클라이언트와의 상호작용을 처리하며, 클라이언트로부터 받은 요청을 Service Layer로 전달합니다.

2. Service Layer
- 비즈니스 로직을 담당하는 계층입니다. 단순히 클라이언트의 요청을 전달하는 것이 아니라, 도메인 로직을 처리하고 트랜잭션, 이벤트 관리 등 핵심 비즈니스 규칙을 실행합니다.

3.	Repository Layer 
- 데이터 액세스를 담당하는 계층으로, 실제 데이터베이스와의 상호작용을 관리합니다. 이 계층은 데이터베이스에 대한 CRUD 작업을 수행하며, 데이터 저장소에 대한 추상화를 제공합니다.

...

다음과 같은 장점으로 Layered 아키텍처를 Pre Market의 소프트웨어 아키텍처로 선택하게 되었습니다.

- 학습 비용이 상대적으로 적습니다. (러닝커브가 높은 아키텍처는 오히려 코드 품질을 떨어뜨릴 수 있다.)
- Spring framework는 기본적으로 @Controller, @Service, @Repository 등을 통해 관심사의 분리를 구현하고, 계층화된 구조를 지향하는 철학을 가지고 있습니다.

이 두 가지를 주요 이유로 Layered Architecture를 선택했습니다. 그리고 여기에 DDD의 일부 원칙을 도입해 아키텍처를 더욱 개선했습니다.

---

## 🌱 DDD (Domain-Driven Design)이란?

> DDD는 도메인 모델을 중심으로 소프트웨어를 설계하는 방식입니다. 
> Aggregate Root, Entity, Value Object, Repository 등의 개념을 통해 도메인의 복잡성을 해결하고, 낮은 결합도와 높은 응집도를 달성하는 것을 목표로 합니다.

Layered Architecture만으로는 서비스 계층이 복잡해지고 유지보수가 어려워질 수 있었습니다. 이런 문제를 해결하기 위해, 핵심 비즈니스 로직을 Domain 영역에 집중하는 DDD 원칙을 도입했습니다. 
DDD를 통해 서비스 로직을 더 명확하게 도메인 객체로 분리할 수 있었고, 비즈니스 규칙을 더욱 응집력 있게 관리할 수 있었습니다.

<img src="https://i.ibb.co/zVQvW4N/architecture3.png" alt="architecture3" border="0">

### 🥕 DDD의 핵심 개념

1. User Interface & Presentation
- 클라이언트의 요청을 직접 받는 계층입니다. 요청을 받은 후, 데이터를 Application Layer로 전달하고 필요한 필드 검증 등을 수행합니다.

2. Application
- 도메인 객체 간의 흐름을 제어하며, 비즈니스 로직을 직접 포함하지 않고 트랜잭션과 이벤트 관리를 담당합니다.

3. Domain
- 핵심 비즈니스 로직을 포함하는 계층입니다. 여기서 Aggregate Root와 Entity를 관리하며, 비즈니스 규칙과 도메인 이벤트를 처리합니다.

4. Infrastrcuture
- 외부 시스템과의 상호작용을 담당합니다. 데이터베이스, SMTP 클라이언트, Redis 등 외부 기술 스택과의 연결을 담당하며, 도메인 로직과 분리된 기술적 세부 사항을 처리합니다.

---

## 🌱 Pre Market Architecture

Pre Market의 아키텍처는 Layered Architecture를 기본으로 하고, DDD의 일부 개념을 도입하여 설계되었습니다.

<img src="https://i.ibb.co/zmzY3wV/pre-architecture.jpg" alt="pre-architecture" border="0">

### 🥕 Presentation Layer

기존의 Presentation 영역과 큰 차이점은 가지고 있지 않습니다.

- 클러이언트로부터 요청을 직접적으로 받고, 요청 받은 데이터를 Application 영역에서 필요한 데이터에 맞게 가공하여 전달 및 기능 처리를 위임합니다.
- 1차적으로 전달받은 필드 값을 Validation 합니다.

```java
@RestController
@RequestMapping(
    value = "/api",
    produces = MediaType.APPLICATION_JSON_VALUE,
    consumes = MediaType.APPLICATION_JSON_VALUE
)
@RequiredArgsConstructor
public final class AccountCommandApi {

    private final AccountFacade accountFacade;
    
    @PostMapping("/v1/accounts")
    public ResponseEntity<ApiResult<RegisterAccountResponse>> register(
        @RequestBody @Valid RegisterAccountRequest request
    ) {
        var memberId = accountFacade.register(request.toRegisterAccountDto(), request.verificationCode());
        return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(ApiResult.created(new RegisterAccountResponse(memberId.value())));
    }
}

//-------------------------------------------

public record RegisterAccountRequest(
    @Email(message = "The email field must contains a valid email address")
    @NotBlank(message = "The email field is required")
    String email,
    
    @NotBlank(message = "The password field is required")
    @Length(min = 8, max = 20, message = "The password must be between 8 and 20 characters")
    String password,
    
    @NotBlank(message = "The name field is required")
    String name,
    
    @JsonProperty(value = "phone_number")
    @NotBlank(message = "The phone number field is required")
    @Pattern(regexp = "[0-9]{10,11}", message = "The phone number must contains between 10 and 11 digits")
    String phoneNumber,
    
    @Valid
    @JsonProperty(value = "address")
    @NotNull(message = "The address field is required")
    Address address,
    
    @JsonProperty(value = "verification_code")
    @NotBlank(message = "The verification code is required")
    String verificationCode
) {
    public record Address(
        @JsonProperty(value = "base_address")
        @NotBlank(message = "The address1 field is required")
        String baseAddress,
    
        @JsonProperty(value = "address_detail")
        @NotBlank(message = "The address2 field is required")
        String addressDetail,
    
        @NotBlank(message = "The zipcode field is required")
        String zipcode
    ) {
    }

    public RegisterAccountDto toRegisterAccountDto() {
        return RegisterAccountDto.builder()
            .email(email)
            .rawPassword(password)
            .name(name)
            .phoneNumber(phoneNumber)
            .address(new AddressDto(address.baseAddress, address.addressDetail, address.zipcode))
            .build();
    }
}
```

Presentation 영역은 Application 영역을 의존하는 <strong>상위 계층이 하위 계층을 의존하는 특징</strong>이 그대로 표현됩니다.

Presentation 영역을 통해 <strong>클라이언트에게 제공하는 API endpoint, Http method, Http Request Body, Response Body 등을 구성합니다.</strong>


### 🥕 Application Layer

Application 영역은 각 도메인 객체의 처리 흐름 및 트랜잭션, Event 등을 담당합니다. 많은 프로젝트들 중 일부분은 <strong>Service</strong>라는 계층 이름 하에 모든 비즈니스 로직과 흐름을 처리합니다. 
하지만 많은 역할을 담당하게 되면서 의존하는 개체들도 증가함에 따라 가독성 및 유지보수가 쉽지 않았던 단점을 보완하기 위해 Application 영역을 도입하였습니다.

```java
@Service
@RequiredArgsConstructor
public class AccountFacade {

    private final RegisterAccountProcessor registerAccountProcessor;
    private final IssueVerificationCodeProcessor issueVerificationCodeProcessor;
    private final VerificationCodeVerifier verificationCodeVerifier;
    private final ApplicationEventPublisher publisher;
    
    @Transactional
    public MemberId register(final RegisterAccountDto accountDto, final String verificationCode) {
        verificationCodeVerifier.verify(accountDto.email(), verificationCode);
        return registerAccountProcessor.register(accountDto.toDomain());
    }
    
    @Transactional
    public void issueVerification(final String toEmail) {
        VerificationCode verificationCode = issueVerificationCodeProcessor.issue(toEmail);
        publisher.publishEvent(
            VerificationCodeEvent.createEvent(verificationCode.getToEmail(), verificationCode.getCode()));
    }
}
```

Application 영역을 담당하는 클래스 이름은 xxxFacade로 명명하여 클래스가 담당하는 역할을 직관적으로 표현하였습니다.

Application(Facade) 영역은 Presentation 영역과 Domain 영역을 이어주는 중간 다리 역할을 수행하며, 각 요청마다 필요한 서브 시스템을 하나의 인터페이스로 묶어주는 
역할을 수행합니다.

이에 따라 자연스럽게 서브 시스템에서 통합으로 필요한 기능인 트랜잭션이나 Event 같은 횡적 관심사 기능도 함께 수행합니다.

### 🥕 Domain Layer

Pre Market 프로젝트의 Domain 영역은 다양한 역할들을 수행합니다. 

먼저, Aggregate로 불리우는 도메인 엔티티를 가집니다. Aggregate는 현재 해결하고자 하는 도메인 영역을 객체화하고, 도메인 문제 해결에 필요한 비즈니스 로직을 담고 있습니다.

```java
@Getter
@ToString(exclude = "password")
@EqualsAndHashCode(of = "memberId")
public final class Account {

    private Long id;
    private MemberId memberId;
    private String email;
    private Password password;
    private String name;
    private String phoneNumber;
    private Address address;
    private MemberRole role;
    private AuditTimestamps auditTimestamps;
    private EntityStatus status;
    
    @Builder
    private Account(Long id, MemberId memberId, String email, Password password, String name, String phoneNumber,
        Address address, MemberRole role, AuditTimestamps auditTimestamps, EntityStatus status) {
        this.id = id;
        this.memberId = memberId;
        this.email = email;
        this.password = password;
        this.name = name;
        this.phoneNumber = phoneNumber;
        this.address = address;
        this.role = role;
        this.auditTimestamps = auditTimestamps;
        this.status = status != null ? status : EntityStatus.ALIVE;
    }
    
    public Account registerWithEncryptedPassword(String encodedPassword) {
        return Account.builder()
            .memberId(generateMemberId())
            .email(email)
            .password(new Password(this.password.rawPassword(), encodedPassword))
            .name(name)
            .phoneNumber(phoneNumber)
            .address(address)
            .role(MemberRole.NORMAL_CUSTOMER)
            .status(EntityStatus.ALIVE)
            .build();
    }
    
    private MemberId generateMemberId() {
        return MemberId.of(UUID.randomUUID().toString());
    }
}

//---------------

public interface AccountReader {

    boolean existsByEmail(String email);
    
    boolean existsByPhoneNumber(String phoneNumber);
    
    Optional<Account> findByEmail(String email);
}

//-------------------


public interface AccountRepository {

    MemberId save(Account account);
}
```

Domain 영역은 순수한 Java 코드로 작성되어 있기 때문에 외부 환경에 구애받지 않습니다. 즉, 구체적인 기술에 의존하는 것이 아니라 고수준의 논리만 담당하기 때문에 
유지보수성이 증가합니다.

또한, Domain 영역에서 Infrastructure 요소에 접근하여 요청을 처리하기 위해서 DIP (Dependency Inversion Principal) 개념을 사용합니다. 기존 Layered Architecture의 특징은 
상위 계층이 하위 계층을 의존하고, 그 역은 성립하지 않는 것입니다. 하지만 Domain 영역에서 Infrastructure 영역을 의존하게 되면 Infrastructure 영역에서 사용하는 특정 기술에 의존하게 되기 때문에(JPA, SMTP, Redis 등) 
의존 관계를 역전시켜 Domain 영역을 외부로부터 보호합니다.

### 🥕 Infrastructure Layer

Pre Market 프로젝트의 Infrastructure 영역은 주로 Persistence를 담당하는 JPA, RedisTemplate, SMTP Client 등이 구현되어 있습니다. 추상화된 개념이 아닌 특정 기술의 구현체를 의존하고, 외부 
시스템과 연동되어 있다는 점이 특징입니다.

```java
@Repository
@RequiredArgsConstructor
public class AccountPersistenceAdapter implements AccountRepository, AccountReader {
    
    private final JpaAccountRepository accountRepository;
    private final AccountMapper accountMapper;
    
    @Override
    public MemberId save(final Account account) {
        MemberEntity savedEntity = accountRepository.save(accountMapper.toEntity(account));
    
        return MemberId.of(savedEntity.getMemberId());
    }
    
    @Override
    public boolean existsByEmail(final String email) {
        return accountRepository.findByEmail(email).isPresent();
    }
    
    @Override
    public boolean existsByPhoneNumber(final String phoneNumber) {
        return accountRepository.findByPhoneNumber(phoneNumber).isPresent();
    }
    
    @Override
    public Optional<Account> findByEmail(String email) {
        Optional<MemberEntity> entity = accountRepository.findByEmail(email);
        return entity.map(accountMapper::toDomain);
    }
}
```

주된 특징 중 하나는 Adpater 패턴을 사용하여 Persistence 구현체를 제어하는 클래스를 도입하였습니다. 직접적으로 외부 시스템과 연동되어 제공할 수 있는 데이터(ex. JpaRepository, RedisTemplate 등)와 
Domain 영역으로부터 요구되는 데이터와의 차이를 효과적으로 조정할 수 있도록 중간 계층을 제공하였습니다.

---

## 🌱 닫는 글

Layered Architecture는 학습 난이도가 낮고, 기본적인 프로젝트 구조를 빠르게 잡을 수 있는 장점이 있습니다. 그러나 프로젝트 규모가 커지거나 복잡도가 증가할 경우, 서비스 계층이 커지면서 유지보수가 어려워질 수 있습니다. 
이러한 문제를 보완하기 위해 DDD 개념을 도입하여 핵심 도메인 로직을 응집력 있게 관리하는 것을 방안으로 설계하였습니다.


#### 📚 Reference

- [도메인 주도 개발 시작하기](https://product.kyobobook.co.kr/detail/S000001810495)
- [Layered Architecture Deep Dive](https://msolo021015.medium.com/layered-architecture-deep-dive-c0a5f5a9aa37)