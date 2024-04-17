# 개념
- `DefaultLogoutPageGeneratingFilter`를 통해 로그아웃 페이지 제공, GET /logout으로 접근 가능
- 실행은 기본적으로 POST /logout으로 가능
	**제외**
	- CSRF 기능을 비활성화할 경우
	- RequestMatcher를 사용할 경우
- logout filter를 거치지 않고 MVC에서 커스텀하게 구현 가능
- 로그인 페이지를 커스텀하게 구현할 경우, 로그아웃 기능도 커스텀하게 구현해야 함
# API
```java
http.logout(httpSecurityLogoutConfigurer -> httpSecurityLogoutConfigurer
	.logoutUrl("/logoutProc")
	.logoutRequestMatcher(new AntPathRequestMatcher("/logoutProc", "POST"))
	.logoutSuccessUrl("/logoutSuccess")
	.logoutSuccessHandler((request, response, authentication) -> {
		response.sendRedirect("/logoutSuccess")
	})
	.deleteCookies("JSESSIONID", "CUSTOM_COOKIE")
	.invalidHttpSession(true)
	.clearAuthentication(true)
	.addLogoutHandler((request, response, authentication) -> {})
	.permitAll()
);
```
- `lougoutUrl()` : 로그아웃이 발생하는 URL (기본값 : /logout)
- `logoutrequestMatcher(~(url, method))`
	- 로그아웃이 발생하는 RequestMatcher 지정하며 logourUrl보다 우선
	- method를 지정하지 않을 시 어떤 HTTP 메서드로든 요청될 때 로그아웃 가능
- `logoutSuccessUrl()` : 로그아웃이 발생한 후 리다이렉션 될 URL (기본값 : /login?logout)
- `logoutSuccessHandler()` : 사용할 LogoutSuccessHandler 설정
	- `response.sendRedirect()` : 지정 시, logoutSuccessUrl()은 무시
- `deleteCookies()` : 로그아웃 성공 시 제거될 쿠키 이름 지정
- `invalidHttpSession()` : HttpSession을 무효화할 경우 true, 아닐 경우 false (기본값 : true)
- `clearAuthentication()` : SecurityContextLogoutHandler가 인증 객체를 삭제 해야 하는지 여부 (기본값 : true)
- `permitAll()` : logoutUrl, requestMatcher의 URL에 대한 모든 사용자 접근 허용
# LogoutFilter
### 흐름도
![](https://i.imgur.com/eQZv8lf.png)
1. Client의 요청
	- Get 방식으로 로그아웃 페이지를 요청하고, 실제 로그아웃 기능은 Post로 동작
2. `LogoutFilter`의 `RequestMatcher`가 요청 정보가 매칭 되는지 확인 (기본값 : logout)
	- logout이 아니라면 다음 필터로 패스 (`chain.doFilter`)
3. `logoutHandler`가 여러 핸들러(쿠키, 커스텀, Csrf, SecurityContext, SuccessEvent)를 가지고 실행
4. 성공한 이후, `logoutSuccessHandler`가 후처리 (리다이렉션 등)
### 코드
```java
package org.springframework.security.web.authentication.logout;  
  
public class LogoutFilter extends GenericFilterBean {  

	//...
  
	private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)  
		 throws IOException, ServletException {  
	  if (requiresLogout(request, response)) {  
		 Authentication auth = this.securityContextHolderStrategy.getContext().getAuthentication();  
		 if (this.logger.isDebugEnabled()) {  
			this.logger.debug(LogMessage.format("Logging out [%s]", auth));  
		 }  
		 this.handler.logout(request, response, auth);  
		 this.logoutSuccessHandler.onLogoutSuccess(request, response, auth);  
		 return;      }  
	  chain.doFilter(request, response);  
	}  

	protected boolean requiresLogout(HttpServletRequest request, HttpServletResponse response) {  
	   if (this.logoutRequestMatcher.matches(request)) {  
	      return true;  
	   }  
	   if (this.logger.isTraceEnabled()) {  
	      this.logger.trace(LogMessage.format("Did not match request to %s", this.logoutRequestMatcher));  
	   }  
	   return false;  
	}
	
}
```
- `requiresLogout` : 로그아웃 필터에 조건이 부합한지 확인
	- `logoutRequestMatcher.matehs()` :  요청 정보 url, method가 일치한지 확인
- `securityContextHolderStrategy.getContext().getAuthentication()` : [[Authentication]] 객체(`UsernamePasswordAuthenticaitonToken`) 추출 (인증을 받았기 때문에 인증 객체 존재)
- `handler.logout()` : 핸들러가 로그아웃 기능 실행
	![](https://i.imgur.com/nHWG48S.png)
	- SecurityConfig는 커스텀핸들러
	```java
	.addLogoutHandler(new LogoutHandler() {  
	    @Override  
	    public void logout(HttpServletRequest request, HttpServletResponse response,  
	                       Authentication authentication) {  
	        HttpSession session = request.getSession();  
	        session.invalidate();  
	        SecurityContextHolder.getContextHolderStrategy().getContext().setAuthentication(null);  
	        SecurityContextHolder.getContextHolderStrategy().clearContext();  
	    }  
	})
	```
	- 5개의 핸들러를 번갈아가며 실행
- `logoutSuccessHandler.onLogoutSuccess()` : 로그아웃 후 후처리