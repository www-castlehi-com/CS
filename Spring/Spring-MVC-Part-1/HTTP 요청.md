# 헤더 조회
```java
package hello.springmvc.basic.request;  
  
@Slf4j  
@RestController  
public class RequestHeaderController {  
  
    @RequestMapping("/headers")  
    public String headers(HttpServletRequest request, HttpServletResponse response, HttpMethod httpMethod,  
       Locale locale, @RequestHeader MultiValueMap<String, String> headerMap, @RequestHeader("host") String host,  
       @CookieValue(value = "myCookie", required = false) String cookie) {  
       log.info("request={}", request);  
       log.info("response={}", response);  
       log.info("httpMethod={}", httpMethod);  
       log.info("locale={}", locale);  
       log.info("headerMap={}", headerMap);  
       log.info("header host={}", host);  
       log.info("myCookie={}", cookie);  
  
       return "ok";  
    }  
}
```
- `MultiValueMap`
	- 하나의 키에 여러 값을 받을 수 있는 Map
	- **keyA=value1&keyA=value2**
# 데이터 조회
## 1️⃣ GET - 쿼리 파라미터
- `/url**?username=hello&age=20`
- 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
> 검색, 필터, 페이징 등에서 사용

### Servlet
```java
package hello.springmvc.basic.request;  
  
@Slf4j  
@Controller  
public class RequestParamController {  
  
    @RequestMapping("/request-param-v1")  
    public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {  
       String username = request.getParameter("username");  
       int age = Integer.parseInt(request.getParameter("age"));  
  
       log.info("username={}, age={}", username, age);  
  
       response.getWriter().write("ok");  
    }  
}
```
### `@RequestParam`
```java
package hello.springmvc.basic.request;  
  
@Slf4j  
@Controller  
public class RequestParamController {  
  
    //...
  
    @ResponseBody  
    @RequestMapping("/request-param-v2")  
    public String requestParamV2(@RequestParam("username") String memberName, @RequestParam("age") int memberAge) {  
       log.info("username={}, age={}", memberName, memberAge);  
  
       return "ok";  
    }  
  
    @ResponseBody  
    @RequestMapping("/request-param-v3")  
    public String requestParamV3(@RequestParam String username, @RequestParam int age) {  
       log.info("username={}, age={}", username, age);  
  
       return "ok";  
    }  
  
    @ResponseBody  
    @RequestMapping("/request-param-v4")  
    public String requestParamV4(String username, int age) {  
       log.info("username={}, age={}", username, age);  
  
       return "ok";  
    }  
}
```
- `RequestBody` : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력
- `RequestParam` : 파라미터 이름으로 바인딩
	- HTTP 파라미터 이름이 변수 이름과 같으면 `name(value)` 속성 생략 가능
	- `String`, `int`, `Integer` 등의 단순 타입일 경우 `@RequestParam` 생략 가능
	- 스프링부트 3.2 이후부터 어노테이션에 적는 이름 생략 불가능하며, 생략할 경우 자바 컴파일럴에 `-parameters` 옵션 필요
```java
package hello.springmvc.basic.request;  
  
@Slf4j  
@Controller  
public class RequestParamController {  
  
    //...
  
    /**  
     * @RequestParam.required  
     * /request-param-required -> username이 없으므로 예외  
     *  
     * 주의!  
     * /request-param-required?username= -> 빈문자로 통과  
     *  
     * 주의!  
     * /request-param-required     * int age -> null을 int에 입력하는 것은 불가능, 따라서 Integer 변경해야 함(또는 다음에 나오는 defaultValue 사용)  
     */    @ResponseBody  
    @RequestMapping("/request-param-required")  
    public String requestParamRequired(@RequestParam(required = true) String username,  
       @RequestParam(required = false) Integer age) {  
       log.info("username={}, age={}", username, age);  
  
       return "ok";  
    }
}
```
- `@RequestParam.required` : 파라미터 필수 여부
	- 기본값 `true`
- 필수값으로 지정된 파라미터가 없을 경우 400 예외 발생
- null이 아닌 blank일 경우 통과
```java
package hello.springmvc.basic.request;  
  
@Slf4j  
@Controller  
public class RequestParamController {  
  
    //... 
  
    @ResponseBody  
    @RequestMapping("/request-param-default")  
    public String requestParamDefault(@RequestParam(required = true, defaultValue = "guest") String username,  
       @RequestParam(required = false, defaultValue = "-1") int age) {  
  
       log.info("username={}, age={}", username, age);  
  
       return "ok";  
    }  
}
```
- `defaultValue`를 사용해 기본 값 적용
- 기본 값이 있기 때문에 `required` 의미 X
- null이 아닌 blank일 경우에도 기본 값이 적용
```java
package hello.springmvc.basic.request;  
  
@Slf4j  
@Controller  
public class RequestParamController {  
  
    //...
  
    /**  
     * @RequestParam Map, MultiValueMap  
     * Map(key=value)     * MultiValueMap(key=[value1, value2, ...]) ex) (key=userIds, value=[id1, id2])     */    @ResponseBody  
    @RequestMapping("/request-param-map")  
    public String requestParamMap(@RequestParam Map<String, Object> paramMap) {  
       log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));  
  
       return "ok";  
    }  
}
```
- 파라미터의 값이 1개가 확실하다면 `Map`, 그렇지 않다면 `MultiValueMap` 사용
### @ModelAttribute
```java
package hello.springmvc.basic;  
  
@Data  
public class HelloData {  
  
    private String username;  
    private int age;  
}
```
- `@Data`
	- `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode`, `@RequiredArgsConstructor`를 자동으로 적용
```java
package hello.springmvc.basic.request;  
  
@Slf4j  
@Controller  
public class RequestParamController {  
  
    @ResponseBody  
    @RequestMapping("/model-attribute-v1")  
    public String modelAttributeV1(@ModelAttribute HelloData helloData) {  
       log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());  
         
       return "ok";  
    }  
}
```
1. `HelloData` 객체 생성
2. 요청 파라미터 이름으로 `HelloData` 객체의 프로퍼티를 찾고, setter를 호출해서 파라미터의 값을 바인딩

- 다른 파라미터 입력 시 `BindException` 발생
-  `@ModelAttribute` 생략 가능
	- `String`, `int`, `Integer`와 같은 단순 타입 = `@RequestParam`
	- 나머지 = `@ModelAttribute` (argument resolver로 지정해둔 타입 외)
## 2️⃣ POST - HTML Form
- `Content-Type: application/x-www-form-urlencoded`
- 메시지 바디에 쿼리 파라미터 형식으로 전달 `username=hello&age=20`
> 회원 가입, 상품 주문, HTML Form 사용
## 3️⃣ HTTP Message Body에 데이터를 직접 담아 요청
- HTTP API에서 주로 사용
- JSON, XML, TEXT 등이 있으며 데이터 형식은 주로 **JSON** 사용
- POST, PUT, PATCH
### Input, Output 스트림
```java
/**  
 * InputStream(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회  
 * OutputStream(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력  
 */  
@PostMapping("/request-body-string-v2")  
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {  
    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);  
  
    log.info("messageBody={}", messageBody);  
  
    responseWriter.write("ok");  
}
```
### HttpEntity
```java
/**  
 * HttpEntity: HTTP header, body 정보를 편리하게 조회  
 * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)  
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용  
 *  
 * 응답에서도 HttpEntity 사용 가능  
 * - 메시지 바디 정보 직접 반환(view 조회X)  
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용  
 */  
@PostMapping("/request-body-string-v3")  
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {  
    String messageBody = httpEntity.getBody();  
  
    log.info("messageBody={}", messageBody);  
  
    return new HttpEntity<>("ok");  
}

@ResponseBody  
@PostMapping("/request-body-json-v4")  
public HttpEntity<String> requestBodyJsonV4(HttpEntity<HelloData> data) {  
    HelloData helloData = data.getBody();  
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());  
  
    return new HttpEntity<>("ok");  
}
```
- `HttpEntity`를 상속받은 객체
	- `RequestEntity`
		- HttpMethod, url 정보 추가
		- 요청에서 사용
	- `ResponseEntity`
		- HTTP 상태 코드 설정 가능
		  `return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED)`
		- 응답에서 사용
### @RequestBody
```java
/**  
 * @RequestBody  
 * - 메시지 바디 정보를 직접 조회(@RequestParam X, @ModelAttribute X)  
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용  
 *  
 * @ResponseBody  
 * - 메시지 바디 정보 직접 반환(view 조회X)  
 * - HttpMessageConverter 사용 -> StringHttpMessageConverter 적용  
 */  
@ResponseBody  
@PostMapping("/request-body-string-v4")  
public String requestBodyStringV4(@RequestBody String messageBody) throws IOException {  
    log.info("messageBody={}", messageBody);  
  
    return "ok";  
}

/**  
 * @RequestBody 생략 불가능 (@ModelAttribute 가 적용되어 버림)  
 * HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter (content-type: application/json)  
 * * @ResponseBody 적용  
 * - 메시지 바디 정보 직접 반환 (view 조회 X)  
 * - HttpMessageConverter 사용 -> MappingJackson2HttpMessageConverter 적용 (Accept: application/json)  
 */@ResponseBody  
@PostMapping("/request-body-json-v5")  
public HelloData requestBodyJsonV5(@RequestBody HelloData helloData) {  
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());  
  
    return helloData;  
}
```
- 헤더 정보가 필요할 경우 `HttpEntity` 혹은 `@RequestHeader` 사용
- 요청 메시지가 json일 경우 content-type이 application/json인지 확인

- `@RequestBody` 요청 : JSON 요청 -> HTTP 메시지 컨버터 -> 객체
- `@ResponseBody` 응답 : 객체 -> HTTP 메시지 컨버터 -> JSON 응답
