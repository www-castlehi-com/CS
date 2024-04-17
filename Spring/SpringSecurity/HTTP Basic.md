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
# 인증 필터
## BasicAuthenticationFilter
### 개념
- 기본 인증 서비스 제공하는데 사용
- `BasicAuthenticationConverter`를 사용해서 request header에 기술된 인증정보의 유효성 체크, Base64 인코딩된 username, password 추출
- 세션을 사용하는 경우 매 요청마다 인증과정을 거치지 않아도 됨
> 💡 Http Basic 인증 방식은 보통 세션을 사용하지 않음
### 흐름
![](https://i.imgur.com/JW3BCO0.png)
1. `BasicAuthenticationFilter`가 요청을 받아 인증정보 유효성 체크
2. 추출한 username, password로 `UsernamePasswordAuthenticationToken` 제작
3. 토큰을 [[AuthenticationManager]]에 전달

**인증 성공 시**
4. UserDetails와 권한 등을 이용해 `UsernamePasswordAuthenticationToken`을 생성 -> 사용자의 정보 저장
5. `SecurityContextHolder`가 사용자의 **인증 상태 유지**를 위해 [[Authentication]]Token 인증 객체를 [[SecurityContext]]에 설정하고 이 컨텍스트를 request context에 저장 -> 요청 범위 내에서만 인증 상태 유지
6. `RememberMeServices`를 통해 [[rememberMe]]가 설정된 경우 loginSuccess를 호출
7. 다음 필터로 진행

**인증 실패 시**
4. `SecurityContextHolder` 삭제 -> 비정상적인 시도 막기
5. `RememberMeServices`의 loginFail을 호출하여 기억하기 설정 삭제
6. WWW-Authenticate를 보내도록 `AuthenticationEntryPoint` 호출
	> 💡 form login 방식은 로그인 페이지로 보내고, http basic방식은 헤더에 다시 담아서 보냄
### 코드
```java
package org.springframework.security.web.authentication.www;  
  
public class BasicAuthenticationFilter extends OncePerRequestFilter {  

	//...

	private AuthenticationConverter authenticationConverter = new BasicAuthenticationConverter();
	
	//...

	@Override  
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)  
	      throws IOException, ServletException {  
	   try {  
	      Authentication authRequest = this.authenticationConverter.convert(request);  
	      if (authRequest == null) {  
	         this.logger.trace("Did not process authentication request since failed to find "  
	               + "username and password in Basic Authorization header");  
	         chain.doFilter(request, response);  
	         return;      }  
	      String username = authRequest.getName();  
	      this.logger.trace(LogMessage.format("Found username '%s' in Basic Authorization header", username));  
	      if (authenticationIsRequired(username)) {  
	         Authentication authResult = this.authenticationManager.authenticate(authRequest);  
	         SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();  
	         context.setAuthentication(authResult);  
	         this.securityContextHolderStrategy.setContext(context);  
	         if (this.logger.isDebugEnabled()) {  
	            this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));  
	         }  
	         this.rememberMeServices.loginSuccess(request, response, authResult);  
	         this.securityContextRepository.saveContext(context, request, response);  
	         onSuccessfulAuthentication(request, response, authResult);  
	      }  
	   }  
	   catch (AuthenticationException ex) {  
	      this.securityContextHolderStrategy.clearContext();  
	      this.logger.debug("Failed to process authentication request", ex);  
	      this.rememberMeServices.loginFail(request, response);  
	      onUnsuccessfulAuthentication(request, response, ex);  
	      if (this.ignoreFailure) {  
	         chain.doFilter(request, response);  
	      }  
	      else {  
	         this.authenticationEntryPoint.commence(request, response, ex);  
	      }  
	      return;  
	   }  
	  
	   chain.doFilter(request, response);  
	}

	//...
}
```
- `OncePerRequestFilter` : 요청에 대해서 한 번만 실행
- `authenticationConverter(BasicAuthenticationConverter)` : 유효성 검증
	```java
	package org.springframework.security.web.authentication.www;  
	  
	public class BasicAuthenticationConverter implements AuthenticationConverter {  
	
		public static final String AUTHENTICATION_SCHEME_BASIC = "Basic";
	  
		// ...
		
		@Override  
		public UsernamePasswordAuthenticationToken convert(HttpServletRequest request) {  
			String header = request.getHeader(HttpHeaders.AUTHORIZATION);  
			if (header == null) {  
			 return null;  
			}  
			header = header.trim();  
			if (!StringUtils.startsWithIgnoreCase(header, AUTHENTICATION_SCHEME_BASIC)) {  
			 return null;  
			}  
			if (header.equalsIgnoreCase(AUTHENTICATION_SCHEME_BASIC)) {  
			 throw new BadCredentialsException("Empty basic authentication token");  
			}  
			byte[] base64Token = header.substring(6).getBytes(StandardCharsets.UTF_8);  
			byte[] decoded = decode(base64Token);  
			String token = new String(decoded, getCredentialsCharset(request));  
			int delim = token.indexOf(":");  
			if (delim == -1) {  
			 throw new BadCredentialsException("Invalid basic authentication token");  
			}  
			UsernamePasswordAuthenticationToken result = UsernamePasswordAuthenticationToken  
			 .unauthenticated(token.substring(0, delim), token.substring(delim + 1));  
			result.setDetails(this.authenticationDetailsSource.buildDetails(request));  
			return result;  
		}  
	
		//...
	
	}
	```
	- `if (headeer == null)` : 헤더에 값(`Basic encoding`)이 실리지 않는다면 더 이상 처리하지 않음
	- `startWithIgnoreCase(header, AUTHENTICATION_SCHEME_BASIC)` : "Basic"으로 시작하는지 확인
	- username, password 추출
	- `UsernamePasswordAuthentication reuslt ...` : 추출한 값으로 토큰 제작
-  `authenticationIsRequired(username)` : `securitContext` 내에 인증 객체가 있는지 확인 -> 있을 경우 인증을 받았다고 판단
	> ⚠️ Session의 경우 항상 존재하지만 Request마다 저장하는 것이기 때문에 항상 null이어서 이 과정이 필수
- `authenticationManager.authenticate(authRequest)` : `authenticationManager`에게 인증 처리 위임
- `securityContextRepository.saveContext(~)` : securityContext에 요청 정보 저장
	![](https://i.imgur.com/68eO7EF.png)
	```java
package org.springframework.security.web.context;  
  
	public final class RequestAttributeSecurityContextRepository implements SecurityContextRepository {  
	
		// ...
		
		@Override  
		public void saveContext(SecurityContext context, HttpServletRequest request, HttpServletResponse response) {  
		  request.setAttribute(this.requestAttributeName, context);  
		}  
		
		// ...
		
	}
	```
	- `ReauestAttributeSecurityContextRepository` : 요청 범위 내에 security context를 저장