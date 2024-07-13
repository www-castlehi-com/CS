## 개요
- 인증된 사용자가 요청 자원에 접근할 수 있는지 여부를 결정하는 인터페이스
- 인증된 사용자의 [[Authorization]] 정보와 요청 자원의 보안 요구 사항을 기반으로 권한 부여 결정을 내림
- Spring Security의 [[요청 기반 권한 부여]], [[메서드 기반 권한 부여]]에서 호출되며 최종 액세스 제어 결정 수행
- 권한 부여 처리는 AuthorizationFilter를 통해 이루어지며 AuthorizationFilter는 AuthorizationManager를 호출하여 권한 부여 결정
## 구조
![](https://i.imgur.com/BoIy5Wh.png)
- `check()`
	- 권한 부여 결정을 내릴 때 필요한 모든 관련 정보 ([[Authentication]] 객체, 권한 정보, 요청 정보, 호출 정보 등)가 전달
	- 액세스가 허용되면 true를 포함하는 `AuthorizationDecision`, 거부되면 false를 포함하는 `AuthorizationDecision`, 결정을 내릴 수 없는 경우 `null` 반환
- `verify()` 
	- `check()` 호출 시 결과가 false를 포함하는 `AuthorizationDecision`인 경우 `AccessDeniedException`을 던짐
## 계층 구조
![](https://i.imgur.com/CnoWMa2.png)
### 1️⃣ 요청 기반 권한 부여 관리자
#### RequestMatcherDelegatingAuthorizationManager
- 클라이언트 요청 시 가장 적합한 구현체를 선택하고 위임하는 구현체
#### AuthenticatedAuthorizationManger
- 인증된 사용자에게 허용

**구조**
![](https://i.imgur.com/Oeqnkpk.png)
- `FullyAuthenticatedAuthorizationStrategy` : 익명 인증 및 기억하기 인증이 아닌지 검사
- `RememberMeAuthorizationStrategy` : 기억하기 인증인지 검사
- `AuthenticatedAuthorizationStrategy` : 인증된 사용자인지 검사
- `AnonymousAuthenticationStrategy` : 익명 사용자인지 검사

**흐름**
![](https://i.imgur.com/9rXckKs.png)
1. Client의 요청
2. RequestMatcherDelegatingAuthorizationManager가 해당 요청과 일치되는 패턴 검색
3. 요청 URL과 매치가 되는지 확인
	- 매치가 되지 않는다면 AuthorizationDecision이 `AccessDeniedException` 발생
4. 매치가 된다면 AuthenticatedAuthorizationManager가 처리를 위임받고 FullyAuthenticatedAuthorizationStrategy를 이용해 인가 처리
	- 인가가 허가되면 요청을 수락하고 아니라면 `AccessDeniedException` 발생
#### AuthorityAuthorizationManager
- 인증 & 특정 권한을 가진 사용자에게 허용

**구조**
![](https://i.imgur.com/bhFqT8x.png)
![](https://i.imgur.com/r1AAGgy.png)
- AuthorityAuthorizationManager는 AuthoritiesAuthorizationManager에게 권한 여부 결정 위임

**흐름**
![](https://i.imgur.com/jEJwLaz.png)

#### WebExpressionAuthorizationManager
- 웹 표현식을 확인하고 허용
### 2️⃣ 메서드 기반 권한 부여 관리자
- [[Spring/AOP]]로 구성

**초기화**
![](https://i.imgur.com/EBihEj8.png)
1. 스프링은 초기화 시 생성되는 전체 빈을 검사하면서 빈이 가진 메소드 중에서 보안이 설정된 메소드가 있는지  탐색
2. 보안이 설정된 메소드가 있다면 빈의 프록시 객체를 자동으로 생성 (Cglib 방식)
	```java
	package org.springframework.aop.framework;  
	  
	public class DefaultAopProxyFactory implements AopProxyFactory, Serializable {  
	
		//...
	
	   @Override  
	   public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {  
	      if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {  

			//...

	         if (targetClass.isInterface() || Proxy.isProxyClass(targetClass) || ClassUtils.isLambdaClass(targetClass)) {  
	            return new JdkDynamicAopProxy(config);  
	         }  
	         return new ObjenesisCglibAopProxy(config);  
	      }  
	      else {  
	         return new JdkDynamicAopProxy(config);  
	      }  
	   }  
	
		//...
	
	}
	```
	-  `JdkDynamicAopProxy` : target 객체가 interface거나, proxy거나 lambda 클래스일 경우
	- `ObjenesisCglibAopProxy` : 그 외
3.  보안이 설정된 메소드에는 인가처리 기능을 하는 Advice 등록
	```java
	package org.springframework.aop.framework.autoproxy;  
	  
	public abstract class AbstractAutoProxyCreator extends ProxyProcessorSupport  
	      implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware {  
	
		//...
	
		@Override  
		public Class<?> determineBeanType(Class<?> beanClass, String beanName) {  
		   Object cacheKey = getCacheKey(beanClass, beanName);  
		   Class<?> proxyType = this.proxyTypes.get(cacheKey);  
		   if (proxyType == null) {  
		      
		      //...
		      
		      if (specificInterceptors != DO_NOT_PROXY) {  
		         this.advisedBeans.put(cacheKey, Boolean.TRUE);  
		         proxyType = createProxyClass(beanClass, beanName, specificInterceptors, targetSource);  
		         this.proxyTypes.put(cacheKey, proxyType);  
		      }  
		   }  
		   return (proxyType != null ? proxyType : beanClass);  
		}
	
		//...
	  
	}
	```
4. 빈 참조 시 실제 빈이 아닌 프록시 빈 객체를 참조하도록 처리
5. 초기화 과정 종료
6. 사용자는 프록시 객체를 통해 메소드를 호출하고 프록시 객체는 Advice가 등록된 메서드가 있다면 호출하여 작동시킴
7. Advice는 메소드 진입 전 인가 처리를 하고, 인가 처리가 승인될 시 실제 객체의 메소드를 호출함 (거부 시 예외 발생)

**인터셉터**
- `AuthorizationManagerBeforeMethodInterceptor` : `@PreAuthorize`, `@Secured`, `JSR-250`에 설정되어 Authentication이 보안 메서드를 호출할 수 있는지 여부를 결정
- `AuthorizationManagerAfterMethodInterceptor` : `@PostAuthorize`에 설정되어 Authentication이 보안 메서드의 반환 결과에 접근할 수 있는지 여부를 결정
- `PreFilterAuthorizationMethodInterceptor` : `@PreFilter`에 설정되어 메소드 인자를 필터링
- `PostFilterAuthorizationMethodInterceptor` : `@PostFilter`에 설정되어 반환된 객체를 필터링

```java
package org.springframework.security.authorization.method;  
  
public enum AuthorizationInterceptorsOrder {  
  
   FIRST(Integer.MIN_VALUE),  
  
   PRE_FILTER,  
  
   PRE_AUTHORIZE,  
  
   SECURED,  
  
   JSR250,  
  
   POST_AUTHORIZE,  
  
   POST_FILTER,  
  
   LAST(Integer.MAX_VALUE);  
  
}
```
- AOP 어드바이저 체인에서 특정 위치 차지
- `@Transactional`과 `@PostAuthorize`가 함께 사용될 때 트랜잭션이 열려있지 않아 `AccessDeniedException`이 발생해도 롤백이 일어나지 않음
- `@EnableTransactionManagement(order = 0)`과 함께 사용 시 `@Transactional`이 보안 어노테이션보다 먼저 실행됨

#### PreAuthorizeAuthorizationManager
- `@PreAuthorize`를 사용하여 메소드 실행 전에 권한 검사

**흐름**
![](https://i.imgur.com/hlk3Mpc.png)
1. Client의 메소드 호출
2. Proxy가 이를 받아 `DefaultAdvisorChainFactory`를 이용하여 해당 클래스와 메소드의 정보를 제공하고 `Advice`를 반환받음
3. `Advice`를 호출 (`AuthorizationManagerBeforeMethodInterceptor`)
4. 인터셉터 클래스가 참조 중인 `PreAuthorizeAuthorizationManager`의 `check()` 메소드를 이용하여 권한 심사
	- `MethodSecurityExpressionHandler`를 이용하여 메소드의 표현식을 처리
		- `MethodSecurityEvalutionContext`가 이를 평가
5. 권한 심사 결과인 `AuthorizationDecision`가 true이면 요청을 처리하고 false이면 `AccessDeniedException` 발생
#### PostAuthorizeAuthorizationManager
- `@PostAuthorize`를 사용하여 메소드 실행 후 결과 검사

**흐름**
![](https://i.imgur.com/5xhTxfr.png)
1. Client의 메소드 호출
2. Proxy가 이를 받아 `DefaultAdvisorChainFactory`를 이용하여 해당 클래스와 메소드의 정보를 제공하고 `Advice`를 반환받음
3. `Advice`를 호출 (`AuthorizationManagerBeforeMethodInterceptor`)
4. 결과값을 반환받은 후 해당 결과값에 대한 인가 확인
	- `MethodSecurityExpressionHandler`를 이용하여 메소드의 표현식을 처리
		- `MethodSecurityEvalutionContext`가 이를 평가
5. 권한 심사 결과인 `AuthorizationDecision`가 true이면 요청을 처리하고 false이면 `AccessDeniedException` 발생
#### Jsr250AuthorizationManager
- JSR-250 어노테이션 (`@RolesAllowed`, `@DenyAll`, `@PermitAll`)을 사용하여 권한 관리
#### SecuredAuthorizationManager
- `@Secured`를 사용하여 권한 검사
## 코드
```java
package org.springframework.security.authorization;  
  
public interface AuthorizationManager<T> {  
  
	default void verify(Supplier<Authentication> authentication, T object) {  
		AuthorizationDecision decision = check(authentication, object);  
		if (decision != null && !decision.isGranted()) {  
		 throw new AccessDeniedException("Access Denied");  
		}  
	}
	
	//...
}
```
- `check(...)` : AuthorizationManager 구현체에게 권한 검사 위임
	```java
	package org.springframework.security.web.access.intercept;  
	  
	public final class RequestMatcherDelegatingAuthorizationManager implements AuthorizationManager<HttpServletRequest> {  
	
		//...
	
		public AuthorizationDecision check(Supplier<Authentication> authentication, HttpServletRequest request) {  
		  if (this.logger.isTraceEnabled()) {  
			 this.logger.trace(LogMessage.format("Authorizing %s", request));  
		  }  
		  for (RequestMatcherEntry<AuthorizationManager<RequestAuthorizationContext>> mapping : this.mappings) {  
		
			 RequestMatcher matcher = mapping.getRequestMatcher();  
			 MatchResult matchResult = matcher.matcher(request);  
			 if (matchResult.isMatch()) {  
				AuthorizationManager<RequestAuthorizationContext> manager = mapping.getEntry();  
				if (this.logger.isTraceEnabled()) {  
				   this.logger.trace(LogMessage.format("Checking authorization on %s using %s", request, manager));  
				}  
				return manager.check(authentication,  
					  new RequestAuthorizationContext(request, matchResult.getVariables()));  
			 }  
		  }  
		  if (this.logger.isTraceEnabled()) {  
			 this.logger.trace(LogMessage.of(() -> "Denying request since did not find matching RequestMatcher"));  
		  }  
		  return DENY;  
		}  
	  
	  //...
	  
	}
	```
	- `for(...) ~ check(...)` : 요청 기반 권한 부여 관리자 구현체를 순회하며 적합한 구현체에게 위임	
```java
package org.springframework.security.web.access.intercept;  
  
public class AuthorizationFilter extends GenericFilterBean {  

	//...

	@Override  
	public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain)  
		 throws ServletException, IOException {  
	
	  HttpServletRequest request = (HttpServletRequest) servletRequest;  
	  HttpServletResponse response = (HttpServletResponse) servletResponse;  
	
	  if (this.observeOncePerRequest && isApplied(request)) {  
		 chain.doFilter(request, response);  
		 return;      }  
	
	  if (skipDispatch(request)) {  
		 chain.doFilter(request, response);  
		 return;      }  
	
	  String alreadyFilteredAttributeName = getAlreadyFilteredAttributeName();  
	  request.setAttribute(alreadyFilteredAttributeName, Boolean.TRUE);  
	  try {  
		 AuthorizationDecision decision = this.authorizationManager.check(this::getAuthentication, request);  
		 this.eventPublisher.publishAuthorizationEvent(this::getAuthentication, request, decision);  
		 if (decision != null && !decision.isGranted()) {  
			throw new AccessDeniedException("Access Denied");  
		 }  
		 chain.doFilter(request, response);  
	  }  
	  finally {  
		 request.removeAttribute(alreadyFilteredAttributeName);  
	  }  
	}  

	//...
	
}
```
- `AuthorizationDecision` : authorizationManager에게 위임하고 나온 결과물
## 내부 구조
```java
http.authorizeHttpRequests(auth -> auth
	.requestMatchers("/user", "/myPage").hasAuthority("USER")
	.requestMatchers("/admin").hasRole("ADMIN")
	.requestMatchers("/db").access(new WebExpressionAuthorizationManager("hasRole('ADMIN') or hasRole('MANAGER')"))
	.anyRequest().authenticated()					  
);
```
![](https://i.imgur.com/9vndhtF.png)
