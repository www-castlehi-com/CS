# Filter
- 웹 애플리케이션에서 클라언트의 요청 (`HttpServletRequest`)과 서버의 응답 (`HttpServletResponse`)을 가공, 검사
- 클라이언트의 요청이 서블릿에 도달하기 전이나 서블릿이 응답을 클라이언트에게 보내기 전에 특정 작업 수행
- **서블릿 컨테이너 (WAS)**에서 생성, 실행, 종료
## WAS 내 동작
![](https://i.imgur.com/j8MIyk0.png)
- FilterChain의 Filter 모두 실행
- Servlet으로 전달
## Filter 구조
![](https://i.imgur.com/9xhcVGX.png)
### init
- 필터 초기화 시 필요한 작업 수행
### doFilter
- `chain.doFilter(request, response)` 메소드 전후로 처리 가능
- 전처리 : 요청 처리 전에 수행할 작업으로 **ServletRequest** 수정
- 후처리 : 응답 처리 후에 수행할 작업으로 **ServletResponse** 수정
### destroy
- 필터가 제거될 때 필요한 작업 수행
## DelegatingFilterProxy
### 개념
- 스프링에서 사용되는 서블릿 필터
- 서블릿 컨테이너와 스프링 애플리케이션 컨텍스트 간의 연결고리
- 서블릿 필터 기능 + 스프링 의존성 주입 & 빈 관리
- `springSecurityFilterChain` 이름으로 생성된 빈을 ApplicationContext에서 찾아 요청 위임
- 실제 보안 처리 X
### 순서
![](https://i.imgur.com/atlYpsL.png)
1. Client가 요청을 함
2. `DelegatingFilterProxy`가 Spring IOC Container에서 `SpringSecurityFilterChain`의 이름을 가진 빈 찾음
3. 해당 빈은 Filter를 구현하고, `DelegatingFilterProxy`가 클라이언트의 요청을 해당 빈에 위임
### 코드
```java
package org.springframework.boot.autoconfigure.security.servlet;  
  
public class SecurityFilterAutoConfiguration {  
  
   private static final String DEFAULT_FILTER_NAME = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME;  

	//public static final String DEFAULT_FILTER_NAME = "springSecurityFilterChain";
  
   @Bean  
   @ConditionalOnBean(name = DEFAULT_FILTER_NAME)  
   public DelegatingFilterProxyRegistrationBean securityFilterChainRegistration(  
         SecurityProperties securityProperties) {  
      DelegatingFilterProxyRegistrationBean registration = new DelegatingFilterProxyRegistrationBean(  
            DEFAULT_FILTER_NAME);  
      registration.setOrder(securityProperties.getFilter().getOrder());  
      registration.setDispatcherTypes(getDispatcherTypes(securityProperties));  
      return registration;  
   }  

	public DelegatingFilterProxyRegistrationBean(String targetBeanName,  
	      ServletRegistrationBean<?>... servletRegistrationBeans) {  
	   super(servletRegistrationBeans);  
	   Assert.hasLength(targetBeanName, "TargetBeanName must not be null or empty");  
	   this.targetBeanName = targetBeanName;  
	   setName(targetBeanName);  
	}
	//...
}
```
- `DEFAULT_FILTER_NAME` : springSecurityFilterChain의 이름을 가진 빈을 찾음
- `DelegatingFilterProxyRegistrationBean` : `DelegatingFilterProxy` 생성
```java
package org.springframework.boot.web.servlet;  
  
public abstract class AbstractFilterRegistrationBean<T extends Filter> extends DynamicRegistrationBean<Dynamic> {  
	//...

	@Override  
	protected Dynamic addRegistration(String description, ServletContext servletContext) {  
	   Filter filter = getFilter();  
	   return servletContext.addFilter(getOrDeduceName(filter), filter);  
	}

	@Override  
	public DelegatingFilterProxy getFilter() {  
	   return new DelegatingFilterProxy(this.targetBeanName, getWebApplicationContext()) {  
	  
	      //...
	  
	   };  
	}

	//...
}
```
- `new DelegatingFilterProxy()` : `DelegatingFilterProxy` 생성
- `targetBeanName` : 스프링 IOC 컨테이너로부터 해당 이름을 가진 빈을 찾음 (= springSecurityFilterChain)
- `servletContext.addFilter()` : 스프링 컨테이너가 아닌 서블릿에 필터 추가

> 💡 `DelegatingFilterProxy`는 스프링에서 활용하지만 실질적으로 서블릿 컨텍스트에 추가되는 필터
## FilterChainProxy
### 개념
- `springSecurityFilterChain`의 이름으로 생성되는 필터 빈
- `DelegatingFilterProxy`로 부터 요청 위임 & 보안 처리
- 내부적으로 하나 이상의 SecurityFilterChain 객체를 가짐
- 요청 URL을 기준으로 적절한 SecurityFilterChain을 선택해 필터 호출
	>💡0번 ~ 15번의 16개의 Filter를 가지고 있음
- `HttpSecurity`를 통해 API 추가 시 관련 필터 추가
- 사용자의 요청을 필터 순서대로 호출하여 보안 기능 동작
- 필요 시 직접 필터 생성
### 순서
![](https://i.imgur.com/fb4kic0.png)
1. Client가 요청을 함
2. `DelegatingFilterProxy`가 ApplicationContext를 찾음
3. 클라이언트의 요청을 `FilterChainProxy`에게 위임
4. `FilterChainProxy`는 `SecurityFilterChain`의 필터 목록을 통해 요청 처리
5. Filter처리 종료 후, Spring MVC Servlet으로 전달
### 코드
```java
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
```
- `@Bean(name = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME)` : springSecurityFilterChain 이름을 가진 webSecurity 빈 생성
	> 💡 `DEFAULT_FILTER_NAME`으로 생성하기 때문에 `DelegatingFilterProxy`가 해당 이름을 가진 `FilterChainProxy`를 찾을 수 있음
## 코드
### 1️⃣ 위임할 필터 찾기
```java
package org.springframework.web.filter;  
  
public class DelegatingFilterProxy extends GenericFilterBean {  

	// ... 
	
	@Nullable  
	private String targetBeanName;  

	//...
	
	@Override  
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)  
		 throws ServletException, IOException {  
	
	  // Lazily initialize the delegate if necessary.  
	  Filter delegateToUse = this.delegate;  
	  if (delegateToUse == null) {  
		 synchronized (this.delegateMonitor) {  
			delegateToUse = this.delegate;  
			if (delegateToUse == null) {  
			   WebApplicationContext wac = findWebApplicationContext();  
			   if (wac == null) {  
				  throw new IllegalStateException("No WebApplicationContext found: " +  
						"no ContextLoaderListener or DispatcherServlet registered?");  
			   }  
			   delegateToUse = initDelegate(wac);  
			}  
			this.delegate = delegateToUse;  
		 }  
	  }  
	
	  // Let the delegate perform the actual doFilter operation.  
	  invokeDelegate(delegateToUse, request, response, filterChain);  
	}  

	//...

	protected Filter initDelegate(WebApplicationContext wac) throws ServletException {  
		String targetBeanName = getTargetBeanName();  
		Assert.state(targetBeanName != null, "No target bean name set");  
		Filter delegate = wac.getBean(targetBeanName, Filter.class);  
		if (isTargetFilterLifecycle()) {  
		  delegate.init(getFilterConfig());  
		}  
		return delegate;  
	}
	
	protected void invokeDelegate(  
	      Filter delegate, ServletRequest request, ServletResponse response, FilterChain filterChain)  
	      throws ServletException, IOException {  
	  
	   delegate.doFilter(request, response, filterChain);  
	}

	//...
	
}
```
- `targetBeanName` : springSecurityFilterChain
- `delegateToUse` : 위임할 필터 찾기
- `WebApplicationContext` : 스프링 컨테이너
	> 💡 스프링 컨테이너로부터 위임할 필터 찾음
- `wac.getBean()` : `CompositeFilter`에 2개의 필터를 가지고 번갈아가며 호출
- `invokeDelegate` : 찾은 필터 호출
### 2️⃣ FilterChainProxy 호출
```java
package org.springframework.security.web;  
  
public class FilterChainProxy extends GenericFilterBean {  

	//...

	@Override  
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)  
		 throws IOException, ServletException {  
		boolean clearContext = request.getAttribute(FILTER_APPLIED) == null;  
		if (!clearContext) {  
		 doFilterInternal(request, response, chain);  
		 return;      }  
		try {  
		 request.setAttribute(FILTER_APPLIED, Boolean.TRUE);  
		 doFilterInternal(request, response, chain);  
		}  
		catch (Exception ex) {  
		 Throwable[] causeChain = this.throwableAnalyzer.determineCauseChain(ex);  
		 Throwable requestRejectedException = this.throwableAnalyzer  
			.getFirstThrowableOfType(RequestRejectedException.class, causeChain);  
		 if (!(requestRejectedException instanceof RequestRejectedException)) {  
			throw ex;  
		 }  
		 this.requestRejectedHandler.handle((HttpServletRequest) request, (HttpServletResponse) response,  
			   (RequestRejectedException) requestRejectedException);  
		}  
		finally {  
		 this.securityContextHolderStrategy.clearContext();  
		 request.removeAttribute(FILTER_APPLIED);  
		}  
	}  

	private void doFilterInternal(ServletRequest request, ServletResponse response, FilterChain chain)  
	      throws IOException, ServletException {  
	   FirewalledRequest firewallRequest = this.firewall.getFirewalledRequest((HttpServletRequest) request);  
	   HttpServletResponse firewallResponse = this.firewall.getFirewalledResponse((HttpServletResponse) response);  
	   List<Filter> filters = getFilters(firewallRequest);  
	   if (filters == null || filters.size() == 0) {  
	      if (logger.isTraceEnabled()) {  
	         logger.trace(LogMessage.of(() -> "No security for " + requestLine(firewallRequest)));  
	      }  
	      firewallRequest.reset();  
	      this.filterChainDecorator.decorate(chain).doFilter(firewallRequest, firewallResponse);  
	      return;   }  
	   if (logger.isDebugEnabled()) {  
	      logger.debug(LogMessage.of(() -> "Securing " + requestLine(firewallRequest)));  
	   }  
	   FilterChain reset = (req, res) -> {  
	      if (logger.isDebugEnabled()) {  
	         logger.debug(LogMessage.of(() -> "Secured " + requestLine(firewallRequest)));  
	      }  
	      // Deactivate path stripping as we exit the security filter chain  
	      firewallRequest.reset();  
	      chain.doFilter(req, res);  
	   };  
	   this.filterChainDecorator.decorate(reset, filters).doFilter(firewallRequest, firewallResponse);  
	}

	//...
	
}
```
- `getFilters()` :  DefaultSecurityFilterChain이 가지고 있는 16개의 필터를 가져옴
- `chain.doFilter(req, res)` : 가상 필터 체인에서 필터 호출
### 3️⃣ 스프링에서 사용할 가상 필터 체인에서 필터 호출
```java
package org.springframework.security.web;  
  
public class FilterChainProxy extends GenericFilterBean {  

	//...

	private static final class VirtualFilterChain implements FilterChain {  

		//...
		
		@Override  
		public void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException {  
		  if (this.currentPosition == this.size) {  
			 this.originalChain.doFilter(request, response);  
			 return;      }  
		  this.currentPosition++;  
		  Filter nextFilter = this.additionalFilters.get(this.currentPosition - 1);  
		  if (logger.isTraceEnabled()) {  
			 String name = nextFilter.getClass().getSimpleName();  
			 logger.trace(LogMessage.format("Invoking %s (%d/%d)", name, this.currentPosition, this.size));  
		  }  
		  nextFilter.doFilter(request, response, this);  
		}  
	
	}
}
```
- `nextFilter.doFilter()` : 16개의 필터를 순서대로 호출
### 4️⃣ - 1️⃣ 서블릿으로 전달
### 4️⃣ - 2️⃣ 인증 필터
- 인증을 받지 못했다면 16개의 필터 중 마지막 필터인 `AuthorizationFilter`를 지날 수 없음
- 인증을 받을 수 있는 페이지로 리다이렉션 (/login)
- 위임할 필터를 찾음
	- `DelegatingFilterProxy`의 `doFilter`에서 `delegateToUse`를 다시 호출
	- 이미 초기화 되었기 때문에 초기화 과정은 건너뛰고 `invokeDelegate()` 호출
> 💡 `DelegatingFilterProxy`, `FilterChainProxy`는 모든 요청에 대해서 항상 거쳐야 함