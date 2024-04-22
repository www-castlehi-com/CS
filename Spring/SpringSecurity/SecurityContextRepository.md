## 개념
- 사용자 인증 후 요청에 대해 계속 사용자의 인증을 유지하기 위해 사용되는 클래스
- 사용자가 인증을 하면 해당 사용자의 UserDetails 정보와 권한 정보를 이용해 [[Authentication]] 객체를 만든 후, [[SecurityContext]]에 저장하고 이를 HttpSession을 통해 요청 간 영속화
## 흐름
![](https://i.imgur.com/yR74gih.png)
**인증 요청**
1. Client의 요청
2. AuthenticationFilter를 통해 사용자 인증 후, [[UserDetails]] 정보와 권한 정보를 이용해 Authentication 객체를 만들어 SecurityContext에 저장
3. SecurityContextRepository에 명시적으로 SecurityContext를 저장
4. SecurityContextRepository를 통해 HttpSession에 SecurityContext 저장

**인증 후 요청**
1. Client의 요청
2. [[SecurityContextHolderFilter]]를 통해 context 저장소로부터 context를 로드
3. SecurityContextRepository를 통해 세션으로부터 context 존재 확인
4. HttpSession으로부터 context를 가져옴
## 구조
### API
![](https://i.imgur.com/bXm581M.png)
- `saveContext(...)` : 인증 요청 완료 시 SecurityContext 저장
- `containsContext(...)` : SecurityContext가 저장소에 있는지 여부 조회
- `loadDeferredContext(...)` : Supplier에 저장해 로딩을 지연시켜 필요 시점에 SecurityContext 로드
### 구현체
![](https://i.imgur.com/ijtiKWX.png)
- `HttpSessionSecurityContextRepository` : HttpSession에 저장 -> context 영속성 유지
- `NullSecurityContextRepository` : 세션을 사용하지 않는 인증 (JWT, OAuth2)일 경우 사용 -> context 관련 처리 X 
- `RequestAttributeSecurityContextRepository` : SerlvletRequest에 저장 -> context 영속성 유지 X
	> ⚠️ 요청 객체는 요청마다 변경되기 때문
![](https://i.imgur.com/KY0en3h.png)
- `DelegatingSecurityContextRepository` : `RequestAttributeSecurityContextRepository`와 `HttpSessionSecurityContextRepositiory`를 동시에 사용 -> 초기화 시 기본 설정
## 코드
**SecurityContextConfigurer**
```java
package org.springframework.security.config.annotation.web.configurers;  
  
public final class SecurityContextConfigurer<H extends HttpSecurityBuilder<H>>  
      extends AbstractHttpConfigurer<SecurityContextConfigurer<H>, H> {  

	//...

	@Override  
	@SuppressWarnings("unchecked")  
	public void configure(H http) {  
	  SecurityContextRepository securityContextRepository = getSecurityContextRepository();  
	  if (this.requireExplicitSave) {  
		 SecurityContextHolderFilter securityContextHolderFilter = postProcess(  
			   new SecurityContextHolderFilter(securityContextRepository));  
		 securityContextHolderFilter.setSecurityContextHolderStrategy(getSecurityContextHolderStrategy());  
		 http.addFilter(securityContextHolderFilter);  
	  }  
	  else {  
		 SecurityContextPersistenceFilter securityContextFilter = new SecurityContextPersistenceFilter(  
			   securityContextRepository);  
		 securityContextFilter.setSecurityContextHolderStrategy(getSecurityContextHolderStrategy());  
		 SessionManagementConfigurer<?> sessionManagement = http.getConfigurer(SessionManagementConfigurer.class);  
		 SessionCreationPolicy sessionCreationPolicy = (sessionManagement != null)  
			   ? sessionManagement.getSessionCreationPolicy() : null;  
		 if (SessionCreationPolicy.ALWAYS == sessionCreationPolicy) {  
			securityContextFilter.setForceEagerSessionCreation(true);  
			http.addFilter(postProcess(new ForceEagerSessionCreationFilter()));  
		 }  
		 securityContextFilter = postProcess(securityContextFilter);  
		 http.addFilter(securityContextFilter);  
	  }  
	}  
}
```
- `getSecurityContextRepository()`
	![](https://i.imgur.com/6DTL7h3.png)
- `requireExplicitSave`가 true일 경우 
	- `new SecurityContextHolderFilter(repository)` : SecurityContextHolderFilter를 생성
- false일 경우
	- `new SecurityContextPersistenceFilter(repository)` : SecurityContextpersistenceFilter를 생성

**FormLoginConfigurer**
```java
package org.springframework.security.config.annotation.web.configurers;  
  
public abstract class AbstractAuthenticationFilterConfigurer<B extends HttpSecurityBuilder<B>, T extends AbstractAuthenticationFilterConfigurer<B, T, F>, F extends AbstractAuthenticationProcessingFilter>  
      extends AbstractHttpConfigurer<T, B> {  

	//...
  
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
- `authFilter.setSecurityContextRepository(repository)`
	- authFilter
		![](https://i.imgur.com/CfNqiXi.png)
	- 요청 후, 세션에 context를 저장해야하기 때문에 SecurityContextRepository 필요