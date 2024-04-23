## 개요
- filter chain 내에서 발생하는 예외
- [[ExceptionTranslationFilter]]가 예외 처리, 사용자의 인증 및 인가 상태에 따라 로그인 재시도, 401, 403 코드 등으로 응답
## 유형
### 1️⃣ AuthenticationException
- 인증 예외
- [[SecurityContext]]에서 [[Authentication]] 삭제
- `AuthenticationEntryPoint`를 실행하고 인증 실패를 공통적으로 처리 & 인증 시도 화면으로 이동
- [[RequestCache]]에 요청 정보를 저장하여 인증 완료 후 재사용 (기본 구현 : `HttpSessionRequestCache`)

```java
package org.springframework.security.web.access;  
  
public class ExceptionTranslationFilter extends GenericFilterBean implements MessageSourceAware {  

	//...
  
	protected void sendStartAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,  
		 AuthenticationException reason) throws ServletException, IOException {  
	  // SEC-112: Clear the SecurityContextHolder's Authentication, as the  
	  // existing Authentication is no longer considered valid      SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();  
	  this.securityContextHolderStrategy.setContext(context);  
	  this.requestCache.saveRequest(request, response);  
	  this.authenticationEntryPoint.commence(request, response, reason);  
	}

	//...
  
}
```
### 2️⃣ AccessDeniedException
- 인가 예외
- [[anonymous]] 사용자 여부를 판단
	- 익명 사용자일 경우 인증 예외 처리 -> 인가를 할 필요가 없으니 인증을 다시 받도록 함
	- 익명 사용자가 아닐 경우 `AccessDeniedHandler`에게 위임
## API
```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
	http.exceptionHandling(exception -> exception
		.authenticationEntryPoint((request, response, authException) -> {
			System.out.println(authException.getMessage());
		})
		.accessDeniedHandler((request, response, accessDeniedException) -> {
			System.out.println(accessDeniedException.getMessage());
		})
	);

	return http.build();
}
```
### 기본 구현체
#### AuthenticaitonEntryPoint
- **UsernamePasswordAuthenticationFilter** : `LoginUrlAuthenticationEntryPoint`
- **BasicAuthenticationFilter** : `BasicAuthenticationEntryPoint`
- 아무 인증 프로세스 설정 X :  `Http403ForbiddenEntryPoint`
- 사용자 정의 AuthenticationEntryPoint 구현체가 우선 수행되며 기본 로그인 페이지 생성 무시
	> 💡 기본 entry point에서는 마지막으로 로그인 페이지로 이동하기 때문
#### AccessDeniedHandler
- `AccessDeniedHandlerImpl`