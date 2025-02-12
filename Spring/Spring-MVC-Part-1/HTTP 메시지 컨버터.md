# 인터페이스
```java
package org.springframework.http.converter;  
  
public interface HttpMessageConverter<T> {  
    boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);  
  
    boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);  
  
    List<MediaType> getSupportedMediaTypes();  
  
    default List<MediaType> getSupportedMediaTypes(Class<?> clazz) {  
        return !this.canRead(clazz, (MediaType)null) && !this.canWrite(clazz, (MediaType)null) ? Collections.emptyList() : this.getSupportedMediaTypes();  
    }  
  
    T read(Class<? extends T> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException;  
  
    void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException;  
}
```
HTTP 요청, 응답 둘 다 사용

- `canRead()`, `canWrite()` : 해당 클래스, 미디어 타입을 지원하는지 체크
- `read()`, `write()` : 메시지 컨버터를 통해 메시지를 읽고 쓰는 기능
## 구현체
- 대상 클래스 타입, 미디어 타입 등을 체크해 사용 여부를 결정함
- 만족하지 않을 시 다음 메시지 컨버터로 우선 순위가 넘어감
### 1️⃣ `ByteArrayHttpMessageConverter`
- 클래스 타입 : `byte[]`
- 미디어 타입 : `*/*`
- 응답 시 쓰기 미디어 타입이 `application/octet-stream`으로 지정
### 2️⃣ `StringHttpMessageConverter`
- 클래스 타입 : `String`
- 미디어 타입 : `*/*`
- 응답 시 쓰기 미디어 타입이 `text/plain`으로 지정
### 3️⃣ `MappingJackson2HttpMessageConverter`
- 클래스 타입 : 객체 또는 `HashMap`
- 미디어 타입 : `application/json`
- 응답 시 쓰기 미디어 타입이 `applicatin/json`으로 지정
# 흐름
## 요청 데이터 읽기
1. HTTP 요청, 컨트롤러에서 `@RequestBody`, `HttpEntity` 파라미터 사용
2. `canRead()` 호출
	- 대상 **클래스 타입**, **Content-Type** 지원 여부 확인
3. `canRead()`만족 시, `read()`를 호출해 객체를 생성하고 반환
## 응답 데이터 생성
1. 컨트롤러에서 `@ResponseBody`, `HttpEntity`로 값 반환
2. `canWrite()` 호출
	- 대상 **클래스 타입**, **Accept**(`@RequestMapping`의 `produces`) 지원 여부 확인
3. `canWrite()` 만족 시, `write()` 호출해 HTTP 응답 메시지 바디에 데이터 생성
# 사용
애노테이션 기반의 컨트롤러이며 `@RequestMapping`을 처리하는 핸들러 어댑터인 `RequestMappingHandlerAdapter`(요청 매핑 핸들러 어댑터)에서 사용됨
## RequestMappingHandlerAdapter 동작 방식
![](https://i.imgur.com/5J7MyZd.png)
### ArgumentResolver
- 컨트롤러 (핸들러)가 필요로 하는 다양한 **파라미터의 값(객체)을 생성**
- 파라미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨줌
```java
package org.springframework.web.method.support;  
  
public interface HandlerMethodArgumentResolver {  
    boolean supportsParameter(MethodParameter parameter);  
  
    @Nullable  
    Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer, NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;  
}
```
- `supportsParameter()`를 호출해서 해당 파라미터를 지원하는지 체크
- 지원할 시 `resolveArgument()` 호출해서 실제 객체 생성
- 생성된 객체는 컨트롤러 호출 시 반환됨
### ReturnValueHandler
- **응답 값**을 변환하고 처리
## HTTP 메시지 컨버터 위치
![](https://i.imgur.com/1hAMQrU.png)
### 요청
- `@RequestBody`를 처리하는 `ArgumentResolver`가 있고, `HttpEntity`를 처리하는 `ArgumentResolver`가 있음
- `ArgumentResolver`들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성
```java
package org.springframework.web.servlet.mvc.method.annotation;   
  
public abstract class AbstractMessageConverterMethodArgumentResolver implements HandlerMethodArgumentResolver {  
    //...
  
    @Nullable  
    protected <T> Object readWithMessageConverters(...) throws ... {  
        //...
  
        try {  
            //... 
            Iterator var13 = this.messageConverters.iterator();  
  
            while(var13.hasNext()) {  
                //...
                } else if (targetClass != null && converter.canRead(targetClass, contentType)) {  
                    //...
                }  
  
                if (converterTypeToUse != null) {  
                    if (message.hasBody()) {  
                        //...
                        switch (converterTypeToUse) {  
                            case BASE -> var34 = converter.read(targetClass, msgToUse);  
                            //...
                        }  
  
                        //...
                    } 
                    //...
                }  
            }  
  
            //...
        }  
  
        //...
    }  
  
    //... 
}
```
### 응답
- `@ResponseBody`와 `HttpEntity`를 처리하는 `ReturnValueHandler`가 있음
- `ReturnValueHandler`가 HTTP 메시지 컨버터를 호출해서 응답 결과를 만듦

>- `@RequestBody`, `@ResponseBody`가 있을 경우, `RequestResponseBodyMethodProcesesor`가 처리
>- `HttpEntity`가 있으면 `HttpEntityMethodProcessor`가 처리
>
>![](https://i.imgur.com/OHWRr7m.png)
> => `ArgumentResolver`, `ReturnValueHandler` 둘 다 가짐
# 확장
`HandlerMethodArgumentResolver`, `HandlerMethodReturnValueHandler`, `HttpMessageConverter`가 모두 인터페이스로 제공되므로 `WebMvcConfigurer`를 상속 받아 스프링 빈으로 등록해 확장 가능
```java
package org.springframework.web.servlet.config.annotation;  
  
public interface WebMvcConfigurer {  
    //...  
  
    default void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {  
    }  
  
    default void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) {  
    }  
  
    //... 
  
    default void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {  
    }  
  
    //... 
}
```