# 소개
- 자바 엔터프라이즈 시장의 주력 기술
- 자바 진영의 [[ORM]] 기술 표준
# 동작
### 구조
![](https://i.imgur.com/qA3Xv0x.png)
- 애플리케이션과 JDBC 사이에서 동작
### 저장
![](https://i.imgur.com/vtF1Bv0.png)
### 조회
![](https://i.imgur.com/l9cVEar.png)
## 표준 명세
![](https://i.imgur.com/CN92r4h.png)
- JPA 2.1 표준 명세를 구현한 3가지 구현체를 모은 인터페이스
# 사용 이유
- SQL 중심적인 개발 -> 객체 중심 개발
- 생산성
	- 저장 : `jpa.persist(member)`
	- 조회 : `Member member = jpa.find(memberId)`
	- 수정 : `member.setName("변경할 이름")`
	- 삭제 : `jpa.remove(member)`
- 유지 보수
	- 필드 변경 시 모든 SQL 수정 -> 필드만 추가
- 패러다임 불일치 해결
	- 상속
		- 저장 : 개발자는 `jpa.persist(album)`만 하고, JPA가 상속 관계인 테이블에 모두 INSERT 쿼리
		- 조회 : 개발자는 `jpa.find(albumId)`만 하고, JPA가 상속 관계 테이블을 JOIN하여 처리
	- 연관관계 객체 그래프 탐색, 신뢰할 수 있는 엔티티 : `jpa.find(memberId)`만 하고 `member.getTeam()`으로 연관관계 객체 탐색
	- 비교 : `jpa.find(memberId)`로 찾은 두 객체의 엔티티가 같음이 보장됨
- 성능 최적화
	- 1차 캐시와 동일성 보장
		- 같은 트랜잭션 안에서는 같은 엔티티를 반환
		- DB Isolation Level이 Read Commit이어도 애플리케이션에서 Repeatable Read 보장
	- 트랜잭션을 지원하는 쓰기 지연
		- **INSERT** : 트랜잭션을 커밋할 때까지 SQL을 모으고, JDBC BATCH SQL 기능을 사용해서 한 번에 SQL 전송
		- **UPDATE** : 로우락 시간 최소화하고 트랜잭션 커밋 시 SQL을 실행하고 바로 커밋
	- 지연 로딩
		- 객체가 실제 사용될 때 로딩
- 데이터 접근 추상화와 벤더 독립성
- 표준 기술
# 설정
- `spring-boot-starter-data-jpa` 라이브러리 사용
	- JPA와 스프링 데이터 JPA를 스프링 부트와 통합
	- `spring-boot-starter-jdbc`도 함께 포함됨
```properties
#JPA log  
logging.level.org.hibernate.SQL=DEBUG  
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```
- `org.hiberante.SQL=DEBUG` : 하이버네이트가 생성하고 실행하는 SQL
- `org.hibernate.type.descriptor.sql.BasicBinder=TRACE`, `org.hibernate.orm.jdbc.bind=TRACE` : SQL에 바인딩 되는 파라미터를 확인하며, `logger`를 통해 SQL이 출력됨
	> `spring.jpa.show-sql=true` : `System.out` 콘솔을 통해 SQL이 출력되어 권장하지 않는 방법
# 사용
## ORM 매핑
```java
package hello.itemservice.domain;  
  
@Data  
@Entity  
public class Item {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    @Column(name = "item_name", length = 10)  
    private String itemName;  
    private Integer price;  
    private Integer quantity;  
  
    public Item() {  
	}  
	  
	public Item(String itemName, Integer price, Integer quantity) {  
	    this.itemName = itemName;  
	    this.price = price;  
	    this.quantity = quantity;  
	}
}
```
 - `@Entity` : JPA가 사용하는 객체
 - `@Id` : 테이블의 PK와 해당 필드를 매핑
 - `@GeneratedValue(strategy = GenerationType.IDENTITY)` : PK 생성 값을 데이터베이스에서 생성하는 IDENTITY 방식
 - `@Column` : 객체의 필드를 테이블의 컬럼과 매핑하며, 생략할 경우 필드의 이름을 사용
	 - `name = "item_name"` : 스프링 부트와 통합 시 객체 필드의 카멜 케이스를 테이블 컬럼의 언더 스코어로 자동 변환해주어 생략 가능
	 - `length = 10` : JPA 매핑 정보로 DDL 생성 시 컬럼 길이 값으로 활용 (`varchar 10`)
- JPA에서 `public`  또는 `protected`의 기본 생성자 필수
## 엔티티 매니저
`EntityManager em`
- 내부에 데이터 소스를 가지고 있음
- 데이터베이스에 접근 가능
`@Transactional`
- JPA의 모든 데이터 변경은 트랜잭션 안에서 이루어져야 함
- 조회는 트랜잭션 없이 가능
- 일반적으로 비즈니스 로직을 시작하는 서비스 계층에 트랜잭션 추가

> - JPA를 설정할 경우 `EntityManagerFactory`, JPA 트랜잭션 매니저 (`JpaTransacitonManager`), 데이터 소스 등등 다양한  설정이 필요하지만 스프링 부트가 위 과정을 모두 자동화
> - 스프링 부트 자동 설정은 `JpaBaseConfiguration` 에서 담당

# Query 방법
## 1️⃣ JPQL
### 개념
- Java Persistence Query Language
- 여러 데이터를 복잡한 조건으로 조회할 때 사용
### 사용
- 파라미터 입력 : `where price <= :maxPrice`
- 파라미터 바인딩 : `query.setParameter("maxPrice", maxPrice)`
### 예시
```java
@Test
public void jpql() {
	String query = 
		"select m from Member m " + 
		"where m.age between 20 and 40 " +
		" and m.name like '김%'" + 
		"order by m.age desc";

	List<Member> resultList = entityManager.createQuery(query, Member.class)
												.setMaxResults(3).getResultList();
}
```

### 장점
- SQL Query와 비슷해서 익숙함
### 단점
- type-safe 하지 않음
- 동적 쿼리 생성 어려움
## 2️⃣ Criteria API

### 예시
```java
@Test
public void jpaCriteriaQuery() {

	CriteriaBuilder cb = entityManager.getCriteriaBuilder();
	CriteriaQuery<Member> cq = cb.createQuery(Member.class);
	Root<Member> root = cq.from(Member.class);

	Path<Integer> age = root.get("age");
	Predicate between = cb.between(age, 20, 40);

	Path<String> path = root.get("name");
	Predicate like = cb.like(path, "김%");

	CriteriaQuery<Member> query = cq.where(cb.and(between, like));
	query.orderBy(cb.desc(age));

	List<Member> resultList = entityManager.createQuery(query).setMaxResults(3).getResultList();
}
```
### 장점
- 동적 쿼리 생성이 JPQL보다는 쉬움
### 단점
- type-safe 하지 않음
- 복잡함
- 알아야 할 것이 많음
## 3️⃣ MetaModel Criteria API
### 소개
- Criteria API + MetaModel
- Criteria API와 거의 동일
### 사용법
`root.get("age") -> root.get(Member_.age)`
### 장점
- type-safe
### 단점
- 복잡함
## 4️⃣ [[Querydsl]]
# 예외 변환
- `EntityManager`는 순수한 JPA 기술이고, 스프링과 관계가 없기 때문에 JPA 관련 예외를 발생 시킨다
	- `PersistenceException`과 그 하위 예외
- JPA 예외를 `@Repository`가 스프링 예외 추상화 (`DataAccessException`)으로 변환
## `@Repository`
- 컴포넌트 스캔의 대상
- 예외 변환 AOP의 적용 대상
	- 스프링과 JPA를 함께 사용하는 경우 스프링은 JPA 예외 변환기 (`PersistenceExceptionTranslator`) 등록
	- 예외 변환 AOP 프록시는 JPA 관련 예외가 발생하면 JPA 예외 변환기를 통해 발생한 예외를 스프링 데이터 접근 예외로 변환

 >- 스프링부트는 `PersistenceExceptionTranslationPostProcessor`를 자동으로 등록하며, 여기에서 `@Repository`를 AOP 프록시로 만드는 어드바이저가 등록
## 결과
**예외 변환 전**
![](https://i.imgur.com/K3g27CY.png)
**예외 변환 후**
![](https://i.imgur.com/cLg2tUs.png)
- 실제 JPA 예외를 변환하는 코드 : `EntityManagerFactoryUtils.convertJpaAccessExceptionIfPossible()`
