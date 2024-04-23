## 흐름
![](https://i.imgur.com/og4CekG.png)
1. 인증을 받아야만 접근이 가능한 페이지에 대해 Client의 요청
2. FilterChain의 가장 마지막 필터인 `AuthorizationFilter`가 `AuthorizationManager`를 통해 인증 필요 여부 판단
3. `AuthorizationFilter`가 `AccessDeniedException`을 날림
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
4. `ExceptionTranslationFilter`가 위 예외를 받아 처리
	- [[anonymous]] 사용자거나 [[rememberMe]]를 통해서 들어온 요청일 경우 핸들러 호출 X -> 인증 재요청
	- 인가되지 않은 사용자일 경우, `AccessDeniedHandler`가 처리