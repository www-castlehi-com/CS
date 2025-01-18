# 구조
![](https://i.imgur.com/7gqkBmu.png)
## DispatcherServlet
- [[프론트 컨트롤러 패턴]]으로 구현
### 등록
- `HttpServlet` 상속
  ![](https://i.imgur.com/aeT6RiX.png)
- 스프링 부트가 서블릿을 자동으로 등록하며 **모든 경로**(`urlPatterns="/")에 대하여 매핑
### 요청 흐름
- 서블릿 호출 시, `HttpServlet.service()` 호출
- 최종적으로 `DispatcherServlet.doDispatch()` 호출
  ```java
	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
	
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		ModelAndView mv = null;
		
		// 1. 핸들러 조회  
		mappedHandler = getHandler(processedRequest); 
		if (mappedHandler == null) {
			 noHandlerFound(processedRequest, response);
			return; 
		}
		
		// 2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터  
		HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
		
		// 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환 
		mv ha.handle(processedRequest, response, mappedHandler.getHandler());
		processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
	}

	private void processDispatchResult(HttpServletRequest request, HttpServletResponse response, HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
		// 뷰 렌더링 호출  
		render(mv, request, response);
	 }

	protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception 
	{
		 View view;
		String viewName = mv.getViewName();  
		
		// 6. 뷰 리졸버를 통해서 뷰 찾기, 7. View 반환
		 view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
		 
		// 8. 뷰 렌더링
		 view.render(mv.getModelInternal(), request, response);
	 }
	```
	1. **핸들러 조회**
		- 요청 URL에 매핑된 핸들러(컨트롤러) 조회
	2. **핸들러 어댑터 조회**
		- 핸들러를 실행할 수 있는(`supports()`) 핸들러 어댑터 조회
	3. **핸들러 어댑터 실행**
	4. **핸들러 실행**
		- 핸들러 어댑터가 실제 핸들러 실행 (`handle()`)
	5. **ModelAndView 반환**
		- 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환
	6. **viewResolver** 호출
	7. **View 반환**
		- 뷰의 논리 이름을 물리 이름으로 바꾸고, 렌더링 역할을 담당하는 뷰 객체 반환
	8. **뷰 렌더링**
### 핸들러 매핑, 핸들러 어댑터
#### 핸들러
##### 과거 스프링 컨트롤러
```java
package org.springframework.web.servlet.mvc;  
  
@FunctionalInterface  
public interface Controller {  
    @Nullable  
    ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception;  
}
```
##### HttpRequestHandler
```java
@FunctionalInterface  
public interface HttpRequestHandler {  
    void handleRequest(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException;  
}
```
- 서블릿과 가장 유사한 형태의 핸들러
#### 실행
1. 핸들러 매핑으로 핸들러 조회
	- `HandlerMapping`을 순서대로 실행해서 핸들러 찾기
	```
	0 = RequestMappingHandlerMapping : 애노테이션 기반의 컨트롤러인 @RequestMapping 에서 사용
	1 = BeanNameUrlHandlerMapping : 스프링 빈의 이름으로 핸들러 매핑
	```
	- 과거 스프링은 빈 이름으로 핸들러를 찾아야하기 때문에 `BeanNameUrlHandlerMapping`이 실행에 성공하고 `Controller`를 구현한 컨트롤러 반환
	- 가장 우선순위가 높은 핸들러 매핑은 `RequestMappingHandlerMapping`
2. 핸들러 어댑터 조회
	- `HandlerAdapter`의 `supports()`를 순서대로 호출
	```
	0 = RequestMappingHandlerAdapter : 애노테이션 기반의 컨트롤러인 @RequestMapping에서 사용
	1 = HttpRequestHandlerAdapter : HttpRequestHandler 처리
	2 = SimpleControllerHandlerAdapter : Controller 인터페이스 처리
	```
	- 과거 스프링은 `SimpleControllerHandlerAdapter`가 대상이 됨
	- 가장 우선순위가 높은 핸들러 어댑터는 `RequestMappingHandlerAdapter`
3. 핸들러 어댑터 실행
	- `DispatcherServlet`이 조회한 핸들러 어댑터를 실행하면서 핸들러 정보도 함께 넘겨줌
	- 핸들러 어댑터는 넘겨 받은 핸들러(컨트롤러)를 내부에서 실행하고, 그 결과를 반환
### 뷰 리졸버
#### 종류
```
1 = BeanNameViewResolver : 빈 이름으로 뷰를 찾아서 반환
2 = InternalResourceViewResolver : JSP를 처리할 수 있는 뷰, forward()를 호출해 JSP 실행
.
.
```
#### 실행
1. 핸들러 어댑터 호출
	- 뷰 정보 획득
2. ViewResolver 호출
	- 뷰 이름으로 viewResolver를 순서대로 호출
3. `view.render()`

> 다른 뷰는 실제 뷰를 렌더링 하며, `forward()` 과정이 필요 없지만,
> JSP의 경우 `forward()`를 통해서 해당 JSP로 이동해야 렌더링이 됨

> Thymeleaf 뷰 템플릿을 사용하면 스프링 부트가 `ThymeleafViewResolver`를 등록
# @RequestMapping
- 핸들러 매핑 : `RequestMappingHandlerMapping`
- 핸들러 어댑터 : `RequestMappingHandlerAdapter`
```java
package hello.servlet.web.springmvc.v1;  
  
@Controller  
public class SpringMemberFormControllerV1 {  
  
    @RequestMapping("/springmvc/v1/members/new-form")  
    public ModelAndView process() {  
       return new ModelAndView("new-form");  
    }  
}
```
- `@Controller`
	-  스프링이 자동으로 스프링 빈 등록
	- 내부에 `@Component` 어노테이션이 있어서 컴포넌트 스캔의 대상이 됨
	- 어노테이션 기반 컨트롤러로 인식
	- `@Component` + `@RequsetMapping`
- `@RequestMapping`
	- 요청 정보 매핑
	- 해당 URL이 호출되면 메서드 호출
	- 어노테이션 기반으로 동작하기 때문에 메소드 이름과 관계 X
- `ModelAndView`
	- 모델과 뷰 정보를 담아 반환
