# 개념
## SecurityBuilder
- 빌더 클래스
- 웹 보안을 구성하는 빈 객체와 설정클래스(`SecurityConfigurer` 타입)를 생성

ex) [[WebSecurity & HttpSecurity]]
## SecurityConfigurer
- Http 요청과 관련된 보안처리 담당하는 [[Security Filter]](모든 인증, 인가 요청 처리) 생성
- 여러 초기화 설정에 관여

> `SecurityBuilder`가 `SecurityConfigurer` 참조 (사용)
> `SecurityConfigurer`의 인증, 인가 초기화 작업은 `SeuciryBuilder`에 의해 실행
> ![](https://i.imgur.com/iAkElCu.png)

![](https://i.imgur.com/xZbItJb.png)
1. **빌더 클래스 생성**
	- 초기화 & 자동 보안이 일어나고 SecurityBuilder 생성
2. **설정 클래스 생성**
	- SecurityBuilder가 SecurityConfigurer 생성
3. **초기화 작업 진행**
	- SecurityConfigurer의 `init(B builder)`, `configure(B builder)` 메소드가 여러 초기화 작업 & 설정에 관여
	- 필터 생성 & 설정 => 인증 & 인과 관련 내용
### 예시
![](https://i.imgur.com/mxoWybh.png)
1. SecurityBuilder의 구현체 중 하나인 `HttpSecurity`
2. `HttpSecurity`가 설정 클래스인 `SecurityConfigurer` 들을 생성
3. `SecurityConfigurer`은 각각의 Filter를 가짐
	ex) `LogoutConfigurer`은 [[Logout]]Filter를 가짐
# 코드
## SecurityBuilder
```java
public interface SecurityBuilder<O> {  
  O build() throws Exception;  
}
```
`O build()` -> `SecurityBuilder`가 어떤 오브젝트를 만드는 인터페이스임을 알 수 있음
## SecurityConfigurer
```java
package org.springframework.security.config.annotation;  
  
public interface SecurityConfigurer<O, B extends SecurityBuilder<O>> {  
	void init(B builder) throws Exception;  
	
	void configure(B builder) throws Exception;  
}
```
`init(B builder)`, `configure(B builder)`를 통해 인증 & 인가 관련 초기화 설정, 빈 생성

### 1️⃣ HttpSecurityConfiguration 실행
- 자동 설정에 의해 `HttpSecurity`의 설정 클래스 실행
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
	
}
```
- `@Scope("prototype")` : 싱글톤이 아닌 `HttpSecurity`를 생성할 때마다 빈 객체 생성
	- prototype : 생성, 의존관계 주입, 초기화까지만 진행
- `csrf()` : csrf에 관련된 설정
	```java
	public HttpSecurity csrf(Customizer<CsrfConfigurer<HttpSecurity>> csrfCustomizer) throws Exception {  
		ApplicationContext context = getContext();  
		csrfCustomizer.customize(getOrApply(new CsrfConfigurer<>(context)));  
		return HttpSecurity.this;  
	}
	```
	- `CsrfConfigurer` : SecurityConfigurer를 상속받은 클래스
	- `getOrApply` : 설정 클래스를 적용
- `exceptionHandling`
	```java
	public HttpSecurity exceptionHandling(  
      Customizer<ExceptionHandlingConfigurer<HttpSecurity>> exceptionHandlingCustomizer) throws Exception {  
	   exceptionHandlingCustomizer.customize(getOrApply(new ExceptionHandlingConfigurer<>()));  
	   return HttpSecurity.this;  
	}
	```
	- `ExceptionHandlingConfigurer`에서 만듦
- 이하 설정 동문
- `return http` : 빈 객체를 생성
### 2️⃣ SpringBootWebSecurityConfiguration에서 빈 주입
```java
@Configuration(proxyBeanMethods = false)  
@ConditionalOnWebApplication(type = Type.SERVLET)  
class SpringBootWebSecurityConfiguration {  
	
	@Configuration(proxyBeanMethods = false)  
	@ConditionalOnDefaultWebSecurity  
	static class SecurityFilterChainConfiguration {  
	
	  @Bean  
	  @Order(SecurityProperties.BASIC_AUTH_ORDER)  
	  SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {  
		 http.authorizeHttpRequests((requests) -> requests.anyRequest().authenticated());  
		 http.formLogin(withDefaults());  
		 http.httpBasic(withDefaults());  
		 return http.build();  
	  }  
	}

	//...
}
```
- `SpringBootWebSecurityConfiguration`에서 파라미터로 http 빈 객체를 받아서 초기화
- `formLogin`, `httpBasic` 또한 configurer 클래스를 추가함
	```java
	public HttpSecurity formLogin(Customizer<FormLoginConfigurer<HttpSecurity>> formLoginCustomizer) throws Exception {  
	   formLoginCustomizer.customize(getOrApply(new FormLoginConfigurer<>()));  
	   return HttpSecurity.this;  
	}

	public HttpSecurity httpBasic(Customizer<HttpBasicConfigurer<HttpSecurity>> httpBasicCustomizer) throws Exception {  
	   httpBasicCustomizer.customize(getOrApply(new HttpBasicConfigurer<>()));  
	   return HttpSecurity.this;  
	}
	```
- `formLogin`, `httpBasic`을 추가하며 `DefulatLoginPageConfigurer`도 추가 (로그인 페이지)
![](https://i.imgur.com/vweBPhg.png)
### 3️⃣ http.build를 통해 초기화
- configurer에 있는 `init`, `configure`를 통해 초기화
```java
@Override  
public final O build() throws Exception {  
   if (this.building.compareAndSet(false, true)) {  
      this.object = doBuild();  
      return this.object;  
   }  
   throw new AlreadyBuiltException("This object has already been built");  
}

@Override  
protected final O doBuild() throws Exception {  
   synchronized (this.configurers) {  
      this.buildState = BuildState.INITIALIZING;  
      beforeInit();  
      init();  
      this.buildState = BuildState.CONFIGURING;  
      beforeConfigure();  
      configure();  
      this.buildState = BuildState.BUILDING;  
      O result = performBuild();  
      this.buildState = BuildState.BUILT;  
      return result;  
   }  
}
```
- 상태 : Initializing -> configuring -> building -> built

**init**
객체를 만들고 객체를 쓰기 위해 공유하는 작업 -> 초기화 작업
```java
private void init() throws Exception {  
   Collection<SecurityConfigurer<O, B>> configurers = getConfigurers();  
   for (SecurityConfigurer<O, B> configurer : configurers) {  
      configurer.init((B) this);  
   }  
   for (SecurityConfigurer<O, B> configurer : this.configurersAddedInInitializing) {  
      configurer.init((B) this);  
   }  
}
```
- configurer들의 init 메소드 재정의
> 초기 configurer의 수는 14개

```java
//재정의되지 않은 init
@Override  
public void init(B builder) throws Exception {  
}

//재정의된 init 예시 (SessionManagementConfigurer)
public void init(H http) {  
   SecurityContextRepository securityContextRepository = http.getSharedObject(SecurityContextRepository.class);  
   boolean stateless = isStateless();  
   if (securityContextRepository == null) {  
      if (stateless) {  
         http.setSharedObject(SecurityContextRepository.class, new RequestAttributeSecurityContextRepository());  
         this.sessionManagementSecurityContextRepository = new NullSecurityContextRepository();  
      }  
      else {  
         HttpSessionSecurityContextRepository httpSecurityRepository = new HttpSessionSecurityContextRepository();  
         httpSecurityRepository.setDisableUrlRewriting(!this.enableSessionUrlRewriting);  
         httpSecurityRepository.setAllowSessionCreation(isAllowSessionCreation());  
         AuthenticationTrustResolver trustResolver = http.getSharedObject(AuthenticationTrustResolver.class);  
         if (trustResolver != null) {  
            httpSecurityRepository.setTrustResolver(trustResolver);  
         }  
         this.sessionManagementSecurityContextRepository = httpSecurityRepository;  
         DelegatingSecurityContextRepository defaultRepository = new DelegatingSecurityContextRepository(  
               httpSecurityRepository, new RequestAttributeSecurityContextRepository());  
         http.setSharedObject(SecurityContextRepository.class, defaultRepository);  
      }  
   }  
   else {  
      this.sessionManagementSecurityContextRepository = securityContextRepository;  
   }  
   RequestCache requestCache = http.getSharedObject(RequestCache.class);  
   if (requestCache == null) {  
      if (stateless) {  
         http.setSharedObject(RequestCache.class, new NullRequestCache());  
      }  
   }  
   http.setSharedObject(SessionAuthenticationStrategy.class, getSessionAuthenticationStrategy(http));  
   http.setSharedObject(InvalidSessionStrategy.class, getInvalidSessionStrategy());  
}
```

**configure**
인증, 인가 관련 필터 & 필터를 위한 핸들러 객체 생성하는 작업 -> 설정 작업
```java
private void configure() throws Exception {  
   Collection<SecurityConfigurer<O, B>> configurers = getConfigurers();  
   for (SecurityConfigurer<O, B> configurer : configurers) {  
      configurer.configure((B) this);  
   }  
}
```
- configurer들의 configure 메소드 재정의
```java
//재정의된 configure 예시 (CsrfConfigurer)
@Override  
public void configure(H http) {  
   CsrfFilter filter = new CsrfFilter(this.csrfTokenRepository);  
   RequestMatcher requireCsrfProtectionMatcher = getRequireCsrfProtectionMatcher();  
   if (requireCsrfProtectionMatcher != null) {  
      filter.setRequireCsrfProtectionMatcher(requireCsrfProtectionMatcher);  
   }  
   AccessDeniedHandler accessDeniedHandler = createAccessDeniedHandler(http);  
   ObservationRegistry registry = getObservationRegistry();  
   if (!registry.isNoop()) {  
      ObservationMarkingAccessDeniedHandler observable = new ObservationMarkingAccessDeniedHandler(registry);  
      accessDeniedHandler = new CompositeAccessDeniedHandler(observable, accessDeniedHandler);  
   }  
   if (accessDeniedHandler != null) {  
      filter.setAccessDeniedHandler(accessDeniedHandler);  
   }  
   LogoutConfigurer<H> logoutConfigurer = http.getConfigurer(LogoutConfigurer.class);  
   if (logoutConfigurer != null) {  
      logoutConfigurer.addLogoutHandler(new CsrfLogoutHandler(this.csrfTokenRepository));  
   }  
   SessionManagementConfigurer<H> sessionConfigurer = http.getConfigurer(SessionManagementConfigurer.class);  
   if (sessionConfigurer != null) {  
      sessionConfigurer.addSessionAuthenticationStrategy(getSessionAuthenticationStrategy());  
   }  
   if (this.requestHandler != null) {  
      filter.setRequestHandler(this.requestHandler);  
   }  
   filter = postProcess(filter);  
   http.addFilter(filter);  
}
```