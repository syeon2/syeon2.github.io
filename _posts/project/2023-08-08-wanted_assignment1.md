---
layout: post
title: "[원티드 프리온보딩 백엔드] #1 JWT를 통한 로그인 인증"
subtitle: "원티드 프리온보딩 사전 과제에서 사용한 기술 정리 - JWT"
categories: project
tags: project wanted pre_onboarding assignment jwt
---

> JWT를 사용하여 로그인 및 사용자 인증, 인가를 구현하고 정리한 글입니다.

<!--more-->

## 🌱 JWT를 통한 인증, 인가를 구현하기에 앞서...

인증, 인가에 대해 설명하기에는 이미 좋은 글들이 많아 패스하고자 한다. ([인증, 인가란?](https://dev.gmarket.com/45))

먼저, JWT(Json Web Token)나 Session과 같은 기술들이 사용되는 핵심은 HTTP의 특징에 있다. HTTP 통신은 stateless(무상태)라는 특징을 가지고 있다. 
즉, HTTP 통신은 이전 통신의 상태를 보관하거나 유지하지 않는 단발적인 통신 방식을 가지고 있다. 요청 하나하나마다 이전의 통신과 연관성을 갖지 않으며, 각 요청은 독립적으로 처리된다.

이러한 무상태 특징은 서버와 클라이언트 간의 간단한 통신을 가능하게 하지만, 사용자의 로그인 상태나 쇼핑 카트와 같은 정보를 유지해야 할 필요가 있을 때는 문제가 될 수 있다. 
JWT나 Session과 같은 기술은 이런 문제를 해결하기 위해 사용되며, 사용자의 상태를 유지할 수 있는 방법을 제공한다.

해당 기술 과제에서는 로그인 시 인증을 통해 JWT 토큰을 발행하고, 각 게시글을 수정할 때 인가 처리하여 자신의 게시글을 수정하거나 삭제하는 권한을 가질 수 있게
구현하도록 요구하고 있다.

### 🟤 구현 계획
JWT와 함께 Spring security를 사용해서 토큰을 발행할 수 있는 방법이 있는 것 같았지만, 다른 프레임워크를 추가적으로 충분히 스터디하기에는 시간 자원이 적다고 판단했다. 
결국 JWT를 별도로 생성할 수 있는 별도의 라이브러리만을 사용해서 구현하기로 했다.

또한, Spring Interceptor를 통해 모든 API를 호출할 때 Token을 검사하는 것과 Argument Resolver에 Token을 체크하는 어노테이션을 등록하는 것 두가지를 놓고 고민했다.
이번 과제에서는 구현해야하는 API가 적고, 어떤 API에서 Header를 체크해야하는지 명확하게 전달하고 있기에 Argument Resolver를 사용하여 인가처리하는 방법을 사용하기로 했다.   

---

## 🌱 어떻게 구현했는가?

### 🟤 라이브러리
사용한 라이브러리는 jjwt이다. 다른 라이브러리도 있는 것 같지만, jjwt는 Java기반으로 되어 있으며, JWT에 있어서 인지도가 있는 라이브러리인 것 같아 사용하게 되었다.
([jjwt github](https://github.com/jwtk/jjwt)를 참고해봐도 많은 사람들이 Fork와 Star를 한 라이브러리라는 것을 확인할 수 있다.)


```
 implementation 'io.jsonwebtoken:jjwt-api:0.11.2'
 implementation 'io.jsonwebtoken:jjwt-impl:0.11.2'
 implementation 'io.jsonwebtoken:jjwt-jackson:0.11.2'
```

jjwt 라이브러리의 특징은 각마다 구현체, 인터페이스 등의 역할을 가지고 있기 때문에 각각 호출할 수 있는 API가 있는 것이 아닌 jjwt-api를 통해 정의된 API 인터페이스에 impl과 jackson이 
구현체로 존재한다.

### 🟤 구현 코드
JWT 기능을 구현하면서 생성한 클래스는 크게 2가지라고 생각된다. 첫번째는 JWT의 메타 정보, 유효성 검사 및 파싱, 생성 등의 메소드를 가지고 있는 Provider 클래스. 
두번째는 인가를 위해 Argument Resolver를 등록한 클래스 정도로 나눌 수 있을 것 같다.

Provider 클래스는 이렇게 구현하였다.

```java
   @Component
   public class JwtAuthTokenProvider {
   
       @Value("${jwt.secret}")
       private String secret;
   
       @Value("${jwt.expired_minutes}")
       private Integer expiredMinutes;
   
       public boolean validateToken(String token) {
           try {
               return !Jwts.parserBuilder()
                   .setSigningKey(getHashingKey())
                   .build()
                   .parseClaimsJws(removeBearer(token))
                   .getBody()
                   .getExpiration().before(new Date());
           } catch (Exception e) {
               throw new CustomJwtTokenException("토큰 검증 실패");
           }
       }
   
       public MemberTokenInfo parsingTokenToMember(String token) {
           try {
               Claims body = Jwts.parserBuilder()
                   .setSigningKey(getHashingKey()).build()
                   .parseClaimsJws(removeBearer(token))
                   .getBody();
   
               Long id = Long.parseLong(body.get("id", String.class));
               String email = body.get("email", String.class);
   
               return MemberTokenInfo.builder()
                   .id(id)
                   .email(email)
                   .build();
           } catch (RuntimeException e) {
               throw new CustomJwtTokenException("토큰 검증 실패");
           }
       }
   
       public String createJwtAuthToken(Long id, String email) {
           Claims claims = Jwts.claims();
           claims.put("id", id.toString());
           claims.put("email", email);
   
           return Jwts.builder()
               .setHeaderParam("typ", "JWT")
               .setClaims(claims)
               .signWith(getHashingKey(), SignatureAlgorithm.HS256)
               .setExpiration(TimeGenerator.getTimeInFuture(expiredMinutes))
               .compact();
       }
   
       private String removeBearer(String token) {
           return token.replace("Bearer", "").trim();
       }
   
       private SecretKey getHashingKey() {
           return Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
       }
   }
```

메소드를 보면 Provider가 가지고 있는 역할은 간단하다. 유효성 검사를 하거나 토큰을 파싱하거나 토큰 생성이 Provider 클래스의 역할이다.

#### 1. 멤버 변수
application.yml에서 Hashing할 때 필요한 secret키와 토큰 만료 시간을 가져와 해당 클래스의 맴버 변수에 할당한다.

#### 2. validation 메소드
```java
   public boolean validateToken(String token) {
     try {
         return !Jwts.parserBuilder()
             .setSigningKey(getHashingKey())
             .build()
             .parseClaimsJws(removeBearer(token))
             .getBody()
             .getExpiration().before(new Date());
     } catch (Exception e) {
         throw new CustomJwtTokenException("토큰 검증 실패");
     }
   }
   
   private SecretKey getHashingKey() {
     return Keys.hmacShaKeyFor(secret.getBytes(StandardCharsets.UTF_8));
   }
```

validateToken은 토큰이 유효한지 확인하는 메소드이다. 차근차근히 살펴보고자 한다.

#### 🥕 jwts.parserBuilder()

Jwts는 jjwt-api에 있는 인터페이스이다. 따라서 실질적으로 동작을 수행하기 위해서는 구현체가 필요하다. 이 때 구현체를 가지고 오는 메소드가 parserBuilder()이다. 
parserBuilder 코드 내부를 보면 위에서 gradle로 함께 install한 라이브러리 중 jjwt-impl에 있는 구현체(DefaultJwtParserBuilder)를 로드하는 것을 볼 수 있다.

```java
   public static JwtParserBuilder parserBuilder() {
     return Classes.newInstance("io.jsonwebtoken.impl.DefaultJwtParserBuilder");
   }
--------
   public static <T> T newInstance(String fqcn) {
     return (T)newInstance(forName(fqcn));
   }
--------
   public static <T> Class<T> forName(String fqcn) throws UnknownClassException {
   
      Class clazz = THREAD_CL_ACCESSOR.loadClass(fqcn);
   
      if (clazz == null) {
         clazz = CLASS_CL_ACCESSOR.loadClass(fqcn);
      }
	  
   .....
   
      return clazz;
   }
--------
   public Class loadClass(String fqcn) {
      Class clazz = null;
      ClassLoader cl = getClassLoader();
      
      if (cl != null) {
         try {
           clazz = cl.loadClass(fqcn);
         } catch (ClassNotFoundException e) {
           //Class couldn't be found by loader
         }
      }
      return clazz;
   }
```

parserBuilder메소드의 내부를 자세히 살펴보면 이런 코드들을 통해 Jwts의 구현체를 가지고 오는 것을 확인할 수 있다. 간단하게 요약하자면 Jwts의 인터페이스에 대한 
구현체를 jjwt-impl 라이브러리에서 찾는 것으로 보인다. 이 때, JVM의 클래스로더를 사용하여 해당 클래스를 찾아 반환한다.


Jwts의 parserBuilder가 호출되기 전까지는 io.jsonwebtoken.impl.DefaultJwtParserBuilder 의 해당 클래스가 로드되어있지 않다. JVM은 동적로드 기법을 사용하여 
동작하기 때문이다. 특히, ClassLoader.loadClass 메소드를 사용하는 런타임 동적로드를 사용하여 해당 parserBuilder에 필요한 구현체를 타 라이브러리로부터 가지고 오고 있다.

클래스로더에 대한 설명은 추후 따로 포스팅해야할 것 같다. <br />
- [클래스 로더 포스팅](https://medium.com/@gsy4568/jvm%EC%9D%98-%EC%B2%AB%EA%B4%80%EB%AC%B8-classloader-ecdf93d53a7b)
- [클래스 로더 간단 활용법](https://syeon2.github.io/devlog/classloader.html)

----

#### 🥕 parseClaimsJws(token).getBody().getExpiration....

JWT는 알겠는데 JWS는 또 무엇일까? 

JWS는 JWT의 종류 중 하나이다. JWT는 단순히 JSON Web Token의 약자로 전자 서명 된 URL-safe (URL로 이용할 수있는 문자 만 구성된)의 JSON이다. 
JWS는 JWT의 가장 일반적인 구조로써, header, payload, signature 3 부분으로 나눌 수 있다. 이 밖에 JWE, JWK, JWA 등의 개념도 있는 것 같지만 나머지는 패스한다.
([JWT 소개](https://codecurated.com/blog/introduction-to-jwt-jws-jwe-jwa-jwk/#json-web-signature-jws))


Claim이란 용어도 등장한다. Claim이란 JWT의 Payload에 들어있는 각 개별 정보이다. Payload는 발행자, 주제, 만료시 등의 표준 클레임이 있으며 사용자가 임의로 
수정한 사용자 정의 클래임이 있다.

----

#### 🥕 validateToken 메소드 정리

해당 메소드의 궁극적인 목적은 토큰의 유효성을 검증한다.
> 🥕 메소드가 토큰을 유효성 검사하는 과정 
> 
> 1. jjwt-impl의 DefaultJwtParserBuilder 클래스를 찾아 해당 .class를 클래스로더를 통해 로드시킨 뒤, 해당 클래스의 인스턴스를 생성해서 반환한다.
> 
> 
> 2. 서버에서 가지고 있던 시크릿 키 문자열을 UTF-8 인코딩으로 바이트 배열로 변환한다. 변환된 바이트 배열은 DefaultJwtParserBuilder의 인스턴스에 저장된다.
> 
> 
> 3. JWT 토큰 구성 중 Payload 부분을 Claim으로 파싱한다. (Claim은 발행자, 주제, 만료 시간을 key-value 형태로 가지는 데이터 집합이다.)
> 이 때, setSigningKey에 저장한 키와 Token에 사용된 시크릿키가 다르다면 서명 에러를 발생시킨다.
> 
> 
> 4. Claim을 getBody로 변환하여 Expiration time을 비교한다. 만료된 토큰이라면 예외를 발생시킨다.

---

#### 3. parsingTokenToMember 메소드 & createJwtAuthToken 메소드

위의 validation 파트에 Jwt와 Claim 등의 개념을 거의 설명한 것 같아 특별하게 설명할 부분은 없는 것 같다.

메소드 명대로 parsingTokenToMember는 토큰에 있는 데이터를 사용하여 Member 인스턴스를 생성해 반환하는 메소드이다. 
createJwtAutoToken은 로그인 시 유저에게 JWT 토큰을 발행하기 위해 토큰을 생성하는 메소드이다.


----

## 🌱 구현하면서 느낀 점

Session는 구현해본 경험이 있다. 경험상 서버를 신경쓰지 않는다면 Session이 오히려 구현하기 간편했던 것으로 기억하고 있다. 보안적인 측면에서도 토큰 값이 
아무런 내용을 내포하고 있지 않을 뿐 아니라 서버에서 관리하고 있는 Session이 더 좋은 성능을 가질 것이라고 생각한다. (JWT는 로그인을 지속적으로 연장시키기 위해서는 
Refresh Token을 사용해야하는데 이것은 결국 서버의 스토리지를 사용해야하는 것이므로 Session과 별다른 차이가 없다.)

JWT의 강점은 타 서비스의 인증을 통한 확정성이라고 생각한다. social login같은 기능은 빅테크 기업의 우수한 보안 기술과 접근성을 용이하게 해준다. Session과 
JWT 모두 장단점을 가지고 있을거라 생각해서 다음 포스팅 주제로 Session vs JWT를 내가 구현했던 방식을 기준으로 작성해보고자 한다.
