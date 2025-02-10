# 응답 데이터 생성
## 1️⃣ 정적 리소스
- 웹 브라우저에 정적인 HTML, css, js 제공
- 경로 : `/static`, `/public`, `/resources`, `/META-INF/resources`
### 2️⃣ 뷰 템플릿
- 뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어 전달
- `@ResponseBody`가 없으면 뷰 리졸버가 실행되어서 뷰를 찾고 렌더링
- 경로 : `src/main/resources/templates`
### 3️⃣ HTTP 메시지
- HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보냄

> HTML이나 뷰 템플릿을 사용해도 HTTP 응답 메시지 바디에 HTML 데이터가 담겨서 전달되지만, 정적 리소스나 뷰 템플릿을 거침
> 데이터를 실어 보낼 경우 직접 HTTP 응답 메시지를 전달하게 됨

- `@RestController`
	```java
	package org.springframework.web.bind.annotation;  
	  
	@Target({ElementType.TYPE})  
	@Retention(RetentionPolicy.RUNTIME)  
	@Documented  
	@Controller  
	@ResponseBody  
	public @interface RestController {  
	    @AliasFor(  
	        annotation = Controller.class  
	    )  
	    String value() default "";  
	}
	```
