## 개념
- [[SecurityContextRepository]]를 사용하여 [[SecurityContext]]를 얻고 이를 SecurityContextHolder에 저장하는 필터
- `SecurityContextRepository.saveContext()`를 강제 호출하지 않고 사용자 명시 호출
	> 기존 `SecurityContextPersistenceFilter`의 경우 강제 호출하여 세션에 무조건 저장
-  인증 지속 여부를 각 인증 메커니즘이 독립적으로 선택하여 더 나은 유연성 제공
- HttpSession에 필요할 경우에만 저장 -> 성능 향상
## SecurityContext 생성, 저장, 삭제
### 1️⃣ 익명 사용자
1. SecurityContextRepository를 사용하여 새로운 SecurityContext 객체를 생성하여 SecurityContextHolder에 저장 후 다음 필터로 전달
	> [[anonymous]] 인증은 세션에 저장 X -> 항상 새로운 SecurityContext 생성
2. AnonymousAuthenticationFilter에서 AnonymousAuthenticationToken 객체를 SecurityContext에 저장
	> SecurityContexet의 Authentication 객체가 `null`이기 때문
### 2️⃣ 인증 요청
1.  SecurityContextRepository를 사용하여 새로운 SecurityContext 객체를 생성하여 SecurityContextHolder에 저장 후 다음 필터로 전달
	> 인증을 받지 못한 상태이기 때문
2. [[form Login]]의 필터인 UsernamePasswordAuthenticationFilter에서 인증 성공 후 SecurityContext에 UsernamePasswordAuthentication 객체를 저장
3. SecurityContextRepository를 사용하여 HttpSession에 context 저장
### 3️⃣ 인증 후 요청
1. SecurityContextRepository를 사용하여 HttpSession에서 context를 로드하여 SecurityContextHolder에서 저장 후 다음 필터로 전달
2. context 내에 Authentication 객체가 존재하면 계속 인증 유지

> 💡 SecurityContextHolderFilter는 언제든지 SecurityContextHolder에 context를 저장하는 역할을 함
### 4️⃣ 클라이언트 응답 시
- `SecurityContextHolder.clearContext()`로 context 삭제
> ⚠️ 스레드 풀의 스레드일 경우 반드시 필요
## 흐름
![](https://i.imgur.com/LFPPFjA.png)
1. Client의 요청
2. SecurityContextFilter가 **무조건** 작동하여 SecurityContextRepository를 이용해 HttpSession에 SecurityContext가 존재하는지 검사

**존재하지 않을 경우**

3. 존재하지 않을 경우, 새로운 SecurityContext 생성하여 SecurityContextHolder에 저장
4. AuthenticationFilter가 인증에 성공할 경우 SecurityContext에 Authentication 객체 저장
5. SecurityContextRepository가 세션에 SecurityContext 저장
	> 💡 form-login의 경우 해당 작업이 내부적으로 존재하지만 커스텀의 경우 직접 해주어야함

**존재할 경우**
3. SecurityContextHolderFilter가 Session으로부터 context를 가져와 SecurityContextHolder에 저장하여 인증 상태 유지
4. 사용자에게 다음 필터로 넘겨줄 때, SecurityContextHolder 내의 context 삭제
## vs SecurityContextPersistenceFilter
![](https://i.imgur.com/C6UHNGh.png)
### 1️⃣ SecurityContextHolderFilter
`SecurityContextRepository.loadContext()`를 이용해 SecurityContextHolder에 로드한 SecurityContext 저장
### 2️⃣ SecurityContextPersistenceFilter
`SecurityContextRepository.loadContext()`를 이용해 SecurityContextHolder에 로드한 SecurityContext 저장
`SecurityContextRepository.saveContext()`를 이용해 클라이언트의 요청 후, 세션에 강제적으로 SecurityContext 저장
## API
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
	http.securityContext(securityContext -> securityContext
		.requireExplicitSave(true));

	return http.build();
}
```
- `requireExplicitSave(boolean)` 
	- SecurityContext를 명시적으로 저장할 것인지 아닌지 여부 설정
	- 기본값 true
	- true일 경우 `SecurityContextHolderFilter`, false일 경우 `SecurityContextPersistanceFilter` 실행
## 코드
```java
package org.springframework.security.web.context;  
  
public class SecurityContextHolderFilter extends GenericFilterBean {  

	//...
  
	private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)  
		 throws ServletException, IOException {  
	  if (request.getAttribute(FILTER_APPLIED) != null) {  
		 chain.doFilter(request, response);  
		 return;      }  
	  request.setAttribute(FILTER_APPLIED, Boolean.TRUE);  
	  Supplier<SecurityContext> deferredContext = this.securityContextRepository.loadDeferredContext(request);  
	  try {  
		 this.securityContextHolderStrategy.setDeferredContext(deferredContext);  
		 chain.doFilter(request, response);  
	  }  
	  finally {  
		 this.securityContextHolderStrategy.clearContext();  
		 request.removeAttribute(FILTER_APPLIED);  
	  }  
	}  

	//...
}
```
- `securityContextRepository.loadDeferredContext(...)` :  SecurityContext를 로드
	```java
	package org.springframework.security.web.context;  
	  
	public class HttpSessionSecurityContextRepository implements SecurityContextRepository {  
	
		//...
	  
	   @Override  
	   public DeferredSecurityContext loadDeferredContext(HttpServletRequest request) {  
	      Supplier<SecurityContext> supplier = () -> readSecurityContextFromSession(request.getSession(false));  
	      return new SupplierDeferredSecurityContext(supplier, this.securityContextHolderStrategy);  
	   }  
	
		//...
		
}
	```
	- `readSecurityContextFromSession(...)` : Session, Request로부터 Security Context 가져옴
	- `Supplier` : 필요 시점에 가져옴 (지연)
		> `Authentication authentication = this.securityContextHolderStrategy.getContext().getAuthentication();` 시점에 가져옴
- `clearContext` : context를 SecurityContextHolder로부터 삭제