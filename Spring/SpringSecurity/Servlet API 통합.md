## 개요
인증 관련 기능들을 필터가 아닌 서블릿 영역에서 처리
## Servlet 3+ 통합
### SecurityContextHolderAwareRequestFilter
- `HttpServletRequest`에 보안 관련 메소드를 추가적으로 제공하는 래퍼 클래스(`SecurityContextHolderAwareRequestWrapper`) 적용
- 서블릿 API의 보안 메소드를 사용하여 인증, 로그인, 로그아웃 등의 작업 수행
![](https://i.imgur.com/uQEqOyp.png)
### HttpServlet3RequestFactory
- `Servlet3SecurityContextHolderAwareRequestWrapper` 객체 생성
![](https://i.imgur.com/lMdNbvn.png)
### Servlet3SecurityContextHolderAwareRequestWrapper
- `HttpServletRequest`의 래퍼 클래스
- Servlet 3.0 기능을 지원하면서 SecurityContextHolder와의 통합 제공
- [[SecurityContext]]에 쉽게 접근, Servlet 3.0의 비동기 처리 기능
![](https://i.imgur.com/Q3SVHZL.png)
- `authenticate()` : 사용자가 인증되었는지 확인하고 인증되지 않았다면 로그인 페이지로 사용자를 보냄
- `login()` : [[AuthenticationManager]]를 사용하여 사용자가 인증할 수 있도록 함
- `logout()` : `LogoutHandlers`를 사용하여 [[Logout]] 수행
- `startAsync()` : 호출한 스레드에서 발견된 [[SecurityContext]]를 Runnable 스레드에 자동 복사
## 예시
```java
@GetMapping("/login")
public String login(HttpServletRequest request, MemberDto memberDto) throws ServletException, IOException {
	request.login(memberDto.getUsername(), memberDto.getPassword());
	System.out.println("PreAuthorize");
	return "/index";
}

@GetMapping("/users")
public List<User> users(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
	boolean authenticate = request.authenticate(response);
	if(authenticate) {
		UserService.users();
	}

	return Collection.emptyList();
}
```
## 코드
```java
package org.springframework.security.web.servletapi;  
  
public class SecurityContextHolderAwareRequestFilter extends GenericFilterBean {  
  
	//...

	@Override  
	public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)  
	      throws IOException, ServletException {  
	   chain.doFilter(this.requestFactory.create((HttpServletRequest) req, (HttpServletResponse) res), res);  
	}
	
	//...

}
```
- `requestFactory.create(...)` : `Servlet3SecurityContextHolderAwareRequestWrapper`를 생성
	```java
	package org.springframework.security.web.servletapi;  
	  
	final class HttpServlet3RequestFactory implements HttpServletRequestFactory {  
	
		//...
	  
		@Override  
		public HttpServletRequest create(HttpServletRequest request, HttpServletResponse response) {  
		  Servlet3SecurityContextHolderAwareRequestWrapper wrapper = new Servlet3SecurityContextHolderAwareRequestWrapper(  
				request, this.rolePrefix, response);  
		  wrapper.setSecurityContextHolderStrategy(this.securityContextHolderStrategy);  
		  return wrapper;  
		}  
	  
		//...
		
	}
	```
```java
package io.security.springsecuritymaster;  
  
import jakarta.servlet.http.HttpServletRequest;  
import org.springframework.web.bind.annotation.GetMapping;  
import org.springframework.web.bind.annotation.RestController;  
  
@RestController  
public class IndexController {  
  
    @GetMapping("/user")  
    public String user(HttpServletRequest request){  
        return "user";  
    }  
  
    //...
}
```
- `Servlet3SecurityContextHolderAwareRequestWrapper`가 `HttpServletRequest`로서 전달
