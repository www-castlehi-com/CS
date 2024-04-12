# HTTP Basic 인증이란?
- 가장 일반적인 인증 방식
- RFC7235 표준
- 인증 프로토콜은 HTTP 인증 헤더에 기술
## 흐름
![](https://i.imgur.com/x7S9Map.png)
1. Client의 요청
2. Server가 Client에게 인증 요구를 보낼 때 `401 Unauthorized`와 함께 `WWW-Authenticate` 헤더를 기술해서 realm(보안 영역)과 Basic 인증방법을 전송
	![](https://i.imgur.com/GwWbC8j.png)
3. Client가 Server로 접속할 때 Base64로 username, password를 인코딩하고 Authorization 헤더에 담아 요청
	![](https://i.imgur.com/zGCBmtP.png)
4. 성공 시, 정상적인 상태 코드 반환 (`200 OK`)

> ⚠️ base-64 인코딩은 디코딩이 가능해 인증정보가 노출되므로 HTTPS와 같이 TLS 기술과 함께 사용
## API
```java
HttpSecurity.httpBasic(httpSecurityHttpBasicConfigurer -> httpSecurityHttpBasicConfigurer
					.realmName("security")
					.authenticationEntryPoint(
						(request, response, authException) -> {}
					)
);
```
- `realmName` : HTTP 기본 영역을 설정
- `AuthenticationEntryPoint` : 인증 실패로 사용자로 하여금 /login 페이지로 돌아가게 하는 인터페이스
- 기본적으로 `BasicAuthenticationEntryPoint` 사용