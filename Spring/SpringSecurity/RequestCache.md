# RequestCache
- 인증 절차 문제로 리다이렉트 된 후 이전에 했던 요청 정보를 담고 있는 `SavedRequest` 객체를 쿠키, 세션에 저장하고 필요시 다시 가져와 실행
![](https://i.imgur.com/NeLAuOT.png)
# SavedRequest
- 인증 절차 후 사용자를 원래 페이지로 안내
- 이전 요청과 관련된 여러 정보 저장
![](https://i.imgur.com/621FQ9b.png)
# 흐름
![](https://i.imgur.com/eE6vyDF.png)
1. Client의 요청
2. `HttpSessionRequestCache`의 `saveRequest()` 호출
3. 인증 이전 접근하려고 했던 페이지에 대한 정보를 `DefaultSavedRequest`에 담아 `HttpSession`에 저장
4. 저장 후 인증 페이지로 리다이렉트
5. 인증 후, 성공 시 `AuthenticationSuccessHandler`가 `HttpSessionRequestCache` 호출
6. `HttpSession`에서 `DefaultSavedRequest`를 호출해 이전 접근하려고 했던 페이지를 추출
7. 해당 페이지로 리다이렉트
## requestCache() API
```java
HttpSessionRequestCache requestCache = new HttpSessionRequestCache();
requestCache.setMatchingRequestParameterName("customparam=y");
http
	.requestCache((cache) -> cache
		.requestCache(requestCache)
	);
```
- 요청 Url에 customParam=y라는 매개 변수가 있는 경우에만 HttpSession에 저장된 `SavedRequest` 객체를 꺼내옴 (기본값 : continue)
```java
RequestCache nullRequestCache = new NullRequestCache();
http
	.reqeuestCache((cache) -> cache
		.requestCache(nullRequestCache)
	);
```
- 요청을 저장하지 않도록 하려면 `NullRequestCache` 구현
## RequestCacheAwareFilter
### 개념
- `SavedRequest`를 다시 불러오는 역할
- `SavedRequest`가 현재 request와 일치하면 이 요청을 필터 체인에 전달하고, `SavedRequest`가 null이라면 원래 request를 그대로 진행
### 흐름
![](https://i.imgur.com/QX4UuWc.png)
1. Client의 요청
2. `RequestCacheAwareFilter`가 세션에 저장된 `SavedRequest` 추출
3. `SavedRequest`가 null이라면 다음 필터 진행 (`chain.doFilter()`)
4. null이 아니라면 현재 요청과 일치하는지 확인
	- 일치하면, 필터 체인에 `SavedRequest` 전달
	- 일치하지 않는다면, 필터 체인에 현재 요청 그대로 진행
### 코드
**인증 전**
```java
package org.springframework.security.web.savedrequest;  
  
public class HttpSessionRequestCache implements RequestCache {  
  
	//...
	
	private String matchingRequestParameterName = "continue";  
	
	public void saveRequest(HttpServletRequest request, HttpServletResponse response) {  
	  if (!this.requestMatcher.matches(request)) {  
		 if (this.logger.isTraceEnabled()) {  
			this.logger  
			   .trace(LogMessage.format("Did not save request since it did not match [%s]", this.requestMatcher));  
		 }  
		 return;  
	  }  
	
	  if (this.createSessionAllowed || request.getSession(false) != null) {  
		 // Store the HTTP request itself. Used by  
		 // AbstractAuthenticationProcessingFilter         // for redirection after successful authentication (SEC-29)         
		DefaultSavedRequest savedRequest = new DefaultSavedRequest(request, this.portResolver,  
			   this.matchingRequestParameterName);  
		 request.getSession().setAttribute(this.sessionAttrName, savedRequest);  
		 if (this.logger.isDebugEnabled()) {  
			this.logger.debug(LogMessage.format("Saved request %s to session", savedRequest.getRedirectUrl()));  
		 }  
	  }  
	  else {  
		 this.logger.trace("Did not save request since there's no session and createSessionAllowed is false");  
	  }  
	}  

	//...
}
```
- `requestMather.matches(request)` : 현재 요청과 일치하는지 확인
- `new DefaultSavedRequest(~)` : parameter name (continue)와 함께 `SavedRequest` 객체 생성
- `getSession().setAttribute(, savedRequest)` : `SavedRequest` 객체를 세션에 저장
**인증 후**
```java
package org.springframework.security.web.authentication;  
  
 public class SavedRequestAwareAuthenticationSuccessHandler extends SimpleUrlAuthenticationSuccessHandler {  
	//...

	private RequestCache requestCache = new HttpSessionRequestCache();  
	
	@Override  
	public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response,  
		 Authentication authentication) throws ServletException, IOException {  
	  SavedRequest savedRequest = this.requestCache.getRequest(request, response);  
	  if (savedRequest == null) {  
		 super.onAuthenticationSuccess(request, response, authentication);  
		 return;      }  
	  String targetUrlParameter = getTargetUrlParameter();  
	  if (isAlwaysUseDefaultTargetUrl()  
			|| (targetUrlParameter != null && StringUtils.hasText(request.getParameter(targetUrlParameter)))) {  
		 this.requestCache.removeRequest(request, response);  
		 super.onAuthenticationSuccess(request, response, authentication);  
		 return;      }  
	  clearAuthenticationAttributes(request);  
	  // Use the DefaultSavedRequest URL  
	  String targetUrl = savedRequest.getRedirectUrl();  
	  getRedirectStrategy().sendRedirect(request, response, targetUrl);  
	}  
	
	//...
  
}
```
- `requestCache.getRequest()` : `SavedRequest` 객체를 `RequestCache`로부터 추출
	```java
	package org.springframework.security.web.savedrequest;  
  
	public class HttpSessionRequestCache implements RequestCache {  
	  
		//...

		@Override  
		public SavedRequest getRequest(HttpServletRequest currentRequest, HttpServletResponse response) {  
		   HttpSession session = currentRequest.getSession(false);  
		   return (session != null) ? (SavedRequest) session.getAttribute(this.sessionAttrName) : null;  
		}

		//...
	}
	```
	- `session.getAttribute()` : 세션으로부터 `SavedRequest` 객체 추출
- `savedRequest.getRedirectUrl()` : 리다이렉트할 URL 추출
	![](https://i.imgur.com/Kmzb1Sp.png)
	- 기본 request parameter name인 continue가 붙음
		```java

		package org.springframework.security.web.savedrequest;

		public class RequestCacheAwareFilter extends GenericFilterBean {  
  
		  //...
		  
			@Override  
			public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)  
				 throws IOException, ServletException {  
			  HttpServletRequest wrappedSavedRequest = this.requestCache.getMatchingRequest((HttpServletRequest) request,  
					(HttpServletResponse) response);  
			  chain.doFilter((wrappedSavedRequest != null) ? wrappedSavedRequest : request, response);  
			}  

		}
		```
		- `requestCache.getMatchingRequest()` : 요청에 대해 검사 작업
			```java
			package org.springframework.security.web.savedrequest;  
  
			public class HttpSessionRequestCache implements RequestCache {  
			  
				//...
		
				@Override  
				public HttpServletRequest getMatchingRequest(HttpServletRequest request, HttpServletResponse response) {  
				   if (this.matchingRequestParameterName != null) {  
				      if (!StringUtils.hasText(request.getQueryString())  
				            || !UriComponentsBuilder.fromUriString(UrlUtils.buildRequestUrl(request))  
				               .build()  
				               .getQueryParams()  
				               .containsKey(this.matchingRequestParameterName)) {  
				         this.logger.trace(  
				               "matchingRequestParameterName is required for getMatchingRequest to lookup a value, but not provided");  
				         return null;      }  
				   }  
				   SavedRequest saved = getRequest(request, response);  
				   if (saved == null) {  
				      this.logger.trace("No saved request");  
				      return null;   }  
				   if (!matchesSavedRequest(request, saved)) {  
				      if (this.logger.isTraceEnabled()) {  
				         this.logger.trace(LogMessage.format("Did not match request %s to the saved one %s",  
				               UrlUtils.buildRequestUrl(request), saved));  
				      }  
				      return null;  
				   }  
				   removeRequest(request, response);  
				   if (this.logger.isDebugEnabled()) {  
				      this.logger.debug(LogMessage.format("Loaded matching saved request %s", saved.getRedirectUrl()));  
				   }  
				   return new SavedRequestAwareWrapper(saved, request);  
				}
		
				//...
			}
			```
			- `matchingRequestParamterName != null` && `containsKey(parameterName)` : parameter name이 null이 아니고 (기본값 : continue), parameter name을 url이 포함하는 경우 `SavedRequest`를 담아 반환하고 아닐 경우 null 반환
		- 반환값이 null이라면 `SavedRequest`로 null을 반환하고 아닐 경우 반환값 반환
- `sendRedirect(..., targetUrl)` : targetUrl로 리다이렉트