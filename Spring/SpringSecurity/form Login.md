# 폼 인증이란?
- HTTP 기반 폼 로그인 인증 메커니즘을 활성화하는 API
- 사용자 인증을 위한 사용자 정의 로그인 페이지를 쉽게 구현할 수 있음
- 기본 로그인 페이지를 사용
	![](https://i.imgur.com/5x1Lzg0.png)
- 폼을 통해 사용자가 <이름, 비밀번호>를 제공하고 SpringSecurity는 이를 `HttpServletRequest`를 통해 읽음
## 흐름
![](https://i.imgur.com/EW1w23y.png)
1. Client가 특정 웹사이트 접근 요청
2. filterChain의 마지막 필터인 `AuthorizationFilter`에서 인증 검사
3. 인증이 되지 않았을 경우, `AccessDeniedException` 발생
4. `ExceptionTranslataionFilter`가 접근 예외 처리 -> 다시 인증을 받도록 로그인 페이지로 이동시킴
5. `AuthenticationEntryPoint`라는 기점을 통해 재인증 -> 로그인 페이지로 리다이렉트
6. <User, Password> 입력 후 인증 허가 시 서버 접근 가능
## API
```java
HttpSecurity.formLogin(httpSecurityFormLoginConfigurer -> httpSecurityFormLoginConfigurer)
	.loginPage("/loginPage")
	.loginProcessingUrl("/loginProc")
	.defaultSuccessUrl("/", [alwaysUse])
	.failureUrl("/failed")
	.usernameParameter("username")
	.passwordParameter("password")
	.failureHandler(AuthenticationFailureHandler)
	.successHandler(AuthenticationSuccessHandler)
	.permitAll()
);
```
- `loginPage()`
	- 사용자 정의 로그인 페이지로 전환
	- 기본 페이지 무시
- `loginProccessingUrl()`
	- 사용자 이름과 비밀번호를 검증할 URL 지정
	- html 태그의 \<form actions = ''>에 해당
- `defaultSuccessUrl()`
	- 로그인 성공 이후 이동할 페이지
	- `alwaysUse`가 true일 경우 무조건 지정된 위치로 이동 (기본값 : false)
	- 인증 전에 보안이 필요한 페이지를 방문하다가 인증에 성공할 경우 이전 위치로 리다이렉트
	- /home으로 이동하고 인증 페이지로 이동한 경우, 기본적으로 인증 후에 /home으로 이동하지만 `alwaysUse`가 true일 경우 `defaultSuccessUrl`로 설정한 URL로 이동
- `failureUrl()`
	- 인증 실패할 경우 사용자에게 보내질 URL (기본값 : /login?error)
- `usernameParameter()`
	- 인증을 수행할 때 사용자 아이디(user)를 찾기 위해 확인하는 HTTP 매개변수 (기본값 : username)
- `passwordParameter()`
	- 인증을 수행할 때 비밀번호를 찾기 위해 확인하는 HTTP 매개변수 (기본값 : password)
- `failureHandler()`
	- 인증 실패 시 사용할 핸들러 지정
	- 기본값 : `SimpleUrlAuthenticationFailureHandler`, /login?error로 리다이렉션
- `successHandler()`
	- 인증 성공 시 사용할 핸들러 지정
	- 기본값 : `SavedRequestAwareAuthenticationSuccessHandler`
- `permitAll()`
	- `failureUrl()`, `loginPage()`, `loginProcessingUrl()`에 대한 모든 사용자의 접근 허용
## 원리
```java
package org.springframework.security.config.annotation.web.builders;  
  
public final class HttpSecurity extends AbstractConfiguredSecurityBuilder<DefaultSecurityFilterChain, HttpSecurity>  
	public HttpSecurity formLogin(Customizer<FormLoginConfigurer<HttpSecurity>> formLoginCustomizer) throws Exception {  
	   formLoginCustomizer.customize(getOrApply(new FormLoginConfigurer<>()));  
	   return HttpSecurity.this;  
	}
}
```
- `customize` : 별도의 커스터마이징할 수 있는 작업, `FormLoginConfigurer` 생성
```java
package org.springframework.security.config.annotation.web.configurers;  
  
 public final class FormLoginConfigurer<H extends HttpSecurityBuilder<H>> extends  
      AbstractAuthenticationFilterConfigurer<H, FormLoginConfigurer<H>, UsernamePasswordAuthenticationFilter> {  
	public FormLoginConfigurer() {  
		super(new UsernamePasswordAuthenticationFilter(), null);  
		usernameParameter("username");  
		passwordParameter("password");  
	}
}
```
- `UsernamePasswordAuthenticationFilter` : form 인증을 처리하는 필터
```java
package org.springframework.security.config.annotation.web.configurers;  
  
 public abstract class AbstractAuthenticationFilterConfigurer<B extends HttpSecurityBuilder<B>, T extends AbstractAuthenticationFilterConfigurer<B, T, F>, F extends AbstractAuthenticationProcessingFilter>  
      extends AbstractHttpConfigurer<T, B> {  

	// ...
  
	private SavedRequestAwareAuthenticationSuccessHandler defaultSuccessHandler = new SavedRequestAwareAuthenticationSuccessHandler();

	// ...

	protected AbstractAuthenticationFilterConfigurer() {  
	   setLoginPage("/login");  
	}

	// ...

	@Override  
	public void init(B http) throws Exception {  
	   updateAuthenticationDefaults();  
	   updateAccessDefaults(http);  
	   registerDefaultAuthenticationEntryPoint(http);  
	}

	protected final void updateAuthenticationDefaults() {  
	   if (this.loginProcessingUrl == null) {  
	      loginProcessingUrl(this.loginPage);  
	   }  
	   if (this.failureHandler == null) {  
	      failureUrl(this.loginPage + "?error");  
	   }  
	   LogoutConfigurer<B> logoutConfigurer = getBuilder().getConfigurer(LogoutConfigurer.class);  
	   if (logoutConfigurer != null && !logoutConfigurer.isCustomLogoutSuccess()) {  
	      logoutConfigurer.logoutSuccessUrl(this.loginPage + "?logout");  
	   }  
	}

	protected final void updateAccessDefaults(B http) {  
	   if (this.permitAll) {  
	      PermitAllSupport.permitAll(http, this.loginPage, this.loginProcessingUrl, this.failureUrl);  
	   }  
	}

	protected final void registerDefaultAuthenticationEntryPoint(B http) {  
	   registerAuthenticationEntryPoint(http, this.authenticationEntryPoint);  
	}

	// ...

	@Override  
	public void configure(B http) throws Exception {  
		PortMapper portMapper = http.getSharedObject(PortMapper.class);  
		if (portMapper != null) {  
		  this.authenticationEntryPoint.setPortMapper(portMapper);  
		}  
		RequestCache requestCache = http.getSharedObject(RequestCache.class);  
		if (requestCache != null) {  
		  this.defaultSuccessHandler.setRequestCache(requestCache);  
		}  
		this.authFilter.setAuthenticationManager(http.getSharedObject(AuthenticationManager.class));  
		this.authFilter.setAuthenticationSuccessHandler(this.successHandler);  
		this.authFilter.setAuthenticationFailureHandler(this.failureHandler);  
		if (this.authenticationDetailsSource != null) {  
		  this.authFilter.setAuthenticationDetailsSource(this.authenticationDetailsSource);  
		}  
		SessionAuthenticationStrategy sessionAuthenticationStrategy = http  
		  .getSharedObject(SessionAuthenticationStrategy.class);  
		if (sessionAuthenticationStrategy != null) {  
		  this.authFilter.setSessionAuthenticationStrategy(sessionAuthenticationStrategy);  
		}  
		RememberMeServices rememberMeServices = http.getSharedObject(RememberMeServices.class);  
		if (rememberMeServices != null) {  
		  this.authFilter.setRememberMeServices(rememberMeServices);  
		}  
		SecurityContextConfigurer securityContextConfigurer = http.getConfigurer(SecurityContextConfigurer.class);  
		if (securityContextConfigurer != null && securityContextConfigurer.isRequireExplicitSave()) {  
		  SecurityContextRepository securityContextRepository = securityContextConfigurer  
			 .getSecurityContextRepository();  
		  this.authFilter.setSecurityContextRepository(securityContextRepository);  
		}  
		this.authFilter.setSecurityContextHolderStrategy(getSecurityContextHolderStrategy());  
		F filter = postProcess(this.authFilter);  
		http.addFilter(filter);  
	}

	//...
}
```
API들이 기술되어 있는 클래스
- defaultSuccessHandler : `SavedRequestAwareAuthenticationSuccessHandler`
- `setLoginPage()` : 기본값 "/login"

init()
- `updateAuthenticationDefaults()` : `loginProcessingUrl`, `failureUrl`, `successUrl` 설정
- `updateAccessDefaults()` : `permitAll()`로 `loginPage`, `loginProcessingUrl`, `failureUrl`을 모두 접근 허용 
- `registerDefaultAuthenticationEntryPoint()` :  기본 entry point 설정

configure()
- `authFilter`인 `UsernamePasswordAuthenticationFilter`에 대한 설정
- `http.addFilter()` : 인증 필터를 `HttpSecurity`에 추가
# 폼 인증 필터
## UsernamePasswordAuthenticationFilter
### 개념
- `AbstractAuhenticaitonProcessingFilter`를 확장한 클래스
- 사용자의 자격 증명을 인증하는 기본 필터
- `HttpServletRequest`에서 제출된 사용자 이름, 비밀번호로부터 인증 수행
- Login, [[Logout]] 페이지 생성을 위한 `DefaultLoginPageGeneratingFilter`, `DefaultLogoutPageGeneratingFilter` 초기화
### 구조
![](https://i.imgur.com/KZm8TO8.png)
- 기본 필터 : `UsernamePasswordAuthenticationFilter`
- 커스텀 필터 : `CustomAuthenticationFilter`
- 재정의 메소드 : `attemptAuthentication()`
### 흐름
![](https://i.imgur.com/OUlmodP.png)
1. Client의 요청
2. `UsernamePasswordAuthenticationFilter`의 `requestMatcher`를 통해 요청 정보가 매칭되는지 확인
	- 기본적으로 login 정보를 가지고 있음
	- 사용자가 login이 아닌 loginProc과 같은 다른 정보로 요청할 시 false를 반환
	- true를 반환할 경우 인증 과정으로 넘어가고 false일 경우 다음 필터(`chain.doFilter`) 호출
3. Username과 Password로 `UsernamePasswordAuthenticationToken`([[Authentication]] interface의 구현체) 제작
4. Token을 `AuthenticationManager`로 전달 -> 전달받은 Token의 username, password가 db 혹은 시스템과 일치하는지 확인

**인증 성공 시**
5. UserDetails와 권한 등을 이용해 `UsernamePasswordAuthenticationToken`을 생성 -> 사용자의 정보 저장
6. `SessionAuthenticationStrategy`를 이용해 사용자가 로그인에 성공했음을 알리고 세션 관련 작업 수행
7. `SecurityContextHolder`가 사용자의 **인증 상태 유지**를 위해 AuthenticationToken 인증 객체를 [[SecurityContext]]에 설정하고 이 컨텍스트를 세션에 저장
8. `RememberMeServices`를 통해 [[rememberMe]]가 설정된 경우 loginSuccess를 호출 -> 세션이 만료되어도 기억할 수 있음
9. `ApplicationEventPublisher`를 통해 인증 성공 이벤트 게시 -> 리스너를 통해 추가 작업 가능케 함
10. 인증 성공 핸들러인 `AuthenticationSuccessHandler`를 호출

**인증 실패 시**
5. `SecurityContextHolder` 삭제 -> 비정상적인 시도 막기
6. `RememberMeServices`의 loginFail을 호출하여 기억하기 설정 삭제
7. 인증 실패 핸들러인 `AuthenticationFailureHandler`를 호출 -> 인증 실패에 대한 예외 처리
### 코드
![](https://i.imgur.com/SRp4Bm6.png)
- `UsernamePasswordAuthenticationFilter`에 대해서 인증을 할 것인지 검사
```java
package org.springframework.security.web.authentication;  
  
 public abstract class AbstractAuthenticationProcessingFilter extends GenericFilterBean  
      implements ApplicationEventPublisherAware, MessageSourceAware {  

	//...

	private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)  
			throws IOException, ServletException {  
		if (!requiresAuthentication(request, response)) {  
		  chain.doFilter(request, response);  
		  return;   }  
		try {  
		  Authentication authenticationResult = attemptAuthentication(request, response);  
		  if (authenticationResult == null) {  
			 // return immediately as subclass has indicated that it hasn't completed  
			 return;  
		  }  
		  this.sessionStrategy.onAuthentication(authenticationResult, request, response);  
		  // Authentication success  
		  if (this.continueChainBeforeSuccessfulAuthentication) {  
			 chain.doFilter(request, response);  
		  }  
		  successfulAuthentication(request, response, chain, authenticationResult);  
		}  
		catch (InternalAuthenticationServiceException failed) {  
		  this.logger.error("An internal error occurred while trying to authenticate the user.", failed);  
		  unsuccessfulAuthentication(request, response, failed);  
		}  
		catch (AuthenticationException ex) {  
		  // Authentication failed  
		  unsuccessfulAuthentication(request, response, ex);  
		}  
	}

	//...
}
```
- `requiresAuthentication` : `requestMatcher`의 정보와 같다면 true 반환
- `attemptAuthentication` : `UsernamePasswordAuthentication`에서 인증처리 -> `User`객체가 반환됨
	```java
	@Override  
	public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)  
	      throws AuthenticationException {  
	   if (this.postOnly && !request.getMethod().equals("POST")) {  
	      throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());  
	   }  
	   String username = obtainUsername(request);  
	   username = (username != null) ? username.trim() : "";  
	   String password = obtainPassword(request);  
	   password = (password != null) ? password : "";  
	   UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username,  
	         password);  
	   // Allow subclasses to set the "details" property  
	   setDetails(request, authRequest);  
	   return this.getAuthenticationManager().authenticate(authRequest);  
	}
	```
	- `obtainXX()` : request에서 username, password를 얻음
	- `UsernamePasswordAuthentication.unauthenticated(username, password)` : authentication 구현체에 username, password를 넣은 인증 객체를 인증 전 단계로 생성
	- `authenticationManager.authenticate()` : 인증 객체를 인증 매니저에 위임하며 인증 처리 요청
- `sessionStrategy.onAuthentication()` : 세션 관리
- `successfulAuthentication()` : `SecurityContextHolder` 설정
	```java
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
	```
	- `securityContextHolderStrategy.createEmptyContext()` :  비어 있는 context holder를 얻음
	- `context.setAuthentication()` : 인증 성공한 객체 저장
	- `securityContextRepository.saveContext()` : 내부 객체인 `HttpSessionSecurityContextRepository`를 이용해 세션에 context holder 저장
		```java
		package org.springframework.security.web.context;  
		  
		public class HttpSessionSecurityContextRepository implements SecurityContextRepository {  
			
			// ...

			private void setContextInSession(SecurityContext context, HttpSession session) {  
			   if (session != null) {  
				  session.setAttribute(this.springSecurityContextKey, context);  
				  if (this.logger.isDebugEnabled()) {  
					 this.logger.debug(LogMessage.format("Stored %s to HttpSession [%s]", context, session));  
				  }  
			   }  
			}

			//...
		}
		```
		- `session.setAttribute()` : key-value 형태로 세션에 context holder 저장
	- `rememberMeServices.loginSuccess()` : remember-me 설정이 되어 있을 경우 처리
	- `eventPublisher.publishEvent()` : 인증 성공 이벤트 게시 -> `InteractiveAuthenticationSuccessEvent`를 통해 수신
	- `successHandler.onAuthenticationSuccess()` : 성공 이벤트 핸들러
- `unsuccessfulAuthentication()` : username, password가 다를 때 `AuthenticationException`을 catch하며 호출
	```java
	protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response,  
      AuthenticationException failed) throws IOException, ServletException {  
	   this.securityContextHolderStrategy.clearContext();  
	   this.logger.trace("Failed to process authentication request", failed);  
	   this.logger.trace("Cleared SecurityContextHolder");  
	   this.logger.trace("Handling authentication failure");  
	   this.rememberMeServices.loginFail(request, response);  
	   this.failureHandler.onAuthenticationFailure(request, response, failed);  
	}
	```
	- `securityContextHolderStrategy.clearContext()` : securityContextHolder 삭제
	- `rememberMeServices.loginFail()` : 기억하기 삭제
	- `failureHandler.onAuthenticationFailure()` : 실패 핸들러 호출
		```java
		package org.springframework.security.web.authentication;  
		
		public class SimpleUrlAuthenticationFailureHandler implements AuthenticationFailureHandler { 
		
			//...
			
			public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response,  
				 AuthenticationException exception) throws IOException, ServletException {  
				if (this.defaultFailureUrl == null) {  
				 if (this.logger.isTraceEnabled()) {  
					this.logger.trace("Sending 401 Unauthorized error since no failure URL is set");  
				 }  
				 else {  
					this.logger.debug("Sending 401 Unauthorized error");  
				 }  
				 response.sendError(HttpStatus.UNAUTHORIZED.value(), HttpStatus.UNAUTHORIZED.getReasonPhrase());  
				 return;      }  
				saveException(request, exception);  
				if (this.forwardToDestination) {  
				 this.logger.debug("Forwarding to " + this.defaultFailureUrl);  
				 request.getRequestDispatcher(this.defaultFailureUrl).forward(request, response);  
				}  
				else {  
				 this.redirectStrategy.sendRedirect(request, response, this.defaultFailureUrl);  
				}  
			}  
			
			//...
		
		}
		```
		- `sendRedirect` : defaultFailureUrl로 리다이렉트
			![](https://i.imgur.com/aiXnD2w.png)