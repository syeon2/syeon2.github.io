---
layout: post
title: "[Hot-Deal #1] Member 도메인 개발"
subtitle: "Hot-Deal 프로젝트"
categories: project
tags: project domain
---

> Member 도메인을 개발하면서 사용한 기술과 특징을 소개합니다.

<!--more-->

- [🌱 Member Domain 소개](#-member-domain-소개)
  - [🟤 QueryDSL로 update 쿼리 인터페이스화](#-querydsl로-update-쿼리-인터페이스화)
  - [🟤 SHA-256 암호화 알고리즘으로 비밀번호 암호화](#-sha-256-암호화-알고리즘으로-비밀번호-암호화)
  - [🟤 Session과 JWT 중 JWT를 선택하여 로그인을 구현한 사례](#-session과-jwt-중-jwt를-선택하여-로그인을-구현한-사례)

## 🌱 Member Domain 소개

> 기본적으로 Member Domain은 회원가입, 프로필 수정, 로그인 등의 기능을 가지고 있습니다.

Member 도메인을 구현하면서 스터디한 기술은 비밀번호 암호화 (SHA-256)와 JWT 방식의 Access Token & Refresh Token으로 지속 가능한 
로그인 기능을 구현한 것입니다.

<img src="https://i.ibb.co/rMbTkDp/hd-project-member.png" alt="hd-project-member" width="800" >

회원 테이블은 이와 같으며 회원 도메인의 기본 서비스를 구현하면서 스터디하고 겪은 사례를 소개합니다.

1. [QueryDSL로 update 쿼리 인터페이스화](#-querydsl로-update-쿼리-인터페이스화)
2. [SHA-256 암호화 알고리즘으로 비밀번호 암호화](#-sha-256-암호화-알고리즘으로-비밀번호-암호화)
3. [Session과 JWT 중 JWT를 선택하여 로그인을 구현한 사례](#-session과-jwt-중-jwt를-선택하여-로그인을-구현한-사례)

-------

### 🟤 QueryDSL로 update 쿼리 인터페이스화

Hot-Deal 프로젝트는 DB에 쿼리를 요청하는 기술로 JPA를 기본으로 사용하는 프로젝트입니다. JPA의 특징은 이미 JpaRepository 인터페이스를 extends하여 
기본적으로 제공하는 API를 통해 쉽게 쿼리 요청할 수 있다는 장점을 가지고 있습니다. 하지만 JPA는 해당 Entity를 통해 엔티티 속성인 멤버 변수의 값을 변경해주기만 해도 
해당 엔티티의 레코드에 update 쿼리가 요청되는 편리함?을 가지고 있습니다.

이러한 편리함은 어떻게 update 쿼리가 요청되는지를 코드 레벨에서 바로 확인할 수 없으며, 해당 도메인의 Repository Layer에서 어떤 update 쿼리들을 사용하고 있는지 
알 수 없는 상황이 생길 수 있습니다.

또한, 무분별하게 엔티티 인스턴스의 멤버 변수를 변경하여 어디에서 update 쿼리가 요청되는지의 트레킹도 추후 유지보수할 시기에 꼭 필요한 단계라고 생각되었습니다. 이에 JPA를 
사용하면서 인터페이스로 어떤 update 쿼리 요청 기능을 명세할 수 있도록 QueryDSL을 사용하였습니다.

가령, 회원 수정을 위한 API를 개발한다고 할 때, 단순히 엔티티 값을 변경하는 메소드를 구현하면, 해당 메소드는 Repository Layer에 선언되는 것이 아닌
엔티티 클래스나 다른 비즈니스 클래스에 선언되어 유지보수 및 가독성에 좋지 않은 영향을 줄 수 있습니다. DB에 직접적으로 쿼리 요청을 보내는 메소드를 Repository Layer에 두어 어떤 update 쿼리들이
사용되고 있는지 필요하다고 판단했습니다.

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

### 🟤 SHA-256 암호화 알고리즘으로 비밀번호 암호화

기본적으로 비밀번호를 DB에 저장할 때는 암호화하여 저장합니다. '단순히 보안을 위해' 라고 표현할 수 있지만 2가지 정도의 이유를 설명드리고자 합니다.

첫번째로는 회사도 해당 유저의 비밀번호를 알 수 없어야합니다. 회사 내의 정책으로도 암호화를 사용하는 것을 권장할 수 있겠습니다. 어딘가에서 들려오는 이야기로는 유저가 가지고 있는 웬만한 
개인정보(주소, 휴대번호 등) 또한 암호화하여 저장한다는 이야기를 들은 적이 있습니다. 회사 내에서 개인의 개인정보를 알 수 없도록 DB에 개인정보를 암호화하여 저장한다고 합니다.

두번째로는 해킹에 대한 방지입니다. 일어나선 안되는 일이지만, 회사도 해킹의 대상이 되어 불시에 회원정보가 유출될 수 있습니다. 일반적으로 사람들은 비밀번호를 비슷하게 구성하여 사용하는 경향이 
많기 때문에 한 번 평문으로 된 비밀번호가 유출되면 다른 도메인에서도 그 비밀번호를 활용한 해킹의 위험에 노출될 수 있습니다.

이러한 이유 2가지를 가지고 이번 프로젝트에서는 비밀번호 정도로 암호화를 진행하여 저장하도록 스터디하였습니다.

```java
public class Sha256Util {
  private static final String ENCRYPTION_TYPE = "SHA-256";

  public static EncryptedSourceDto getEncryptDto(String source) {
    String salt = generateSalt();
    String encryptedSource = getEncryptedSourceWithSalt(source, salt);

    return EncryptedSourceDto.builder()
            .encryptedSource(encryptedSource)
            .salt(salt)
            .build();
  }

  public static String getEncryptedSourceWithSalt(String source, String salt) {
    try {
      MessageDigest md = MessageDigest.getInstance(ENCRYPTION_TYPE);

      byte[] byteData = md.digest(source.concat(salt).getBytes());

      return IntStream.range(0, byteData.length)
              .mapToObj(i -> String.format("%02x", byteData[i]))
              .collect(Collectors.joining());
    } catch (NoSuchAlgorithmException e) {
      throw new RuntimeException("SHA-256 암호화 에러");
    }
  }

  private static String generateSalt() {
    ThreadLocalRandom random = ThreadLocalRandom.current();

    byte[] salt = new byte[8];
    random.nextBytes(salt);

    StringBuilder sb = new StringBuilder();
    for (byte b : salt) {
      sb.append(String.format("%02x", b));
    }

    return sb.toString();
  }
}

```

비밀번호 암호화 알고리즘으로는 SHA-256 알고리즘을 선택하였습니다. SHA-256은 단방향 암호화 알고리즘으로 복호화가 불가능하다는 특징을 가지고 있습니다. 
하지만 비밀번호를 클라이언트에서 직접 받아서 무언가를 하는 동작은 존재하지 않을 뿐더러, 복호화할 수 있는 키를 외부에 제공하는 것은 위험부담이 크다고 생각하여 
단방향 암호화 알고리즘으로 비밀번호를 암호화하기에 충분하다고 판단하였습니다.

여기서 발생할 수 있는 문제점이 존재합니다. 단순히 비밀번호 하나를 아무런 추가적인 처리없이 해싱하게 된다면, 같은 비밀번호를 설정한 유저들의 암호화된 비밀번호는 
동일할 것입니다. 이를 통해 해커가 `레인보우 어택`, `브루트 포스 어택` 등의 방법을 통해 비밀번호가 유출될 가능성이 높습니다.

> 레인보우 어택 : 해커들이 미리 지정해놓은 다이제스트를 활용하여 해싱된 비밀번호를 복호화하는 방법
> 브루트 포스 어택 : 무차별적으로 모든 문자를 한번식 대입해서 비밀번호를 풀어내는 방법

이런 SHA-256의 단점을 보완하기 위해 salt라는 것을 추가시킵니다. salt는 인증하고자 하는 비밀번호에 무작위의 문자열을 합하여 함께 암호화한 방식입니다. 
같은 "12345"라는 문자를 암호화하더라도 1번 "12345"에 "a"라는 문자를 추가하여 암호화하는 것과 "b"라는 문자를 추가하여 암호화하는 것이 다르듯이 salt는 
같은 비밀번호라할지라도 서로 다르게 암호화하여 DB에 저장할 수 있도록 합니다. 물론 유저마다 가질 salt는 회원가입시 함께 생성하여 DB에 저장하는 것으로 설계하였습니다.

아래 블로그에서 자세하게 설명되어 있어 자세한 설명은 생략합니다.

##### 🧾 참고 Reference
- [패스워드의 암호화와 저장 - Hash(해시)와 Salt(솔트)](https://st-lab.tistory.com/100)


### 🟤 Session과 JWT 중 JWT를 선택하여 로그인을 구현한 사례

Session과 JWT는 둘다 로그인 인증/인가를 위해 사용되는 기술입니다. 로그인을 하고 난 이후에 이 로그인을 유지하기 위해서는 해당 유저가 로그인을 했다는 것을 어딘가에 
저장할 필요가 있습니다. 하지만 Http 통신은 stateless 하기 때문에 유저가 로그인을 했다는 정보를 항상 가지고 통신하지 않습니다.

로그인을 유지하기 위해서는 유저가 로그인을 했다는 정보를 어딘가에 저장해야합니다. 이 때 저장하는 공간을 클라이언트인지, 서버인지에 따라 Session 방식과 JWT 토큰 방식으로 
구분할 수 있습니다.

Session은 서버에서 로그인을 지속시킬 수 있는 토큰을 생성하고 클라이언트에 제공합니다. 클라이언트는 제공된 Session Id를 Http 통신 시 헤더에 넣고 
통신하게 됩니다. 서버는 클라이언트에서 헤더에 담아 보낸 토큰을 자신이 가지고 있는 토큰과 비교하여 요청을 성공시키거나 실패시키도록 합니다.

반면에 JWT는 클라이언트가 토큰을 보관하고 있는 주체가 됩니다. access Token이라고 불리는 토큰을 서버에서 생성하여 클라이언트에게 전달하면 클라이언트는 이 Access Token을 
브라우저에 저장하고 인가가 필요할 시 저장한 토큰을 사용하는 방식입니다. 서버는 해당 토큰이 자신이 생성한 토큰이 맞는지 해쉬 함수를 통해 복호화하여 비교합니다.

Session과 JWT는 어디에서 토큰을 주로 보관하는가를 기준으로 나뉩니다. 이러한 특성은 각 기술들의 장단점을 명확하게 나눕니다.

---

#### 🥕 Session의 장/단점

Session은 서버가 토큰을 관리하기 때문에 보안성이 JWT보다 뛰어나다는 장점을 가지고 있습니다. 유저는 로그인 시 서버에 요청에 Session Id를 생성하게 되면 
Session Id는 서버에서 관리하기 때문에, 서버에서 강제적으로 Session Id를 삭제하여 로그인 상태를 제거할 수 있습니다.

하지만 브라우저에도 Session ID를 저장하고 통신 요청을 보내기 때문에 만약 세션 아이디를 탈취당하면 세션 아이디를 탈취한 해커가 서버에게 요청해도 
정상적인 요청이 가능하여 이러한 부분을 해소할 방법이 없다는 것입니다. 하지만 이 단점은 모든 토큰을 활용한 인가 방식을 가지고 있는 모든 기술들이 
가지고 있는 단점이기 때문에 서버에서 강제적으로 Session Id를 제거하여 보안성을 높였다라고 설명할 수 있습니다.

#### 🥕 JWT의 장/단점

JWT는 서버에서 토큰을 생성하여 클라이언트에게 전달하면, 클라이언트는 해당 토큰을 브라우저에 저장하여 인가가 필요할 요청 시 헤더에 담아 보냅니다.

프로세스는 간단하지만, 간단한만큼 보안적으로 굉장히 취약합니다. JWT의 Access Token을 발행하게 되면 이후 Access Token을 관리하는 것은 브라우저이기 때문에 
한번 토큰을 탈취당하면 서버측에서도 어떻게 할 수 있는 방법이 없습니다. 그렇기 때문에 Access Token에는 보통 만료시간을 넣어 토큰을 사용할 수 있는 
제한 시간을 설정합니다. 하지만 이 제한시간은 본래 로그인을 유지시키려는 목적에 반하기 때문에 지속적으로 로그인을 유지시키지 못한다는 특징을 가지고 있습니다.


JWT의 Access Token만으로는 로그인을 장시간 유지시키지 못하기 때문에 Refresh Token이라는 장치를 따로 마련합니다. Refresh Token은 서버에 저장되는 토큰입니다. 
유저가 로그인을 하면 서버는 Access Token과 Refresh Token을 생성하여 클라이언트에게 제공합니다. 클라이언트는 Access Token을 사용하여 로그인 인가를 처리합니다. 
이후 Access Token이 만료되면 서버측에서 401 에러를 반환하여 클라이언트에게 알리고, 클라이언트는 Refresh Token을 서버에게 보내어 서버가 가지고 있는 Refresh Token과 
일치하는지 비교합니다. 비교가 성공하면 서버가 다시 새로운 AccessToken을 발행하여 제공하는 방식으로 처리합니다.

이 방법은 본래의 JWT가 가지고 있었던 토큰을 서버에 저장하지 않아 서버 비용을 단축시킬 수 있었던 이점을 활용하지 못한다는 점이 특징입니다.

---

결국 Session과 JWT는 비슷한 메커니즘을 가지면서 필요한 비용도 비슷합니다. JWT의 장점이었던 서버 비용의 단축은 Refresh Token 방식을 채택하면서 
사라집니다. 그럼 차라리 서버에서 제어 가능한 Session 방식을 선택하는 것이 좋지 않을까 생각됩니다. 그럼에도 JWT가 사용되는 이유는 무엇일까요? 바로 `확장성`이라고 생각합니다.

요즘 다양한 서비스들은 큰 기업에서 제공하는 소셜 로그인 기능을 사용합니다. 소셜 로그인은 큰 기업에서 제공하는 기술인만큼 유저들에게 접근성을 용이하게 하고 
여러가지 보안적인 측면을 간접적으로 사용한다는 장점을 가지고 있습니다. 구현해보지는 않았지만, 소셜 로그인에서 사용되는 기술이 JWT와 유사하여 쉽게 
로그인 기능을 확장 가능다하고 합니다. 따라서 이번 프로젝트에서는 JWT를 사용하여 로그인 인증/인가를 구현하였습니다.

```java
@Component
@RequiredArgsConstructor
public class JwtAuthTokenProvider {

	@Value("${jwt.secret_key}")
	private String secretKey;

	@Value("${jwt.at-expired_minutes}")
	private Integer accessTokenExpiredMinutes;

	@Value("${jwt.rt-expired_days}")
	private Integer refreshTokenExpiredDays;

	private static final String PAYLOAD_KEY = "id";

	private final RefreshTokenRepositoryImpl refreshTokenRepository;

	public JwtTokenResponse generateToken(Long memberId) {
		return JwtTokenResponse.builder()
			.accessToken(createAccessToken(memberId))
			.refreshToken(createRefreshToken(memberId))
			.build();
	}

	public JwtTokenResponse generateRenewToken(Long memberId, String refreshToken) {
		try {
			validateRefreshToken(memberId, refreshToken);

			return JwtTokenResponse.builder()
				.accessToken(createAccessToken(memberId))
				.build();
		} catch (ExpiredJwtException exception) {
			String newAccessToken = createAccessToken(memberId);
			String newRefreshToken = createRefreshToken(memberId);

			return JwtTokenResponse.builder()
				.accessToken(newAccessToken)
				.refreshToken(newRefreshToken)
				.build();
		} catch (Exception exception) {
			throw new TokenValidationException("유효하지 않은 토큰입니다.");
		}
	}

	public void removeRefreshTokenInStorage(String accessToken) {
		try {
			Claims body = Jwts.parserBuilder()
				.setSigningKey(generateHashingKey())
				.build()
				.parseClaimsJws(removeBearer(accessToken))
				.getBody();

			String id = body.get(PAYLOAD_KEY, String.class);
			refreshTokenRepository.delete(id);
		} catch (Exception exception) {
			throw new TokenValidationException("유효하지 않은 토큰입니다.");
		}
	}

	public boolean validateAccessToken(String accessToken) {
		try {
			Jwts.parserBuilder()
				.setSigningKey(generateHashingKey())
				.build()
				.parseClaimsJws(removeBearer(accessToken))
				.getBody();

			return true;
		} catch (ExpiredJwtException exception) {
			throw new ExpiredAccessTokenException("Access Token 토큰 만료");
		} catch (Exception exception) {
			throw new TokenValidationException("유효하지 않은 토큰입니다.");
		}
	}

	private String createAccessToken(Long id) {
		Claims claims = Jwts.claims();
		claims.put(PAYLOAD_KEY, id.toString());

		return Jwts.builder()
			.setHeaderParam("type", "JWT")
			.setClaims(claims)
			.signWith(generateHashingKey(), SignatureAlgorithm.HS256)
			.setExpiration(TimeGenerator.getMinuteInFuture(accessTokenExpiredMinutes))
			.compact();
	}

	private String createRefreshToken(Long id) {
		Claims claims = Jwts.claims();
		claims.put(PAYLOAD_KEY, id.toString());

		String refreshToken = Jwts.builder()
			.setHeaderParam("type", "JWT")
			.setClaims(claims)
			.signWith(generateHashingKey(), SignatureAlgorithm.HS256)
			.setExpiration(TimeGenerator.getDayInFuture(refreshTokenExpiredDays))
			.compact();

		refreshTokenRepository.save(id, refreshToken);

		return refreshToken;
	}

	private void validateRefreshToken(Long memberId, String refreshToken) {
		String originRefreshToken = refreshTokenRepository.getRefreshToken(memberId);

		if (!originRefreshToken.equals(refreshToken)) {
			throw new TokenValidationException("유효하지 않는 Refresh Token입니다..");
		}

		Claims body = Jwts.parserBuilder()
			.setSigningKey(generateHashingKey())
			.build()
			.parseClaimsJws(refreshToken)
			.getBody();

		String id = body.get(PAYLOAD_KEY, String.class);

		if (!memberId.toString().equals(id)) {
			throw new TokenValidationException("유효하지 않는 Refresh Token입니다.");
		}
	}

	private String removeBearer(String token) {
		return token.replaceFirst("Bearer ", "").trim();
	}

	private SecretKey generateHashingKey() {
		return Keys.hmacShaKeyFor(secretKey.getBytes(StandardCharsets.UTF_8));
	}
}
```

JWTAuthTokenProvider 클래스는 단순하게 4가지 정도의 메소드를 public으로 제공하고 있습니다.

#### 1. generateToken 메소드
generateToken 메소드는 유저가 로그인을 하면 accessToken과 refreshToken을 생성합니다. Access Token과 Refresh Token은 기본적인 토큰 생성 로직은 
거의 비슷하지만 만료시간이 다릅니다. 보통 Access Token을 Refresh Token보다 만료시간을 짧게 주어 해커가 토큰을 탈취하더라도 사용에 제한이 생길 수 있도록 합니다. 

#### 2. generateRenewToken 메소드
generateRenewToken은 Refresh Token을 비교하여 새로운 토큰을 발행합니다. 현 프로젝트에서는 Refresh Token을 사용하는 storage로 
Redis를 사용하고 있습니다. Redis에 저장된 Token값과 클라이언트로부터 받은 토큰이 일치하는지 비교하고, 만료시간의 여부를 파악합니다. 만약 
Refresh Token도 만료시간이 지났다면 Access Token과 함께 Refresh Token도 새로 발행하여 클라이언트에게 제공합니다.

#### 3. removeRefreshTokenInStorage 메소드
유저가 로그아웃 시 Redis에 저장되어 있는 토큰을 제거하는 메소드입니다.

#### 4. validateAccessToken 메소드
로그인 인가를 처리하기 위해서는 인가처리를 하기 위한 코드가 각 API에 선언되어 있어야합니다. 하지만 인가가 필요한 모든 API 컨트롤러 코드에 인가 코드를 
넣으면 반복적으로 사용되기 때문에 Spring의 interceptor를 사용하여 인가가 필요한 API만 따로 access Token을 validate하는 방법을 선택하였습니다.


----

#### 🥕 마무리

로그인에 관련된 인증/인가와 쿠키, 세션, 토큰에 대한 정보는 더 잘 설명되어 있는 레퍼런스가 있습니다. 기본적인 내용은 아래의 레퍼런스를 참고하는 것이 좋을 것이라 생각합니다.

...

로그인 인증/인가는 사실 위의 Session과 JWT 방식 2가지같이 이분법적으로 나눌 수 있는 것이 아닙니다. 
당연히 각 회사마다 사용하는 기술들은 조금씩 다르며, 무엇이 더 좋은 기술인지에 대해서는 일반화할 수 없습니다. 제공하고자하는 서비스와 회사마다의 보안정책에 따라 
얼마든지 다르게 구성할 수 있기 때문에 해당 기술은 회사의 내규에 따라야합니다.

이번 프로젝트에서는 소셜 로그인으로 확장에 대한 여부만으로 JWT를 사용하여 구현했습니다. 각 Session과 JWT에 대한 오해들도 많이 수반되어 있기 때문에 JWT에 대한 
공식 문서을 읽고 기술적 특징을 잘 분별해야겠다고 느끼게 되었습니다.

##### 🧾 참고 Reference

- [JWT 토큰 인증 이란? (쿠키 vs 세션 vs 토큰)](https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-JWTjson-web-token-%EB%9E%80-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC#session_%EC%9D%B8%EC%A6%9D)
- [세션 기반 인증과 토큰 기반 인증 (feat. 인증과 인가)](https://hudi.blog/session-based-auth-vs-token-based-auth/)
- [JWT에 대한 오해와, 세션으로 JWT(Json Web Token)을 사용하지 말자는 주장의 정리.](https://team00csduck.tistory.com/93)
- [JSON Web Token Cheat Sheet for Java](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
