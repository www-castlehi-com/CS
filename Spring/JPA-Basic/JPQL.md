## 1. 개념
테이블을 대상으로 쿼리하는 것이 아닌 엔티티 객체를 대상으로 쿼리하는 [[객체지향 쿼리 언어]]이다.
SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않고, 결국 SQL로 변환된다.
## 2. 문법
```sql
select m from Member as m where m.age > 18
```
- 엔티티와 속성은 대소문자를 구분한다.
  ex. Member, age
- JPQL 키워드는 대소문자를 구분하지 않는다.
  ex. SELECT , FROM, where
- 테이블 이름이 아닌 엔티티 이름을 사용한다.
  ex. Member
- 별칭을 필수로 사용하며, as는 생략 가능하다.
### 1️⃣ 집합과 정렬
- GROUP BY, HAVING
- ORDER BY
### 2️⃣ TypeQuery, Query
- **TypeQuery** : 반환 타입이 명확할 때 사용
  ```java
  TypedQuery<Member> query = 
		em.createQuery("SELECT m FROM Member m", Member.class);
  ```
- **Query** : 반환 타입이 명확하지 않을 때 사용
  ```java
  Query query = 
	  em.createQuery("SELECT m.username, m.age from Member m");
  ```
### 3️⃣ 결과 조회
- `query.getResultList()`
	- 결과가 하나 이상일 때 리스트를 반환한다.
	- 결과가 없으면 빈 리스트를 반환한다.
- `query.getSingleResult()`
	- 결과가 정확히 하나여야 하며 단일 객체를 반환해야 한다.
	- 결과가 없으면 `javax.persistence.NoResultException`이 반환되고, 둘 이상이면 `javax.persistence.NonUniqueResultException`이 반환된다.
### 4️⃣ 파라미터 바인딩
- 이름 기준
  ```java
  query.setParameter("username", usernameParam);
  ```
- 위치 기준
  ```java
  query.setParameter(1, usernameParam);
  ```
  위치가 변경되면 순서를 모두 다시 조정해야하기 때문에 잘 사용하지 않는다.