- Java Persistent API
- 인터페이스이며 구현체의 예로 Hibernate 등이 있음
- ORM 기술로 관계형을 객체와 매핑시켜줌
## JPQL
```java
@Override  
public List<Member> findAll() {  
    return em.createQuery("select m from Member m", Member.class).getResultList();  
}
```
테이블 대상이 아닌 객체 (`Member`)를 대상으로 쿼리를 날리는 것
## 스프링 데이터 JPA
JPA 사용을 편리하게 해주는 프레임워크
```java
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {  
}
```
- 스프링 데이터 JPA가 interface를 자동으로 빈으로 등록해줌
![](https://i.imgur.com/lUzHsOj.png)
- PK와 관련된 CRUD 메소드가 이미 존재