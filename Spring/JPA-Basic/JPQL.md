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
### 5️⃣ 타입
- 문자
  ex. HELLO, She"s
- 숫자
  ex. 10L, 10D, 10F
- Boolean : TRUE, FALSE
- ENUM
- 엔티티 타입
	- 상속 관계에서 사용
  ex. `TYPE(m) = Member`
### 6️⃣ 조건식
- CASE 식
  ex. `select case when m.age <= 10 then '학생요금' when m.age >= 60 then '경로요금' else '일반요금' end from Member m`
- COALESCE : null이 아니면 반환
  ex. `select coalesce(m.username, '이름 없는 회원') from Member m`
- NULLIF : 두 값이 같으면 null, 다르면 첫번째 값 반환
  ex. `select nullif(m.username, '관리자') from Member m`
## 3. 프로젝션
SELECT 절에 조회할 대상을 지정하는 것이다.
#### 1) 조회 대상
- 엔티티 타입
	- 조회 시 영속성 컨텍스트로 관리된다.
	- 연관관계의 엔티티 타입 프로젝션 시 묵시적 프로젝션보다 join을 명시적으로 지정하는 것이 좋다.
  ex. `SELECT m FROM Member m`, `SELECT m.team FROM Member m`
- 임베디드 타입
  ex. `SELECT m.address FROM Member m`
- 스칼라 타입
  ex. `SELECT m.username, m.age FROM Member m`
#### 2) 중복 제거
DISTINCT로 중복을 제거한다.
#### 3) 여러 값 조회
`SELECT m.username, m.age FROM Member m`처럼 여러 값을 조회할 수 있다.
- Query 타입으로 조회
- Object[] 타입으로 조회
- new 명령어로 조회
	- 단순 값을 DTO로 바로 조회할 수 있으나, 순서와 타입이 일치하는 생성자가 필요하다.
  ```java
  List<MemberDTO> result = em.createQuery("select new MemberDTO(m.username, m.age) from Member m", MemberDTO.class).getResultList();
  ```
## 4. 페이징
JPA는 페이징을 두 API로 추상화한다.

```java
List<Member> result = em.createQuery("select m FROM Member m order by m.age desc", Member.class)  
       .setFirstResult(1)  
       .setMaxResults(10)  
       .getResultList();
```
-  `setFirstResult(int startPosition)`
	- 조회 시작 위치이다.
	- 0부터 시작한다.
- `setMaxResult(int maxResult)`
	- 조회할 데이터 수이다.
## 5. 조인
### 1) 종류
- 내부 조인
  ex. `SELECT m FROM Member m [INNER] JOIN m.team t`
- 외부 조인
  ex. `SELECT m FROM Member m LEFT [OUTER] JOIN m.team t`
- 세타 조인
  ex. `SELECT cuont(m) FROM Member m, Team t WHERE m.username = t.name`
### 2) ON절
- 조인 대상 필터링
  ex. `SELECT m, t FROM Member m LEFT JOIN m.team t on t.name = 'A'`
- 연관관계 없는 엔티티 외부 조인
  ex. `SELECT m, t FROM Member m LEFT JOIN Team t on m.username = t.name`
## 6. 서브 쿼리
하나의 SQL 쿼리 문장 내부에 또 다른 SELECT 문을 포함한다.
### 지원 함수
- `[NOT] EXISTS (subquery)` : 서브쿼리에 결과가 존재하면 참이다.
- `{ALL | ANY | SOME} (subquery)`
	- ALL : 모두 만족하면 참
	- ANY, SOME : 조건을 하나라도 만족하면 참
- `[NOT] IN (subquery)` : 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참
### 한계
JPA는 WHERE, HAVING 절에서만 서브 쿼리 사용이 가능하고, 하이버네이트에서 SELECT 절도 지원하며 이것도 가능하다.

하지만, Hibernate 6 이전 버전에서는 FROM 절의 서브 쿼리는 불가능하다. 따라서 가능하다면 조인으로 풀어 해결한다.
(Hibernate 6+ 에서는 가능)
