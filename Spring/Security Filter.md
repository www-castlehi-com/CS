# Filter
- 웹 애플리케이션에서 클라언트의 요청 (`HttpServletRequest`)과 서버의 응답 (`HttpServletResponse`)을 가공, 검사
- 클라이언트의 요청이 서블릿에 도달하기 전이나 서블릿이 응답을 클라이언트에게 보내기 전에 특정 작업 수행
- **서블릿 컨테이너 (WAS)**에서 생성, 실행, 종료
## WAS 내 동작
![](https://i.imgur.com/j8MIyk0.png)
- FilterChain의 Filter 모두 실행
- Servlet으로 전달
## Filter 구조
![](https://i.imgur.com/9xhcVGX.png)
### init
- 필터 초기화 시 필요한 작업 수행
### doFilter
- `chain.doFilter(request, response)` 메소드 전후로 처리 가능
- 전처리 : 요청 처리 전에 수행할 작업으로 **ServletRequest** 수정
- 후처리 : 응답 처리 후에 수행할 작업으로 **ServletResponse** 수정
### destroy
- 필터가 제거될 때 필요한 작업 수행
## DelegatingFilterProxy
### 개념
- 스프링에서 사용되는 서블릿 필터
- 서블릿 컨테이너와 스프링 애플리케이션 컨텍스트 간의 연결고리
- 서블릿 필터 기능 + 스프링 의존성 주입 & 빈 관리
- `springSecurityFilterChain` 이름으로 생성된 빈을 ApplicationContext에서 찾아 요청 위임
- 실제 보안 처리 X
### 순서
![](https://i.imgur.com/atlYpsL.png)
1. Client가 요청을 함
2. `DelegatingFilterProxy`가 Spring IOC Container에서 `SpringSecurityFilterChain`의 이름을 가진 빈 찾음
3. 해당 빈은 Filter를 구현하고, `DelegatingFilterProxy`가 클라이언트의 요청을 해당 빈에 위임
### 코드
```java
package org.springframework.boot.autoconfigure.security.servlet;  
  
public class SecurityFilterAutoConfiguration {  
  
   private static final String DEFAULT_FILTER_NAME = AbstractSecurityWebApplicationInitializer.DEFAULT_FILTER_NAME;  

	//public static final String DEFAULT_FILTER_NAME = "springSecurityFilterChain";
  
   @Bean  
   @ConditionalOnBean(name = DEFAULT_FILTER_NAME)  
   public DelegatingFilterProxyRegistrationBean securityFilterChainRegistration(  
         SecurityProperties securityProperties) {  
      DelegatingFilterProxyRegistrationBean registration = new DelegatingFilterProxyRegistrationBean(  
            DEFAULT_FILTER_NAME);  
      registration.setOrder(securityProperties.getFilter().getOrder());  
      registration.setDispatcherTypes(getDispatcherTypes(securityProperties));  
      return registration;  
   }  

	//...
}
```
- `DEFAULT_FILTER_NAME` : springSecurityFilterChain의 이름을 가진 빈을 찾음
- `DelegatingFilterProxyRegistrationBean` : `DelegatingFilterProxy` 생성
## FilterChainProxy
- `springSecurityFilterChain`의 이름으로 생성되는 필터 빈
- `DelegatingFilterProxy`로 부터 요청 위임 & 보안 처리
- 내부적으로 하나 이상의 SecurityFilterChain 객체를 가짐
- 요청 URL을 기준으로 적절한 SecurityFilterChain을 선택해 필터 호출
	>💡0번 ~ 15번의 16개의 Filter를 가지고 있음
- `HttpSecurity`를 통해 API 추가 시 관련 필터 추가
- 사용자의 요청을 필터 순서대로 호출하여 보안 기능 동작
- 필요 시 직접 필터 생성

![](https://i.imgur.com/fb4kic0.png)
1. Client가 요청을 함
2. `DelegatingFilterProxy`가 ApplicationContext를 찾음
3. 클라이언트의 요청을 `FilterChainProxy`에게 위임
4. `FilterChainProxy`는 `SecurityFilterChain`의 필터 목록을 통해 요청 처리
5. Filter처리 종료 후, Spring MVC Servlet으로 전달
