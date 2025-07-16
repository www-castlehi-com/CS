# 주요 기능
## 1️⃣ 공통 인터페이스
![](https://i.imgur.com/cNn4SpP.png)
-  기본적인 CRUD
- 공통화 가능한 기능 포함
### 사용법
```java
public interface ItemRepository extends JpaRepository<Item, Long> {}
```
- `JpaRepository` 인터페이스 상속
- 제네릭에 관리할 `<엔티티, 엔티티ID`> 부여
### 원리
![](https://i.imgur.com/wYxKFkh.png)
- 인터페이스만 상속받으면 스프링 데이터 JPA가 프록시 기술을 사용해서 구현 클래스를 만들고, 인스턴스를 만들어 빈으로 등록
## 2️⃣ 쿼리 메서드
### 사용법
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
	List<Member> findByUsernamdAndAgeGreaterThan(String username, int age);
}
```
- 메서드 이름을 분석해서 필요한 JPQL을 만들고 실행
- JPQL은 JPA가 SQL로 번역해서 실행
```java
public interface SpringDataJpaItemRepository extends JpaRepository<Item, Long> {

	List<Item> findByItemNameLike(String itemName);

	@Query("select i from Item i where i.itemName like :itemName and i.price <= :price)
	List<Item> findItems(@Param("itemName") String itemName, @Param("price") Integer price);
}
```
- 쿼리 메서드 기능 대신에 직접 JPQL을 사용하고 싶을 때에는 `@Query`와 함께 JPQL 작성
	- 메서드 이름으로 실행하는 규칙은 무시됨
- JPA 네이티브 쿼리 기능도 제공하며, JPQL 대신에 SQL을 직접 작성 가능
