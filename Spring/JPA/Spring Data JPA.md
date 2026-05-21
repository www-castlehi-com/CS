## 구현체
```java
@Repository  
@Transactional(  
    readOnly = true  
)  
public class SimpleJpaRepository<T, ID> implements JpaRepositoryImplementation<T, ID> {
	//...
}
```
- `@Repository` : JPA 예외를 스프링이 추상화한 예외로 변환
- `@Transactional` 적용
	- JPA의 모든 변경은 트랜잭션 안에서 동작
	- 스프링 데이터 JPA는 변경 메서드를 트랜잭션 내부에서 처리
	- service 계층에서 트랜잭션을 시작하지 않으면 repository에서 트랜잭션을 시작하고,
	  service 계층에서 트랜잭션을 시작하면 repository는 해당 트랜잭션을 전파 받아서 사용
- `readOnly = true`
	- 플러시를 생략해서 약간의 성능 향상을 얻을 수 있음