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
`HandlerExceptionResolverComposite`에 다음 순서로 등록
### 1️⃣ `ResponseStatusExceptionResolver`
#### 개념
- HTTP 상태 코드 지정
  `@ResponseStatus(value = HttpStatus.NOT_FOUND)`
- `ResponseStatusException` 사용
	- 개발자가 직접 변경할 수 없는 예외에 적용
	- 조건에 따라 동적으로 변경시킬 경우에 적용
#### 처리
1. `@ResponseStatus`가 달려있는 예외
2. `ResponseStatusException` 예외
#### 원리
```java
package org.springframework.web.servlet.mvc.annotation;  
  
public class ResponseStatusExceptionResolver extends AbstractHandlerExceptionResolver implements MessageSourceAware {  
    //...
  
    @Nullable  
    protected ModelAndView doResolveException(HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) {  
        try {  
            if (ex instanceof ResponseStatusException rse) {  
			    return this.resolveResponseStatusException(rse, request, response, handler);  
			}
  
            ResponseStatus status = (ResponseStatus)AnnotatedElementUtils.findMergedAnnotation(ex.getClass(), ResponseStatus.class);  
            if (status != null) {  
                return this.resolveResponseStatus(status, request, response, handler, ex);  
            }  
  
            //... 
        }  
  
        return null;  
    }  
  
    //...
}
```
1. `ResponseStatus`  혹은 `ResponseStatusException` 찾기
```java
package org.springframework.web.servlet.mvc.annotation;  
  
public class ResponseStatusExceptionResolver extends AbstractHandlerExceptionResolver implements MessageSourceAware {  
    //...
  
    protected ModelAndView resolveResponseStatus(ResponseStatus responseStatus, HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) throws Exception {  
        //...
        return this.applyStatusAndReason(statusCode, reason, response);  
    }  
  
    //...
  
    protected ModelAndView applyStatusAndReason(int statusCode, @Nullable String reason, HttpServletResponse response) throws IOException {  
        if (!StringUtils.hasLength(reason)) {  
            //...
        } else {  
            //...
            response.sendError(statusCode, resolvedReason);  
        }  
  
        return new ModelAndView();  
    }  
}
```
2. `response.sendError()` 혹은 빈 `ModelAndView` 반환
### 2️⃣ `DefaultHandlerExceptionResolver`
#### 개념
- 스프링 내부에서 발생하는 스프링 예외 해결
	> 파라미터 바인딩 시점에 타입 오류(`TypeMismatchException`) 에러
	> - 스프링이 500 오류를 발생
	> - HTTP에서는 400오류를 사용해야 함
#### 원리
```java
package org.springframework.web.servlet.mvc.support;  

public class DefaultHandlerExceptionResolver extends AbstractHandlerExceptionResolver {  
    //...
  
    @Nullable  
    protected ModelAndView doResolveException(HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) {  
        try {  
            if (ex instanceof ErrorResponse errorResponse) {  
                //...
            }  
  
            //...
  
            if (ex instanceof TypeMismatchException theEx) {  
                return this.handleTypeMismatch(theEx, request, response, handler);  
            }  
  
            //...
        }  
  
        return null;  
    }  
  
    //...
  
    protected ModelAndView handleTypeMismatch(TypeMismatchException ex, HttpServletRequest request, HttpServletResponse response, @Nullable Object handler) throws IOException {  
        response.sendError(400);  
        return new ModelAndView();  
    }  
  
    //...
}
```
1. 스프링이 발생한 에러를 잡아 handle 메소드 호출
2. handle 메소드에서 `response.sendError()` 혹은 빈 `ModelAndView` 반환
### 3️⃣ `ExceptionHandlerExceptionResolver`
#### `@ExceptionHandler`
##### 예외 처리 방법
**범위**
- 해당 컨트롤러만 효력
**우선순위**
```java
@ExceptionHandler(부모예외.class)
public String 부모예외처리() (부모예외 e) {}

@ExceptionHandler(자식예외.class)
public String 자식예외처리() (부모예외 e) {}
```
- 자식예외의 우선권 > 부모예외의 우선권
**배치 처리**
```java
@ExceptionHandler({AException.class, BException.class})
public String ex(Exception e) {
	log.info("exception e", e);
}
```
##### 문제
- 정상 코드와 예외 처리 코드가 하나의 컨트롤러에 섞여 있음
##### `@ControllerAdvice`
##### RestControllerAdvice
```java
@ControllerAdvice  
@ResponseBody  
public @interface RestControllerAdvice {}
```
##### 예외 처리 방법
```java
@Slf4j  
@RestControllerAdvice  
public class ExControllerAdvice {  
  
    @ResponseStatus(HttpStatus.BAD_REQUEST)  
    @ExceptionHandler(IllegalArgumentException.class)  
    public ErrorResult illegalHandler(IllegalArgumentException e) {  
       log.error("[exceptionHandler] ex", e);  
       return new ErrorResult("BAD", e.getMessage());  
    }  
  
    @ExceptionHandler  
    public ResponseEntity<ErrorResult> userExHandler(UserException e) {  
       log.error("[exceptionHandler] ex", e);  
       ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());  
       return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);  
    }  
  
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)  
    @ExceptionHandler  
    public ErrorResult exHandler(Exception e) {  
       log.error("[exceptionHandler] ex", e);  
       return new ErrorResult("EX", "내부 오류");  
    }  
}
```
- 대상으로 지정한 여러 컨트롤러에 `@ExceptionHandler`, `@InitBindier` 기능을 부여
- 대상을 지정하지 않을 시 글로벌 적용