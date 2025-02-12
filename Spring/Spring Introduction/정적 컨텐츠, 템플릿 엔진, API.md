## 1️⃣ 정적 컨텐츠
서버에 저장되어 있으며 모든 사용자에게 동일하게 전달되는 파일
![](https://i.imgur.com/OX8dOa4.png)
`resources/static`에 위치
### 원리
![](https://i.imgur.com/dBn0A0B.png)
**Controller**가 없을 경우, `resources/static`에서 요청된 html파일을 찾아 그대로 반환
## 2️⃣ 템플릿 엔진
MVC를 이용하여 값을 주입할 수 있음
![](https://i.imgur.com/akzDhPz.png)
`resources/templates`에 위치
### 컨트롤러
```java
@GetMapping("hello-mvc")  
public String helloMvc(@RequestParam("name") String name, Model model) {  
    model.addAttribute("name", name);  
    return "hello-template";  
}
```
### 원리
![](https://i.imgur.com/KqlK4zo.png)
요청이 들어왔을 때 `DispatherServlet`에 의해 `ViewResolver`가 호출됨
`DispatcherServlet`은 컨트롤러에서 반환된 `ModelAndView` 객체를 지니고 있으며 이 Model에 key, value 쌍으로 값을 저장함
> 위 컨트롤러에서 `name`이라는 key에 입력받은 파라미터 name을 value로 지님

`viewResolver`가 템플릿 파일에서 key값에 해당하는 부분 (`${name}`)에 모델이 저장하고 있는 value로 치환하여 반환
## 3️⃣ API
`@ResponseBody`를 이용하여 객체를 반환함
### 컨트롤러
```java
@GetMapping("hello-string")  
@ResponseBody  
public String helloString(@RequestParam("name") String name) {  
    return "hello " + name;  
}
```
`hello + ${name}`의 String을 그대로 웹페이지에 전달함
```java
@GetMapping("hello-api")  
@ResponseBody  
public Hello helloApi(@RequestParam("name") String name) {  
    Hello hello = new Hello();  
    hello.setName(name);  
  
    return hello;  
}  
  
static class Hello {  
    private String name;  
  
    public String getName() {  
        return name;  
    }  
  
    public void setName(String name) {  
        this.name = name;  
    }  
}
```
반환값이 객체일 경우 json 형식으로 전달됨
### 원리
![](https://i.imgur.com/LvrbrmL.png)
`ViewResolver` 대신 `HttpMessageConverter`([[HTTP 메시지 컨버터]])가 해당 객체를 형태를 바꾸어 전달함

- 기본 문자 처리 : `StringHttpMessageConverter`
- 기본 객체 처리 : `MappingJackson2HttpMessageConverter`

> XML형식으로 변환도 가능하며, json 기본 형태 변환기인 jackson을 gson으로 변경할 수도 있음
