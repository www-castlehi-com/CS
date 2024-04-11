# HttpSecurity
- `HttpSecurityConfiguration`에서 생성 & 초기화
- 보안에 필요한 각 설정 클래스 & 필터 생성 -> **`SecurityFilterChain`** 빈 생성
![](https://i.imgur.com/mmqU1f6.png)
## SecurityFilterChain
[[Security Filter]] 묶음
![](https://i.imgur.com/ecTfOav.png)
- `boolean matches(HttpServletRequest request)`
	- 현재 SecurityFilterChain에 의해 처리되어야 하는지 여부 결정
		> 다중 보안의 경우, 여러 SecurityFilterChain이 있을 수도 있음

	- `true` : 현재 요청이 해당 필터 체인에 의해 처리되어야 함
	- `false` : 다른 필터 체인이나 로직에 의해 처리되어야 함
- `List<Filter> getFilters()`
	- 현재 SecurityFilterChain에 포함된 Filter 리스트를 반환
	- 어떤 필터들이 현재 필터 체인에 포함되어 있는지 확인
![](https://i.imgur.com/gILQlsX.png)
1. 클라이언트의 요청 (ex http://localhost:8080 에 접속)
2. 요청과 SecurityFilterChain이 가지고 있는 RequestMatcher로 매칭
3. 유저가 해당 SecurityFilterChain을 가지고 있다면 Filter 동작
4. 가지고 있지 않다면 해당 SecurityFilterChain은 건너뛰고 다른 SecurityFilterChain을 찾음
5. FilterChain 내 필터들을 모두 실행시키면 서블릿으로 전달
# WebSecurity
- `WebSecurityConfiguration`에서 생성 & 초기화
- `HttpSecurity`에서 생성한 `SecurityFilterChain` 빈을 `SecurityBuilder`에 저장
- `build()` 시, SecurityBuilder에서 `SecurityFilterChain`을 꺼내어 **`FilterChainProxy`** 생성자에게 전달
## FilterChainProxy
모든 요청에 대한 정보 존재

![](https://i.imgur.com/bIqVRrw.png)
- HttpSecurity에서 만든 SecurityFilterChain  빈을 `SecurityBuilder`에 저장
- `build()` 시, `FilterChainproxy`에 ArrayList 형태로 저장
# 순서
## 1️⃣ HttpSecurityConfiguration
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
`HttpSecurity` 빈 생성
## 2️⃣ WebSecurityConfiguration
```java
package org.springframework.security.config.annotation.web.configuration;  
  
public class WebSecurityConfiguration implements ImportAware, BeanClassLoaderAware {  

	//...
   
   @Autowired(required = false)  
	public void setFilterChainProxySecurityConfigurer(ObjectPostProcessor<Object> objectPostProcessor,  
	      ConfigurableListableBeanFactory beanFactory) throws Exception {  
	   this.webSecurity = objectPostProcessor.postProcess(new WebSecurity(objectPostProcessor));  
	   if (this.debugEnabled != null) {  
	      this.webSecurity.debug(this.debugEnabled);  
	   }  
	   List<SecurityConfigurer<Filter, WebSecurity>> webSecurityConfigurers = new AutowiredWebSecurityConfigurersIgnoreParents(  
	         beanFactory)  
	      .getWebSecurityConfigurers();  
	   webSecurityConfigurers.sort(AnnotationAwareOrderComparator.INSTANCE);  
	   Integer previousOrder = null;  
	   Object previousConfig = null;  
	   for (SecurityConfigurer<Filter, WebSecurity> config : webSecurityConfigurers) {  
	      Integer order = AnnotationAwareOrderComparator.lookupOrder(config);  
	      if (previousOrder != null && previousOrder.equals(order)) {  
	         throw new IllegalStateException("@Order on WebSecurityConfigurers must be unique. Order of " + order  
	               + " was already used on " + previousConfig + ", so it cannot be used on " + config + " too.");  
	      }  
	      previousOrder = order;  
	      previousConfig = config;  
	   }  
	   for (SecurityConfigurer<Filter, WebSecurity> webSecurityConfigurer : webSecurityConfigurers) {  
	      this.webSecurity.apply(webSecurityConfigurer);  
	   }  
	   this.webSecurityConfigurers = webSecurityConfigurers;  
	} 

	//...
}
```
- `new WebSecurity()` : WebSecurity 생성 (HttpSecurity와 관련된 설정을 필요로 하기 때문에 HttpSecurity 먼저 생성)
## 3️⃣ SecurityFilterChain 생성
```java
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
```
`SecurityFilterChain` 생성
### HttpSecurity 빌드
```java
package org.springframework.security.config.annotation.web.builders;  
  
public final class HttpSecurity extends AbstractConfiguredSecurityBuilder<DefaultSecurityFilterChain, HttpSecurity>  

	//...
	
	private RequestMatcher requestMatcher = AnyRequestMatcher.INSTANCE;

	//...

	@Override  
	protected DefaultSecurityFilterChain performBuild() {  
	   ExpressionUrlAuthorizationConfigurer<?> expressionConfigurer = getConfigurer(  
	         ExpressionUrlAuthorizationConfigurer.class);  
	   AuthorizeHttpRequestsConfigurer<?> httpConfigurer = getConfigurer(AuthorizeHttpRequestsConfigurer.class);  
	   boolean oneConfigurerPresent = expressionConfigurer == null ^ httpConfigurer == null;  
	   Assert.state((expressionConfigurer == null && httpConfigurer == null) || oneConfigurerPresent,  
	         "authorizeHttpRequests cannot be used in conjunction with authorizeRequests. Please select just one.");  
	   this.filters.sort(OrderComparator.INSTANCE);  
	   List<Filter> sortedFilters = new ArrayList<>(this.filters.size());  
	   for (Filter filter : this.filters) {  
	      sortedFilters.add(((OrderedFilter) filter).filter);  
	   }  
	   return new DefaultSecurityFilterChain(this.requestMatcher, sortedFilters);  
	}

	//...
}
```
- filter를 이용해 filter chain으로 만드는 build 과정
	1. `DefaultSecurityFilterChain` 생성
	2. `requestMatcher`, 필터 목록을 매개변수로 전달
- `requestMathcer = AnyRequestMathcer.INSTANCE` : 모든 요청에 대해서 그 요청이 체인에 적합한지 확인
## 4️⃣ WebSecurityConfiguraiton에 FilterChain 등록
```java
package org.springframework.security.config.annotation.web.configuration;  
  
public class WebSecurityConfiguration implements ImportAware, BeanClassLoaderAware {  

	//...

	@Autowired(required = false)  
	void setFilterChains(List<SecurityFilterChain> securityFilterChains) {  
	   this.securityFilterChains = securityFilterChains;  
	}

	//...
}
```
- DI에 의해서 filter chain 주입
- `List<SecurityFilterChain>` : 여러 개의 filter chain이 생길 수 있음
## 5️⃣ WebSecurity에 FilterChain 저장
```java
package org.springframework.security.config.annotation.web.configuration;  
  
public class WebSecurityConfiguration implements ImportAware, BeanClassLoaderAware {  

	//...

	@Bean(name = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME)  
	public Filter springSecurityFilterChain() throws Exception {  
	   boolean hasFilterChain = !this.securityFilterChains.isEmpty();  
	   if (!hasFilterChain) {  
	      this.webSecurity.addSecurityFilterChainBuilder(() -> {  
	         this.httpSecurity.authorizeHttpRequests((authorize) -> authorize.anyRequest().authenticated());  
	         this.httpSecurity.formLogin(Customizer.withDefaults());  
	         this.httpSecurity.httpBasic(Customizer.withDefaults());  
	         return this.httpSecurity.build();  
	      });  
	   }  
	   for (SecurityFilterChain securityFilterChain : this.securityFilterChains) {  
	      this.webSecurity.addSecurityFilterChainBuilder(() -> securityFilterChain);  
	   }  
	   for (WebSecurityCustomizer customizer : this.webSecurityCustomizers) {  
	      customizer.customize(this.webSecurity);  
	   }  
	   return this.webSecurity.build();  
	}

	//...
}
```

```java
package org.springframework.security.config.annotation.web.builders;  
  
 public final class WebSecurity extends AbstractConfiguredSecurityBuilder<Filter, WebSecurity>  
      implements SecurityBuilder<Filter>, ApplicationContextAware, ServletContextAware { 

	//...

	public WebSecurity addSecurityFilterChainBuilder(  
	  SecurityBuilder<? extends SecurityFilterChain> securityFilterChainBuilder) {  
	   this.securityFilterChainBuilders.add(securityFilterChainBuilder);  
	   return this;
	}

	//...
}
```
- `this.webSecurity.addSecurityFilterChainBuilder(() -> securityFilterChain)` : `FilterChainProxy`를 만들기 위해 webSecurity에 filter chain 저장
- `this.securityFilterChainBuilders.add(securityFilterChainBuilder)` : 여러 개 저장 가능
### WebSecurity 빌드
```java
package org.springframework.security.config.annotation.web.builders;  
  
 public final class WebSecurity extends AbstractConfiguredSecurityBuilder<Filter, WebSecurity>  
      implements SecurityBuilder<Filter>, ApplicationContextAware, ServletContextAware { 

	//...

	@Override  
	protected Filter performBuild() throws Exception {  
	   Assert.state(!this.securityFilterChainBuilders.isEmpty(),  
	         () -> "At least one SecurityBuilder<? extends SecurityFilterChain> needs to be specified. "  
	               + "Typically this is done by exposing a SecurityFilterChain bean. "  
	               + "More advanced users can invoke " + WebSecurity.class.getSimpleName()  
	               + ".addSecurityFilterChainBuilder directly");  
	   int chainSize = this.ignoredRequests.size() + this.securityFilterChainBuilders.size();  
	   List<SecurityFilterChain> securityFilterChains = new ArrayList<>(chainSize);  
	   List<RequestMatcherEntry<List<WebInvocationPrivilegeEvaluator>>> requestMatcherPrivilegeEvaluatorsEntries = new ArrayList<>();  
	   for (RequestMatcher ignoredRequest : this.ignoredRequests) {  
	      WebSecurity.this.logger.warn("You are asking Spring Security to ignore " + ignoredRequest  
	            + ". This is not recommended -- please use permitAll via HttpSecurity#authorizeHttpRequests instead.");  
	      SecurityFilterChain securityFilterChain = new DefaultSecurityFilterChain(ignoredRequest);  
	      securityFilterChains.add(securityFilterChain);  
	      requestMatcherPrivilegeEvaluatorsEntries  
	         .add(getRequestMatcherPrivilegeEvaluatorsEntry(securityFilterChain));  
	   }  
	   for (SecurityBuilder<? extends SecurityFilterChain> securityFilterChainBuilder : this.securityFilterChainBuilders) {  
	      SecurityFilterChain securityFilterChain = securityFilterChainBuilder.build();  
	      securityFilterChains.add(securityFilterChain);  
	      requestMatcherPrivilegeEvaluatorsEntries  
	         .add(getRequestMatcherPrivilegeEvaluatorsEntry(securityFilterChain));  
	   }  
	   if (this.privilegeEvaluator == null) {  
	      this.privilegeEvaluator = new RequestMatcherDelegatingWebInvocationPrivilegeEvaluator(  
	            requestMatcherPrivilegeEvaluatorsEntries);  
	   }  
	   FilterChainProxy filterChainProxy = new FilterChainProxy(securityFilterChains);  
	   if (this.httpFirewall != null) {  
	      filterChainProxy.setFirewall(this.httpFirewall);  
	   }  
	   if (this.requestRejectedHandler != null) {  
	      filterChainProxy.setRequestRejectedHandler(this.requestRejectedHandler);  
	   }  
	   else if (!this.observationRegistry.isNoop()) {  
	      CompositeRequestRejectedHandler requestRejectedHandler = new CompositeRequestRejectedHandler(  
	            new ObservationMarkingRequestRejectedHandler(this.observationRegistry),  
	            new HttpStatusRequestRejectedHandler());  
	      filterChainProxy.setRequestRejectedHandler(requestRejectedHandler);  
	   }  
	   filterChainProxy.setFilterChainDecorator(getFilterChainDecorator());  
	   filterChainProxy.afterPropertiesSet();  
	  
	   Filter result = filterChainProxy;  
	   if (this.debugEnabled) {  
	      this.logger.warn("\n\n" + "********************************************************************\n"  
	            + "**********        Security debugging is enabled.       *************\n"  
	            + "**********    This may include sensitive information.  *************\n"  
	            + "**********      Do not use in a production system!     *************\n"  
	            + "********************************************************************\n\n");  
	      result = new DebugFilter(filterChainProxy);  
	   }  
	  
	   this.postBuildAction.run();  
	   return result;  
	}

	//...
}
```
- `securityFilterChains.add(securityFilterChain);` : `securityFilterChains`에 `DefaultSecurityFilterChain` 추가
- `FilterChainProxy filterChainProxy = new FilterChainProxy(securityFilterChains);`
 : `securityFilterChains`를 주입 받는 빈이 됨

> 💡 `FilterChainProxy`는 Filter 타입이므로 `WebSecurityConfiguration`의 `springSecurityFilterChain`에서 Filter로 반환