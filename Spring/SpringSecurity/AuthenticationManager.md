# AuthenticationManager
- 필터로부터 [[Authentication]] 객체를 전달 받아 인증 시도
- 인증 성공 시, 사용자 정보와 권한 등을 포함한 완전히 채워진 Authentication 객체 반환
- 여러 AuthenticaitonProvider를 관리 -> 순차적으로 순회하며 인증 처리 요건에 맞는 적절한 AuthenticationProvider를 찾아 인증처리 위임
- `AuthenticationManagerBuilder`에 의해 객체 생성
- 주로 ProviderManager가 구현체
## AuthenticationManagerBuilder
- AuthenticationManager 객체 생성
- `UserDetailsService`, `AuthenticationProvider` 추가
- `HttpSecurity.getSharedObject(AuthenticationManagerBuilder.class)`를 통해 객체 참조
![](https://i.imgur.com/fUzwRHl.png)
- Security Builder의 구현체들([[WebSecurity & HttpSecurity]])와 더불어 `ProviderManagerBuilder`도 구현체
## 흐름
![](https://i.imgur.com/1eQ8v4u.png)
1. Client의 인증 요청
2. `Authentication` 객체를 AuthenticationManager의 구현체인 `ProviderManager`에게 전달
3. `AuthenticationProviders`를 선택하는 기준은 Manager가 가지고 있음
	- null이 아닌 응답을 받을 때까지 차례로 실행
	- 처리할 수 있는 Providers를 들고 있지 않을 경우 부모의 ProviderManager에서 Providers로 전달
	- 응답을 받지 못할 경우 `ProviderNotFoundException` 던짐
4. `UserDetails`로 만든 인증 객체를 다시 Manager에게 전달

```java
package org.springframework.security.authentication;  
  
public class ProviderManager implements AuthenticationManager, MessageSourceAware, InitializingBean {  

	//...
	
	@Override  
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {  
		Class<? extends Authentication> toTest = authentication.getClass();  
		AuthenticationException lastException = null;  
		AuthenticationException parentException = null;  
		Authentication result = null;  
		Authentication parentResult = null;  
		int currentPosition = 0;  
		int size = this.providers.size();  
		for (AuthenticationProvider provider : getProviders()) {  
		 if (!provider.supports(toTest)) {  
			continue;  
		 }  
		 if (logger.isTraceEnabled()) {  
			logger.trace(LogMessage.format("Authenticating request with %s (%d/%d)",  
				  provider.getClass().getSimpleName(), ++currentPosition, size));  
		 }  
		 try {  
			result = provider.authenticate(authentication);  
			if (result != null) {  
			   copyDetails(authentication, result);  
			   break;            }  
		 }  
		 catch (AccountStatusException | InternalAuthenticationServiceException ex) {  
			prepareException(ex, authentication);  
			// SEC-546: Avoid polling additional providers if auth failure is due to  
			// invalid account status            throw ex;  
		 }  
		 catch (AuthenticationException ex) {  
			lastException = ex;  
		 }  
		}  
		if (result == null && this.parent != null) {  
		 // Allow the parent to try.  
		 try {  
			parentResult = this.parent.authenticate(authentication);  
			result = parentResult;  
		 }  
		 catch (ProviderNotFoundException ex) {  
			// ignore as we will throw below if no other exception occurred prior to  
			// calling parent and the parent            // may throw ProviderNotFound even though a provider in the child already            // handled the request         }  
		 catch (AuthenticationException ex) {  
			parentException = ex;  
			lastException = ex;  
		 }  
		}  
		if (result != null) {  
		 if (this.eraseCredentialsAfterAuthentication && (result instanceof CredentialsContainer)) {  
			// Authentication is complete. Remove credentials and other secret data  
			// from authentication            ((CredentialsContainer) result).eraseCredentials();  
		 }  
		 // If the parent AuthenticationManager was attempted and successful then it  
		 // will publish an AuthenticationSuccessEvent         // This check prevents a duplicate AuthenticationSuccessEvent if the parent         // AuthenticationManager already published it         if (parentResult == null) {  
			this.eventPublisher.publishAuthenticationSuccess(result);  
		 }  
		
		 return result;  
		}  
		
		// Parent was null, or didn't authenticate (or throw an exception).  
		if (lastException == null) {  
		 lastException = new ProviderNotFoundException(this.messages.getMessage("ProviderManager.providerNotFound",  
			   new Object[] { toTest.getName() }, "No AuthenticationProvider found for {0}"));  
		}  
		// If the parent AuthenticationManager was attempted and failed then it will  
		// publish an AbstractAuthenticationFailureEvent      // This check prevents a duplicate AbstractAuthenticationFailureEvent if the      // parent AuthenticationManager already published it      if (parentException == null) {  
		 prepareException(lastException, authentication);  
		}  
		throw lastException;  
	}  

	//...
	
}
```
- `for (AuthenticationProvider provider : getProviders())` : List 타입의 providers를 순회
- `provider.authenticate(authentication)` : provider에게 인증 처리 위임
- `parent.authenticate(authentication)` : 결과값이 null일 경우, parent provider에게 위임
- `new ProviderNotFoundException(...)` : 부모 provider도 null일 경우 예외 발생
## 사용 방법
### 1️⃣ HttpSecurity
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
	AuthenticationManagerBuilder authenticationManagerBuilder = http.getSharedObject(AuthenticationManagerBuilder.class);
	AuthenticationManager authenticationManager = authenticationManagerBuilder.build();
	AuthenticationManager authenticationManager = authenticationManagerBuilder.getObject()

	http
		.authorizeHttpRequests(auth -> auth
			.requestMatchers("/api/login").permitAll()
			.anyRequest().authenticated()
		)
		.authenticationManager(authenticationManager)
		.addFilterBefore(customFilter(authenticationManager), UsernamePasswordAuthenticationFilter.class);

	return http.build();
}

public CustomAuthenticationFilter customFilter(AuthenticationManager authenticationManager) throws Exception {
	CustomAuthenticationFilter customAuthenticationFilter = new CustomAuthenticationFilter();
	customAuthenticationFilter.setAuthenticationManager(authenticationManager);
	return customAuthenticationFilter;
}
```
- `authenticationManagerBuilder.build()` : 최초 한 번만 호출
- `authenticationManagerBuilder.getObject()` : build() 후에는 getObject()로 참조
- `authenticationManager(authenticationManager)` : 생성한 `AuthenticationManager`를 HttpSecurity에 저장
- `customFilter` : `AuthenticationManager`가 빈이 아니기 때문에 `@Bean` X
### 2️⃣ 직접 생성
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
	http
		.authorizeHttpRequests(auth -> auth
			.anyRequest()
			.authenticated()
		)
		.formLogin(Customizer.withDefaults())
		.addFilterBefore(customFilter(), UsernamePasswordAuthenticationFilter.class);

	return http.build();
}

@Bean
public CustomAuthenticationFilter customFilter() {
	List<AuthenticationProvider> list1 = List.of(new DaoAuthenticationProvider());
	ProviderManager parent = new ProviderManager(list1);
	List<AuthenticationProvider> list2 = List.of(new AnonymousAuthenticationProvider("key"), new CustomAuthenticationProvider());
	ProviderManager authenticationManager = new ProviderManagere(list2, parent);

	CustomAuthenticationFilter customAuthenticationFilter = new CustomAuthenticationFilter();
	customAuthenticationFilter.setAutenticationManager(authenticationManager);

	return customAuthenticationFilter;
}
```
## 코드
```java
package org.springframework.security.config.annotation.web.configuration;  
  
class HttpSecurityConfiguration {  

	//...
	
	@Bean(HTTPSECURITY_BEAN_NAME)  
	@Scope("prototype")  
	HttpSecurity httpSecurity() throws Exception {  
		LazyPasswordEncoder passwordEncoder = new LazyPasswordEncoder(this.context);  
		AuthenticationManagerBuilder authenticationBuilder = new DefaultPasswordEncoderAuthenticationManagerBuilder(  
			this.objectPostProcessor, passwordEncoder);  
		authenticationBuilder.parentAuthenticationManager(authenticationManager());  
		authenticationBuilder.authenticationEventPublisher(getAuthenticationEventPublisher());  
		HttpSecurity http = new HttpSecurity(this.objectPostProcessor, authenticationBuilder, createSharedObjects());  
		WebAsyncManagerIntegrationFilter webAsyncManagerIntegrationFilter = new WebAsyncManagerIntegrationFilter();  
		webAsyncManagerIntegrationFilter.setSecurityContextHolderStrategy(this.securityContextHolderStrategy);  
		// @formatter:off  
		http  
			 .csrf(withDefaults())  
			 .addFilter(webAsyncManagerIntegrationFilter)  
			 .exceptionHandling(withDefaults())  
			 .headers(withDefaults())  
			 .sessionManagement(withDefaults())  
			 .securityContext(withDefaults())  
			 .requestCache(withDefaults())  
			 .anonymous(withDefaults())  
			 .servletApi(withDefaults())  
			 .apply(new DefaultLoginPageConfigurer<>());  
		http.logout(withDefaults());  
		// @formatter:on  
		applyCorsIfAvailable(http);  
		applyDefaultConfigurers(http);  
		return http;  
	}  

	//...

	private AuthenticationManager authenticationManager() throws Exception {  
	   return this.authenticationConfiguration.getAuthenticationManager();  
	}

	//...
  
}
```
- `authenticationBuilder.parentAuthenticationManager()` : AuthenticationManager 추출 작업
	```java
	package org.springframework.security.config.annotation.authentication.configuration;  
	  
	@Import(ObjectPostProcessorConfiguration.class)  
	public class AuthenticationConfiguration {  
	
		//...
		
	   public AuthenticationManager getAuthenticationManager() throws Exception {  
	      if (this.authenticationManagerInitialized) {  
	         return this.authenticationManager;  
	      }  
	      AuthenticationManagerBuilder authBuilder = this.applicationContext.getBean(AuthenticationManagerBuilder.class);  
	      if (this.buildingAuthenticationManager.getAndSet(true)) {  
	         return new AuthenticationManagerDelegator(authBuilder);  
	      }  
	      for (GlobalAuthenticationConfigurerAdapter config : this.globalAuthConfigurers) {  
	         authBuilder.apply(config);  
	      }  
	      this.authenticationManager = authBuilder.build();  
	      if (this.authenticationManager == null) {  
	         this.authenticationManager = getAuthenticationManagerBean();  
	      }  
	      this.authenticationManagerInitialized = true;  
	      return this.authenticationManager;  
	   }  
	
		//...
		
	}
	```
	- `applicationContext.getBean()` : spring container에 빈으로 등록된 것이 있는지 확인
	- `authBuilder.build()` : AuthenticationManager 생성
		```java
		package org.springframework.security.config.annotation.authentication.builders;  
		  
		public class AuthenticationManagerBuilder  
		      extends AbstractConfiguredSecurityBuilder<AuthenticationManager, AuthenticationManagerBuilder>  
		      implements ProviderManagerBuilder<AuthenticationManagerBuilder> {  
		
			//...
			
			@Override  
			protected ProviderManager performBuild() throws Exception {  
			   if (!isConfigured()) {  
				  this.logger.debug("No authenticationProviders and no parentAuthenticationManager defined. Returning null.");  
				  return null;   }  
			   ProviderManager providerManager = new ProviderManager(this.authenticationProviders,  
					 this.parentAuthenticationManager);  
			   if (this.eraseCredentials != null) {  
				  providerManager.setEraseCredentialsAfterAuthentication(this.eraseCredentials);  
			   }  
			   if (this.eventPublisher != null) {  
				  providerManager.setAuthenticationEventPublisher(this.eventPublisher);  
			   }  
			   providerManager = postProcess(providerManager);  
			   return providerManager;  
			}
			
			//...
			
		}
		```
		- `ProviderManager` 생성
- `new HttpSecurity(..., authenticationBuilder, ...)`
	```java
	package org.springframework.security.config.annotation.web.builders;  
	  
	public final class HttpSecurity extends AbstractConfiguredSecurityBuilder<DefaultSecurityFilterChain, HttpSecurity>  
	      implements SecurityBuilder<DefaultSecurityFilterChain>, HttpSecurityBuilder<HttpSecurity> {  
	
		//...
		
		public HttpSecurity(ObjectPostProcessor<Object> objectPostProcessor,  
			 AuthenticationManagerBuilder authenticationBuilder, Map<Class<?>, Object> sharedObjects) {  
		  super(objectPostProcessor);  
		  Assert.notNull(authenticationBuilder, "authenticationBuilder cannot be null");  
		  setSharedObject(AuthenticationManagerBuilder.class, authenticationBuilder);  
		  for (Map.Entry<Class<?>, Object> entry : sharedObjects.entrySet()) {  
			 setSharedObject((Class<Object>) entry.getKey(), entry.getValue());  
		  }  
		  ApplicationContext context = (ApplicationContext) sharedObjects.get(ApplicationContext.class);  
		  this.requestMatcherConfigurer = new RequestMatcherConfigurer(context);  
		}  
		
		//...
	  
	}
	```
	- `setSharedObject` : 전역적으로 `AuthenticaitonBuilder` 사용할 수 있도록 설정