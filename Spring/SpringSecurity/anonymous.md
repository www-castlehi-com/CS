# 익명사용자란?
- '익명으로 인증된 사용자'
- 인증되지 않은 사용자 간에 실제적인 개념적 차이는 없으나 이를 객체로 관리
- `SecurityContextHolder`가 `Authentication` 객체를 포함하고, null을 포함하지 않기 위함
- 익명사용자는 세션에 저장하지 않음
- 익명사용자의 권한 별도로 운용 가능 -> 인증된 사용자가 접근할 수 없도록 구성 가능
## 스프링 MVC에서 익명 인증
- 스프링 MVC는 `HttpServletRequest.getPrincipal()`을 사용하여 파라미터 해결
- 요청이 익명일 경우 authentication이 null임
- 인증 사용자에서 로그아웃 후 접속할 경우 정상 작동하지 않음
```java
public String method(Authentication authentication) {
	if (authentication instanceof AnonymousAuthenticationToken) {
		return "anonymous";
	} else {
		return "not anonymous";
	}
}
```
- 익명 요청에서 authentication을 얻고 싶을 땐 `@CurrentSecurityContext` 사용
- `CurrentSecurityContextArgumentResolver`에서 요청을 가로채 처리
```java
public String method(@CurrentSecurityContext SecurityContext context) {
	return context.getAuthentication().getName();
}
```
### 코드
```java
	package org.springframework.security.web.method.annotation;  
	  
	public final class CurrentSecurityContextArgumentResolver implements HandlerMethodArgumentResolver {  
	
		//...
	  
		@Override  
		public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,  
			 NativeWebRequest webRequest, WebDataBinderFactory binderFactory) {  
		  SecurityContext securityContext = this.securityContextHolderStrategy.getContext();  
		  if (securityContext == null) {  
			 return null;  
		  }  
		  Object securityContextResult = securityContext;  
		  CurrentSecurityContext annotation = findMethodAnnotation(CurrentSecurityContext.class, parameter);  
		  String expressionToParse = annotation.expression();  
		  if (StringUtils.hasLength(expressionToParse)) {  
			 StandardEvaluationContext context = new StandardEvaluationContext();  
			 context.setRootObject(securityContext);  
			 context.setVariable("this", securityContext);  
			 context.setBeanResolver(this.beanResolver);  
			 Expression expression = this.parser.parseExpression(expressionToParse);  
			 securityContextResult = expression.getValue(context);  
		  }  
		  if (securityContextResult != null  
				&& !parameter.getParameterType().isAssignableFrom(securityContextResult.getClass())) {  
			 if (annotation.errorOnInvalidType()) {  
				throw new ClassCastException(  
					  securityContextResult + " is not assignable to " + parameter.getParameterType());  
			 }  
			 return null;  
		  }  
		  return securityContextResult;  
		}  
	
		//...
		
	}
```
## anoymous API
```java
http.anonymous(anoymous -> anonymous
	.principal("guest")
	.authorities("ROLE_GUEST")
);
```
## AnonymousAuthenticationFilter
### 개념
- SecurityContext의 authentication객체는 초기값이 null임
- `SecurityContextHolder`의 authentication객체가 null일 경우 필터가 이를 객체로 채움
### 구조
![](https://i.imgur.com/Cid2NWo.png)
1. Client의 요청
2. 요청 과정에서 `AnonymousAuthenticationFilter`에 올 때까지 인증을 받지 못했다면 실행
	- [[Authentication]]이 null이 아니라면 그 다음 필터 실행
3. Authentication이 null일 경우, (anonymousUser, ROLE_ANONYMOUS)를 가진 `AnonymousAuthenticationToken`을 생성
4. Authentication을 `SecurityContextHolder`([[SecurityContext]])에 저장