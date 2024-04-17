# SecurityContext란?
- **Authentication 저장** : 인증된 사용자의 [[Authentication]] 객체 저장
- **ThreadLocal 저장소 사용** : SecurityContextHolder를 통해 접근되며 ThreadLocal 저장소를 사용해 각 스레드가 자신만의 security context를 가짐
	![](https://i.imgur.com/Ml93fca.png)
	- 스레드마다 할당 되는 ThreadLocal 저장소에 SecurityContext를 저장
	- 동시성 문제 X
	- 스레드 풀에서 운용될 경우 기존의 ThreadLocal이 재사용될 수 있으므로 응답 직전에 SecurityContext 삭제
- **전역적** : 애플리케이션의 어느 곳에서나 접근 가능
# SecurityContextHolder란?
- **SecurityContext  저장**
- **전략 패턴** : 다양한 저장 전략을 지원하기 위해 SecurityContextHolderStrategy 인터페이스 사용
	- 기본 : `MODE_THREADLOCAL` -> ThreadLocal에 저장
	- 커스텀 : `SecurityContextHolder.setStrategyName(String)`
## 전략 패턴
- `MODE_THREADLOCAL`
	- 기본 모드
	- 각 스레드가 독립적인 security context를 가짐
- `MODE_INHERITABLETHREADLOCAL`
	- 부모 스레드로부터 자식 스레드로 security context 상속
	- 스레드 간 분산 실행할 경우 유용
- `MODE_GLOBAL`
	- 전역적으로 단일 security context
	- 서버 환경에서 부적합, 간단한 애플리케이션에 적합
## 구조
![](https://i.imgur.com/VAwUlJl.png)
- `getContext` : 현재 context를 얻음
	> 이전 : `SecurityContextHolder.getContext()`
	> 최신 : `SecurityContextHolder.getContextHolderStrategy.getContext()`
- `createEmptyContext` : 비어 있는 context 생성
- `clearContext` : 현재 context 삭제
	> 이전 : `SecurityContextHolder.clearContext()`
	> 최신 : `SecurityContextHolder.getContextHolderStrategy.clearContext()`
- `getDeferredContext` : 현재 context를 반환하는 Supplier를 얻음
- `setContext` : 현재 context 저장
- `setDeferredContext` : 현재 context를 반환하는 Supplier 저장

> 💡 `xxDeferredContext` : 런타임에 생성하여 지연 효과 -> 성능상의 이점
## 코드
### 모드
```java
package org.springframework.security.core.context;  

public class SecurityContextHolder {  
  
	public static final String MODE_THREADLOCAL = "MODE_THREADLOCAL";  
	
	public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";  
	
	public static final String MODE_GLOBAL = "MODE_GLOBAL";  

	//...
}
```
### 초기화
```java
package org.springframework.security.core.context;  

public class SecurityContextHolder {  
  
	//...

	private static void initializeStrategy() {  
	   if (MODE_PRE_INITIALIZED.equals(strategyName)) {  
	      Assert.state(strategy != null, "When using " + MODE_PRE_INITIALIZED  
	            + ", setContextHolderStrategy must be called with the fully constructed strategy");  
	      return;   }  
	   if (!StringUtils.hasText(strategyName)) {  
	      // Set default  
	      strategyName = MODE_THREADLOCAL;  
	   }  
	   if (strategyName.equals(MODE_THREADLOCAL)) {  
	      strategy = new ThreadLocalSecurityContextHolderStrategy();  
	      return;   }  
	   if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {  
	      strategy = new InheritableThreadLocalSecurityContextHolderStrategy();  
	      return;   }  
	   if (strategyName.equals(MODE_GLOBAL)) {  
	      strategy = new GlobalSecurityContextHolderStrategy();  
	      return;   }  
	   // Try to load a custom strategy  
	   try {  
	      Class<?> clazz = Class.forName(strategyName);  
	      Constructor<?> customStrategy = clazz.getConstructor();  
	      strategy = (SecurityContextHolderStrategy) customStrategy.newInstance();  
	   }  
	   catch (Exception ex) {  
	      ReflectionUtils.handleReflectionException(ex);  
	   }  
	}

	//...
	
}
```
- `MODE_THREADLOCAL` : `ThreadLocalSecurityContextHolderStrategy` 사용
- `MODE_INHERITABLETHREADLOCAL` : `InheritableThreadLocalSecurityContextHolderStrategy` 사용
- `MODE_GLOBAL` : `GlobalSecurityContextHolderStrategy` 사용