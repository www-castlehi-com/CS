# 개념
- MVC  컨트롤러(핸들러) 밖으로 예외가 던져진 경우 예외를 해결하고, 동작을 새로 정의할 수 있는 방법 제공
- `ExceptionResolver`
# 적용
**적용 전**
![](https://i.imgur.com/fdUsmLG.png)
**적용 후**
![](https://i.imgur.com/fInHKbQ.png)
- 예외를 해결해도 `postHandle()` 호출 X
# 인터페이스
```java
package org.springframework.web.servlet;  

public interface HandlerExceptionResolver {  
    @Nullable  
    ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex);  
}
```
- `handler` : 핸들러 정보
- `Exception ex` : 핸들러에서 발생한 예외
**반환값**
- 빈 ModelAndView : 뷰를 렌더링하지 않고 정상 흐름으로 서블릿 리턴
- ModelAndView 지정 : 뷰 렌더링
- null : 다음 `ExceptionResolver`를 찾아 실행하고, 처리할 수 없는 resolver가 없다면 예외 처리를 하지 못하고 서블릿 밖으로 예외를 던짐
# 활용
- 예외 상태 코드 변환
	- `response.sendError(xx)` 호출로 변경해서 상태 코드에 따른 오류 처리
	- WAS는 서블릿 오류 페이지를 찾아 내부 호출
- 뷰 템플릿 처리
	- `ModelAndView`에 값을 채워서 예외에 따른 새로운 오류 화면 뷰 렌더링
- API 응답 처리
	- `response.getWriter().println("xx")`과 같이 HTTP 응답 바디에 직접 데이터 주입
	- JSON으로 응답 시 API 응답 처리 가능
# 등록
- `WebMvcConfigurer` 통해 등록
- `extendHandlerExceptionResolvers(..)` 사용
- `configureHandlerExceptionResolvers(..)` 사용 시  스프링이 기본으로 등록하는 `ExceptionResolver`가 제거됨
# 스프링
## 종류
### 1️⃣ `ExceptionHandlerExceptionResolver`
### 2️⃣ `ResponseStatusExceptionResolver`
#### 개념
- HTTP 상태 코드 지정
  `@ResponseStatus(value = HttpStatus.NOT_FOUND)`
#### 처리
1. `@ResponseStatus`가 달려있는 예외
2. `ResponseStatusException` 예외
### 3️⃣ `DefaultHandlerExceptionResolver`
