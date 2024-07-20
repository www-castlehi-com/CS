- 애플리케이션의 전체 동작 방식을 구성(config)
- **구현 객체 생성**, **연결**하는 책임을 가지는 별도의 설정 클래스
```java
public class AppConfig {  
  
    public MemberService memberService() {  
       return new MemberServiceImpl(new MemoryMemberRepository());  
    }  
  
    public OrderService orderService() {  
       return new OrderServiceImpl(new MemoryMemberRepository(), new FixDiscountPolicy());  
    }  
}
```
### 1️⃣ 구현 객체 생성
- `MemberServiceImpl`
- `MemoryMemberRepository`
- `OrderServiceImpl`
- `FixDiscountPolicy`
### 2️⃣ 생성자를 통해 주입(연결)
- `MemberServiceImpl`-> `MemoryMemberRepository`
- `OrderServiceImpl` -> `MemoryMemberRepository`,`FixDiscountPolicy`
