## 개요
- Cross Site Request Forgery
- 사이트 간 요청 위조
- 공격자가 사용자로 하여금 이미 인증된 다른 사이트에 대해 원치 않는 작업을 수행하게 만드는 기법
- 사용자의 브라우저가 자동으로 보낼 수 있는 인증 정보 (쿠키, 기본 인증 세션)를 이용하여 사용자가 의도하지 않은 요청을 서버로 전송
- 사용자가 로그인한 상태에서 악의적인 웹사이트를 방문하거나 이메일 등을 통해 악의적인 링크를 클릭할 때 발생
## 순서
![](https://i.imgur.com/W92Y2yr.png)
## 기능 활성화
```java
@Bean
SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
	http.csrf(Customizer.withDefaults());
	return http.build();
}
```
- `default` : 별도로 설정하지 않아도 활성화 상태로 초기화
- 토큰은 서버에 의해 생성되어 클라이언트의 세션에 저장되고 모든 변경 요청에서 서버는 이 토큰을 검증하여 요청의 유효성을 확인
	> 💡 토큰은 클라이언트마다 상이
- GET, HEAD, TRACE, OPTIONS와 같은 안전한 메서드는 무시하고 POST, PUT, DELETE와 같은 변경 요청 메서드만 CSRF 토큰 검사
- CSRF 토큰이 브라우저에 의해 자동으로 포함되지 않는 요청 부분에 위치해야하므로 HTTP 매개변수나 헤더에 요구
	> 💡 쿠키는 브라우저에서 자동으로 포함하므로 효과적이지 않음
## 기능 비활성화
### 전체 비활성화
```java
@Bean
SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
	http.csrf(csrf -> csrf.disabled())
	return http.build();
}
```
### 특정 엔드포인트만 비활성화
```java
@Bean
SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
	http.csrf(csrf -> csrf.ignoringRequestMatchers("/api/*"));
	return http.build();
}
```
## 토큰 유지
- `CsrfTokenRepository`를 사용하여 영속화
- `HttpSessionCsrfTokenRepository`와 `CookieCsrfTokenRepository`가 그 구현체
### 1️⃣ 세션에 토큰 저장 - HttpSessionCsrfTokenRepository
```java
@Bean
SecurityFilterChain defulatSecurityFilterChain(HttpSecurity http) throws Exception {
	HttpSessionCsrfTokenRepository repository = new HttpSessionCsrfTokenRepository();
	http.csrf(csrf -> csrf.csrfTokenRepository(repository));

	return http.build();
}
```
- HTTP 요청 헤더인 `X-CSRF-TOKEN` 또는 매개변수인 `_csrf`에서 토큰을 읽음
### 2️⃣ 쿠키에 토큰 저장 - CookieCsrfTokenRepository
```java
@Bean
SecurityFilterChain defulatSecurityFilterChain(HttpSecurity http) throws Exception {
	CookieCsrfTokenRepository repository = new CookieCsrfTokenRepository();
	http.csrf(csrf -> csrf.csrfTokenRepository(repository));
	http.csrf(csrf -> csrf.csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()));

	return http.build();
}
```
- `csrfTokenRepository(repository)` : 쿠키에 저장해 response로 담아 브라우저에 저장하며 http 요청에서만 볼 수 있음
- `csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())` : http뿐만 아니라 스크립트에서도 볼 수 있음
	> ⚠️ JavaScript에서 직접 쿠키를 읽을 경우에만 선택하는 것이 좋음

- `XSRF-TOKEN` 명을 가진 쿠키에 작성하고 HTTP 요청 헤더인 `X-XSRF-TOKEN` 또는 요청 매개변수인 `_csrf`에서 토큰을 읽음
## 토큰 처리
- `CsrfTokenRequestHandler`를 사용하여 토큰 생성 및 응답, HTTP 헤더 또는 요청 매개변수로부터 토큰의 유효성 검증
- `XorCsrfTokenRequestAttributehandler`와 `CsrfTokenRequestAttributeHandler` 구현체, 커스텀 핸들러 생성 가능
```java
@Bean
SecurityFilterChain defulatSecurityFilterChain(HttpSecurity http) throws Exception {
	XorCsrfTokenRequestAttributeHandler csrfTokenHandler = new XorCsrfTokenRequestAttributeHandler();
	http.csrf(csrf -> csrf.csrfTokenRequestHandler(csrfTokenHandler));

	return http.build();
}
```
- `_csrf`, `CsrfToken.class.getName()`으로 요청 객체 (`HttpServlerRequest`) 속성에 CsrfToken을 저장 & 로드
- 요청 헤더 또는 요청 매개변수 중 하나로부터 토큰의 유효성 비교 및 검증
- 클라이언트의 매 요청마다 CSRF 토큰 값(UUID)에 난수를 인코딩하여 변경한 CsrfToken이 반환되도록 보장하며 세션에 저장된 원본 토큰 값은 그대로 유지
	> 💡 헤더 값 또는 요청 매개변수로 전달된 인코딩된 토큰은 원본 토큰을 얻기 위해 디코딩됨
## 토큰 지연 로딩
- CsrfToken을 필요할 때까지 로딩을 지연 -> HttpSession으로부터 매 요청마다 로드할 필요가 없음
```java
@Bean
SecurityFilterChain defulatSecurityFilterChain(HttpSecurity http) throws Exception {
	XorCsrfTokenRequestAttributeHandler handler = new XorCsrfTokenRequestAttributeHandler();
	handler.setCsrfRequestAttributeName(null);

	http.csrf(csrf -> csrf
		.csrfTokenRequestHandler(handler));
	return http.build();
}
```
- `setCsrfRequestAttributeName(null)` : 지연토큰을 사용하지 않고 모든 요청마다 토큰을 로드
## 코드
### GET
```java
package org.springframework.security.web.csrf;  
  
public final class CsrfFilter extends OncePerRequestFilter {  

	//...
	
	@Override  
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)  
		 throws ServletException, IOException {  
		DeferredCsrfToken deferredCsrfToken = this.tokenRepository.loadDeferredToken(request, response);  
		request.setAttribute(DeferredCsrfToken.class.getName(), deferredCsrfToken);  
		this.requestHandler.handle(request, response, deferredCsrfToken::get);  
		if (!this.requireCsrfProtectionMatcher.matches(request)) {  
		 if (this.logger.isTraceEnabled()) {  
			this.logger.trace("Did not protect against CSRF since request did not match "  
				  + this.requireCsrfProtectionMatcher);  
		 }  
		 filterChain.doFilter(request, response);  
		 return;      
		}  
		//...
	}  
	
	//...
  
}
```
- `requireCsrfProtectionMatcher.matches(request)` : 요청객체로부터 csrf 보호가 필요한지 확인
	```java
	private static final class DefaultRequiresCsrfMatcher implements RequestMatcher {  
  
	   private final HashSet<String> allowedMethods = new HashSet<>(Arrays.asList("GET", "HEAD", "TRACE", "OPTIONS"));  
	  
	   @Override  
	   public boolean matches(HttpServletRequest request) {  
	      return !this.allowedMethods.contains(request.getMethod());  
	   }  
	  
	   @Override  
	   public String toString() {  
	      return "CsrfNotRequired " + this.allowedMethods;  
	   }  
	  
	}
	```
	- GET, HEAD, TRACE, OPTIONS인 경우 csrf 보호가 필요하지 않음
### POST
```java
package org.springframework.security.web.csrf;  
  
public final class CsrfFilter extends OncePerRequestFilter {  

	//...
	
	@Override  
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)  
		 throws ServletException, IOException {  
		DeferredCsrfToken deferredCsrfToken = this.tokenRepository.loadDeferredToken(request, response);  
		request.setAttribute(DeferredCsrfToken.class.getName(), deferredCsrfToken);  
		this.requestHandler.handle(request, response, deferredCsrfToken::get);  
		if (!this.requireCsrfProtectionMatcher.matches(request)) {  
			//...
		}  
		CsrfToken csrfToken = deferredCsrfToken.get();  
		String actualToken = this.requestHandler.resolveCsrfTokenValue(request, csrfToken);  
		if (!equalsConstantTime(csrfToken.getToken(), actualToken)) {  
		   boolean missingToken = deferredCsrfToken.isGenerated();  
		   this.logger  
		      .debug(LogMessage.of(() -> "Invalid CSRF token found for " + UrlUtils.buildFullRequestUrl(request)));  
		   AccessDeniedException exception = (!missingToken) ? new InvalidCsrfTokenException(csrfToken, actualToken)  
		         : new MissingCsrfTokenException(actualToken);  
		   this.accessDeniedHandler.handle(request, response, exception);  
		   return;}  
		filterChain.doFilter(request, response);
	}  
	
	//...
  
}
```
- `deferredCsrfToken.get()` : 메소드를 호출하는 시점에 토큰 init
	```java
	package org.springframework.security.web.csrf;  
	  
	final class RepositoryDeferredCsrfToken implements DeferredCsrfToken {  
	
		//...
		
		private void init() {  
		  if (this.csrfToken != null) {  
			 return;  
		  }  
		
		  this.csrfToken = this.csrfTokenRepository.loadToken(this.request);  
		  this.missingToken = (this.csrfToken == null);  
		  if (this.missingToken) {  
			 this.csrfToken = this.csrfTokenRepository.generateToken(this.request);  
			 this.csrfTokenRepository.saveToken(this.csrfToken, this.request, this.response);  
		  }  
		}  
	  
	}
	```
	- `loadToken(request)` : 세션 혹은 쿠키로부터 CsrfToken을 가져옴
	- `generateToken(request)` : CsrfToken이 존재하지 않는다면 생성
	- `saveToken(...)` : Session에 생성한 CsrfToken 저장
		```java
		package org.springframework.security.web.csrf;  
		  
		public final class HttpSessionCsrfTokenRepository implements CsrfTokenRepository {  
		
			//...
		  
		   @Override  
		   public void saveToken(CsrfToken token, HttpServletRequest request, HttpServletResponse response) {  
		      if (token == null) {  
		         HttpSession session = request.getSession(false);  
		         if (session != null) {  
		            session.removeAttribute(this.sessionAttributeName);  
		         }  
		      }  
		      else {  
		         HttpSession session = request.getSession();  
		         session.setAttribute(this.sessionAttributeName, token);  
		      }  
		   }  
		
			//...
		  
		}
		```
- `resolveCsrfTokenValue(...)` : 요청으로부터 토큰 로드
	```java
	package org.springframework.security.web.csrf;  
	  
	public interface CsrfTokenRequestHandler extends CsrfTokenRequestResolver {  
	  
		//...
		
		@Override  
		default String resolveCsrfTokenValue(HttpServletRequest request, CsrfToken csrfToken) {  
		  Assert.notNull(request, "request cannot be null");  
		  Assert.notNull(csrfToken, "csrfToken cannot be null");  
		  String actualToken = request.getHeader(csrfToken.getHeaderName());  
		  if (actualToken == null) {  
			 actualToken = request.getParameter(csrfToken.getParameterName());  
		  }  
		  return actualToken;  
		}  
	
	}
	```
	- `getHeader(...)`, `getParameter(...)` : 헤더 또는 파라미터로부터 토큰 로드
- `equalsConstantTime(csrfToken, actualToken)` : 시스템에 저장된 CsrfToken과 요청으로 넘어온 CsrfToken 비교
	- 같지 않을 경우 `accessDeniedHandler`가 처리
