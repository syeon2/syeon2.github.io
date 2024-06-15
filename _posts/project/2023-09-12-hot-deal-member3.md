---
layout: post
title: "[Hot-Deal #3] JWT를 활용하여 로그인 인증/인가"
subtitle: "JWT를 활용하여 로그인을 유지하는 기능을 소개합니다."
categories: devlog
tags: troubleshooting jwt session
---

> JWT를 활용하여 로그인을 유지하는 기능을 소개합니다.

<!--more-->

## 🌱 Session과 JWT 중 JWT를 선택하여 로그인을 구현한 사례

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

1. Session은 서버가 토큰을 관리하기 때문에 보안성이 JWT보다 뛰어납니다.
   - 유저 로그인 시 Session Id를 생성합니다. Session Id는 서버에서 관리하기 때문에 강제적으로 Session Id를 삭제하여 로그인 상태를 제거할 수 있습니다.

2. 브라우저에도 Session ID를 저장하고 통신 요청을 보내기 때문에 만약 세션 아이디를 탈취당하면 세션 아이디를 탈취한 해커가 서버에게 요청해도
   정상적인 요청이 가능하여 이러한 부분을 해소할 방법이 없습니다.
   - 모든 토큰을 활용한 인가 방식을 가지고 있는 모든 기술들이 가지고 있는 단점이기 때문에 서버에서 강제적으로 Session Id를 제거하여 보안성을 높였다라고 설명할 수 있습니다.

...

#### 🥕 JWT의 장/단점

JWT는 서버에서 토큰을 생성하여 클라이언트에게 전달하면, 클라이언트는 해당 토큰을 브라우저에 저장하여 인가가 필요할 요청 시 헤더에 담아 보냅니다.

1. JWT의 Access Token을 발행하게 되면 브라우저가 토큰을 관리하기 때문에 한번 토큰을 탈취당하면 서버 측에서도 어떻게 할 수 있는 방법이 없습니다.
   - Access Token에는 보통 만료시간을 넣어 토큰을 사용할 수 있는 제한 시간을 설정합니다.
   - 제한시간은 본래 로그인을 유지시키려는 목적에 반하기 때문에 지속적으로 로그인을 유지시키지 못한다는 특징을 가지고 있습니다.

2. JWT의 Access Token만으로는 로그인을 장시간 유지시키지 못하기 때문에 Refresh Token이라는 장치를 따로 마련합니다.
   - 유저가 로그인을 하면 서버는 Access Token과 Refresh Token을 생성하여 클라이언트에게 제공합니다.
   - Access Token이 만료되면 서버측에서 401 에러를 반환하여 클라이언트에게 알리고, 클라이언트는 Refresh Token을 서버에게 보내어 서버가 가지고 있는 Refresh Token과
     일치하는지 비교합니다. 비교가 성공하면 서버가 다시 새로운 AccessToken을 발행하여 제공하는 방식으로 처리합니다.
   - 이 방법은 본래의 JWT가 가지고 있었던 토큰을 서버에 저장하지 않아 서버 비용을 단축시킬 수 있었던 이점을 활용하지 못합니다.

---

결국 Session과 JWT는 비슷한 메커니즘을 가지면서 필요한 비용도 비슷합니다. JWT의 장점이었던 서버 비용의 단축은 Refresh Token 방식을 채택하면서 
사라집니다. 그럼 차라리 서버에서 제어 가능한 Session 방식을 선택하는 것이 좋지 않을까 생각됩니다. 그럼에도 JWT가 사용되는 이유는 무엇일까요? 바로 `확장성`이라고 생각합니다.

요즘 다양한 서비스들은 큰 기업에서 제공하는 소셜 로그인 기능을 사용합니다. 소셜 로그인은 큰 기업에서 제공하는 기술인만큼 유저들에게 접근성을 용이하게 하고 
여러가지 보안적인 측면을 간접적으로 사용한다는 장점을 가지고 있습니다. 구현해보지는 않았지만, 소셜 로그인에서 사용되는 기술이 JWT와 유사하여 쉽게 
로그인 기능을 확장 가능다하고 합니다. 따라서 이번 프로젝트에서는 JWT를 사용하여 로그인 인증/인가를 구현하였습니다.

...

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

---

##### 🧾 참고 Reference

- [JWT 토큰 인증 이란? (쿠키 vs 세션 vs 토큰)](https://inpa.tistory.com/entry/WEB-%F0%9F%93%9A-JWTjson-web-token-%EB%9E%80-%F0%9F%92%AF-%EC%A0%95%EB%A6%AC#session_%EC%9D%B8%EC%A6%9D)
- [세션 기반 인증과 토큰 기반 인증 (feat. 인증과 인가)](https://hudi.blog/session-based-auth-vs-token-based-auth/)
- [JWT에 대한 오해와, 세션으로 JWT(Json Web Token)을 사용하지 말자는 주장의 정리.](https://team00csduck.tistory.com/93)
- [JSON Web Token Cheat Sheet for Java](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
