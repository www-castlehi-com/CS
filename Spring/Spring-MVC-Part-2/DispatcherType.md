- 필터에서 `dispatcherTypes` 옵션 제공
```java
package jakarta.servlet;  
  
public enum DispatcherType {  
    FORWARD,  
    INCLUDE,  
    REQUEST,  
    ASYNC,  
    ERROR;  
  
    private DispatcherType() {  
    }  
}
```
- `REQUEST` : 클라이언트 요청
- `ERROR` : 오류 요청
- `FORWARD` : 다른 서블릿이나 JSP 호출 시
- `INCLUDE` : 다른 서블릿이나 JSP의 결과를 포함할 시
- `ASYNC` : 서블릿 비동기 호출