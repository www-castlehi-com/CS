- Aspect Oriented Programming
- 공통 관심 사항 (cross-cutting concern) vs 핵심 관심 사항 (core concern)
- 원하는 곳에 공통 관심 사항 적용
## 적용
```java
@Aspect  
@Component  
public class TimeTraceAop {  
  
    @Around("execution(* hello.hellospring..*(..))")  
    public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {  
       long start = System.currentTimeMillis();  
  
       System.out.println("START: " + joinPoint.toString());  
       try {  
          return joinPoint.proceed();  
       } finally {  
          long finish = System.currentTimeMillis();  
          long timeMs = finish - start;  
          System.out.println("END:" + joinPoint.toString() + " " + timeMs + " ms");  
       }  
    }  
}
```
![](https://i.imgur.com/s1PN0oQ.png)
호출 이전에 프록시 객체를 호출하며, `joinPoint.proceed()`시, 실제 객체를 호출
적용 타겟은 `@Around("execution()")`에서 결정