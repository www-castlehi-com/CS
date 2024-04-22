# RememberMe 인증이란?
![](https://i.imgur.com/4Rmwr9a.png)
- 로그인 시 자동으로 인증 정보를 기억하는 기능
- `UsernamePasswordAuthenticationFilter`와 함께 사용, `AbstractAuthenticationProcessingFilter` 슈퍼클래스에서 훅을 통해 구현
- 인증 성공 : `RememberMeServices.loginSuccess()`를 통해 RememberMe 토큰을 생성하고 쿠키로 전달
- 인증 실패 : `RememberMeServices.loginFail()`을 통해 쿠키 삭제
- 로그아웃 : [[Logout]]Filter와 연계하여 쿠키 삭제
## RememberMe 토큰
- 암호화된 토큰
- 세션에서 해당 쿠키를 감지하여 자동 로그인이 이루어지는 방식
- `base64(username + ":" + expirationTime + ":" + algorithmName + ":" + algorithmHex(username + ":" + expirationTime + ":" + password + ":" + key)`
	- **username** : [[UserDetailsService]]로 식별 가능한 사용자 이름
	- **password** : `UserDetails`에 일치하는 비밀번호
	- **expirationTime** : 토큰이 만료되는 날짜, 시간
	- **key** : 토큰 수정을 방지하기 위한 개인 키
	- **algorithmName** : 토큰을 생성하고 검증하는데 사용되는 알고리즘 (기본 : SHA-256)
## RememberMeServices 구현체
구현체들은 `UserDetailsService`가 필요
### 1️⃣ TokenBasedRememberMeServices
- 쿠키 기반
- 보안을 위해 해싱 사용
- 메모리 저장
### 2️⃣ PersistentTokenBasedRememberMeServices
- DB / 영구 저장 매체 필요
## rememberMe() API
- `RememberMeConfigurer` 설정 클래스를 이용
- 내부적으로 `RememberMeAuthenticationFilter`가 생성되어 담당
```java
http.rememberMe(httpSecurityRememberMeConfigurer -> httpSecurityRememberMeConfigurer
	.alwaysRemember(true)
	.tokenValiditySeconds(3600)
	.userDetailsService(userDetailService)
	.rememberMeParameter("remember")
	.rememberMeCookieName("remember")
	.key("security")			   
);
```
- `alwaysRemember(true)` : 기억하기 설정이 되지 않아도 항상 기억
- `tokenValiditySeconds()` : 토큰 유효 시간 지정
- `userDetailsService()` : `UserDetails`를 조회하기 위해 사용되는 서비스 지정
- `rememberMeParameter()` : 기억하기 기능의 이름 (기본값 : 'remember-me')
	![](https://i.imgur.com/Heoi6Na.png)
- `rememberMeCookieName()` : 토큰을 저장하는 쿠키 이름 (기본값 : 'remember-me')
		![](https://i.imgur.com/bJiMuG6.png)
- `key()` : 토큰을 식별하는 키
## RememberMeAuthenticationFilter
### 개념
- `SecurityContextHolder`에 Authentication이 포함되지 않은 경우 실행 -> 인증을 받지 않았을 경우 실행
- 세션 만료, 애플리케이션 종료로 인해 인증 상태가 소멸된 경우 실행
- 토큰 기반 인증으로 토큰 유효성을 검사하고 검증 후, 자동 로그인 처리
### 흐름
![](https://i.imgur.com/XknrtFj.png)
1. Client가 요청
2. `RememberMeAuthenticationFilter`가 해당 요청을 받음
	- Authentication 객체가 Null일 경우 인증 처리
	- Null이 아닐 경우 인증이 된 상태이므로 다음 필터로 전환 (`chain.doFilter`)
3. `autoLogin()`을 호출하여 자동 로그인 처리
4. UserDetails, Authorities를 이용해 `RememberMeAuthenticationToken` 생성
5. 토큰을 `AuthenticationManager`가 처리

**인증 성공 시**
6. `RememberMeAuthenticationToken`을 `SeucirytContextHolder`에 저장
7. [[SecurityContextRepository]]를 이용해 세션에 [[SecurityContext]] 저장
8. `ApplicationEventPublisher`를 통해 인증 성공 이벤트 게시

**인증 실패 시**
6. `loginFail()`을 이용해 remember-me 쿠키 삭제
### 코드
```java
package org.springframework.security.web.authentication;  
  
public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean  
      implements ApplicationEventPublisherAware, MessageSourceAware {  

	//...

	protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,  
      Authentication authResult) throws IOException, ServletException {  
	   SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();  
	   context.setAuthentication(authResult);  
	   this.securityContextHolderStrategy.setContext(context);  
	   this.securityContextRepository.saveContext(context, request, response);  
	   if (this.logger.isDebugEnabled()) {  
	      this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));  
	   }  
	   this.rememberMeServices.loginSuccess(request, response, authResult);  
	   if (this.eventPublisher != null) {  
	      this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(authResult, this.getClass()));  
	   }  
	   this.successHandler.onAuthenticationSuccess(request, response, authResult);  
	}

	//...
	
}
```
```java
package org.springframework.security.web.authentication.rememberme;  
  
public abstract class AbstractRememberMeServices  
      implements RememberMeServices, InitializingBean, LogoutHandler, MessageSourceAware {  

	//...

	@Override  
	public void loginSuccess(HttpServletRequest request, HttpServletResponse response,  
	      Authentication successfulAuthentication) {  
	   if (!rememberMeRequested(request, this.parameter)) {  
	      this.logger.debug("Remember-me login not requested.");  
	      return;   }  
	   onLoginSuccess(request, response, successfulAuthentication);  
	}

	//...
	
}
```
- `UsernamePasswordAuthenticationFilter`의 슈퍼클래스인 `AbstractAuthenticationProcessingFilter`에서 인증 성공 시, `rememberMeServices.loginSuccess()` 호출
- `this.parameter`와 요청정보가 동일한지 확인 (`SecurityConfig`에서 `rememberMeParameter()`로 설정 가능)
- `onLoginSuccess` : 토큰 생성 후 쿠키로 전달
	```java
	package org.springframework.security.web.authentication.rememberme;  
	  
	public class TokenBasedRememberMeServices extends AbstractRememberMeServices {  
	
		//...
	
			@Override  
		public void onLoginSuccess(HttpServletRequest request, HttpServletResponse response,  
		      Authentication successfulAuthentication) {  
		   String username = retrieveUserName(successfulAuthentication);  
		   String password = retrievePassword(successfulAuthentication);  
		   // If unable to find a username and password, just abort as  
		   // TokenBasedRememberMeServices is   // unable to construct a valid token in this case.   if (!StringUtils.hasLength(username)) {  
		      this.logger.debug("Unable to retrieve username");  
		      return;   }  
		   if (!StringUtils.hasLength(password)) {  
		      UserDetails user = getUserDetailsService().loadUserByUsername(username);  
		      password = user.getPassword();  
		      if (!StringUtils.hasLength(password)) {  
		         this.logger.debug("Unable to obtain password for user: " + username);  
		         return;      }  
		   }  
		   int tokenLifetime = calculateLoginLifetime(request, successfulAuthentication);  
		   long expiryTime = System.currentTimeMillis();  
		   // SEC-949  
		   expiryTime += 1000L * ((tokenLifetime < 0) ? TWO_WEEKS_S : tokenLifetime);  
		   String signatureValue = makeTokenSignature(expiryTime, username, password, this.encodingAlgorithm);  
		   setCookie(new String[] { username, Long.toString(expiryTime), this.encodingAlgorithm.name(), signatureValue },  
		         tokenLifetime, request, response);  
		   if (this.logger.isDebugEnabled()) {  
		      this.logger  
		         .debug("Added remember-me cookie for user '" + username + "', expiry: '" + new Date(expiryTime) + "'");  
		   }  
		}
	
		//...
		
	}
	```
	- `setCookie(~, signatureValue)` : username, password, expireTime, algorithm으로 만든 `signatureValue`로 토큰을 만들어 쿠키에 전달
		![](https://i.imgur.com/0gPnHiJ.png)

> 💡 전체적으로 기억하기 토큰을 만들어 클라이언트에게 전달하는 것은 `UsernamePasswordAuthenticationFilter`([[form Login]], [[HTTP Basic]]) 에서 진행

```java
package org.springframework.security.web.authentication.rememberme;  
   
public class RememberMeAuthenticationFilter extends GenericFilterBean implements ApplicationEventPublisherAware {  
  
	//...

	private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)  
		 throws IOException, ServletException {  
	  if (this.securityContextHolderStrategy.getContext().getAuthentication() != null) {  
		 this.logger.debug(LogMessage  
			.of(() -> "SecurityContextHolder not populated with remember-me token, as it already contained: '"  
				  + this.securityContextHolderStrategy.getContext().getAuthentication() + "'"));  
		 chain.doFilter(request, response);  
		 return;      }  
	  Authentication rememberMeAuth = this.rememberMeServices.autoLogin(request, response);  
	  if (rememberMeAuth != null) {  
		 // Attempt authentication via AuthenticationManager  
		 try {  
			rememberMeAuth = this.authenticationManager.authenticate(rememberMeAuth);  
			// Store to SecurityContextHolder  
			SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();  
			context.setAuthentication(rememberMeAuth);  
			this.securityContextHolderStrategy.setContext(context);  
			onSuccessfulAuthentication(request, response, rememberMeAuth);  
			this.logger.debug(LogMessage.of(() -> "SecurityContextHolder populated with remember-me token: '"  
				  + this.securityContextHolderStrategy.getContext().getAuthentication() + "'"));  
			this.securityContextRepository.saveContext(context, request, response);  
			if (this.eventPublisher != null) {  
			   this.eventPublisher.publishEvent(new InteractiveAuthenticationSuccessEvent(  
					 this.securityContextHolderStrategy.getContext().getAuthentication(), this.getClass()));  
			}  
			if (this.successHandler != null) {  
			   this.successHandler.onAuthenticationSuccess(request, response, rememberMeAuth);  
			   return;            }  
		 }  
		 catch (AuthenticationException ex) {  
			this.logger.debug(LogMessage  
			   .format("SecurityContextHolder not populated with remember-me token, as AuthenticationManager "  
					 + "rejected Authentication returned by RememberMeServices: '%s'; "  
					 + "invalidating remember-me token", rememberMeAuth),  
				  ex);  
			this.rememberMeServices.loginFail(request, response);  
			onUnsuccessfulAuthentication(request, response, ex);  
		 }  
	  }  
	  chain.doFilter(request, response);  
	}  

	//...
	
}
```
- `securityContextHolderStrategy.getContext().getAuthentication()` : form login의 경우 세션에 저장되어 있기 때문에 null이 아님
	- null이 아닐 경우 자동 로그인 후 다음 필터
- null일 경우
	```java
	package org.springframework.security.web.authentication.rememberme;  
  
	public abstract class AbstractRememberMeServices  
	      implements RememberMeServices, InitializingBean, LogoutHandler, MessageSourceAware {  
	
		//...
	
		@Override  
		public Authentication autoLogin(HttpServletRequest request, HttpServletResponse response) {  
		   String rememberMeCookie = extractRememberMeCookie(request);  
		   if (rememberMeCookie == null) {  
		      return null;  
		   }  
		   this.logger.debug("Remember-me cookie detected");  
		   if (rememberMeCookie.length() == 0) {  
		      this.logger.debug("Cookie was empty");  
		      cancelCookie(request, response);  
		      return null;   }  
		   try {  
		      String[] cookieTokens = decodeCookie(rememberMeCookie);  
		      UserDetails user = processAutoLoginCookie(cookieTokens, request, response);  
		      this.userDetailsChecker.check(user);  
		      this.logger.debug("Remember-me cookie accepted");  
		      return createSuccessfulAuthentication(request, user);  
		   }  
		   catch (CookieTheftException ex) {  
		      cancelCookie(request, response);  
		      throw ex;  
		   }  
		   catch (UsernameNotFoundException ex) {  
		      this.logger.debug("Remember-me login was valid but corresponding user not found.", ex);  
		   }  
		   catch (InvalidCookieException ex) {  
		      this.logger.debug("Invalid remember-me cookie: " + ex.getMessage());  
		   }  
		   catch (AccountStatusException ex) {  
		      this.logger.debug("Invalid UserDetails: " + ex.getMessage());  
		   }  
		   catch (RememberMeAuthenticationException ex) {  
		      this.logger.debug(ex.getMessage());  
		   }  
		   cancelCookie(request, response);  
		   return null;
	   }
	
		//...

		protected Authentication createSuccessfulAuthentication(HttpServletRequest request, UserDetails user) {  
		   RememberMeAuthenticationToken auth = new RememberMeAuthenticationToken(this.key, user,  
		         this.authoritiesMapper.mapAuthorities(user.getAuthorities()));  
		   auth.setDetails(this.authenticationDetailsSource.buildDetails(request));  
		   return auth;  
		}

		//...
		
	}
	```
	- `rememberMeCookie == null` : remeber me 쿠키가 null이면 처리 X
	- `processAutoLoginCookie()` : 쿠키에서 [[UserDetails]] 정보 추출
	- `createSuccessfulAuthentication` : 인증 처리
		- key, user 객체, 권한을 이용해 `RememberMeAuthenticationToken` 생성
- `authenticationmanager.authenticate()` : [[AuthenticationManager]]에게 [[Authentication]] 객체 전달
- `securityContextHolderStrategy.setContext()` : security context 내에 저장 후 context를 세션에 저장
	![](https://i.imgur.com/eIWVisX.png)
- `eventPublisher.publishEvent()` : 이벤트 게시
- `successHandler.onAuthenticationSuccess()` : 핸들러 실행