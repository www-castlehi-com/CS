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
## 7. 경로 표현식
점을 찍어 객체 그래프를 탐색하는 것을 의미한다.
- 상태 필드 : 단순히 값을 저장하기 위한 필드
  ex. `m.username`
- 연관 필드 : 연관관계를 위한 필드
	- 단일 값 연관 필드 : `@ManyToOne`, `@OneToOne`, 대상이 엔티티
	  ex. `m.team`
	- 컬렉션 값 연관 필드 : `@OneToMany`, `@ManyToMany`, 대상이 컬렉션
	  ex. `m.orders`
## 8. fetch join
SQL 조인 종류가 아니며 JPQL에서 성능 최적화를 위해 제공하는 기능이다.
연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회할 수 있다.
### 사용
```sql
[LEFT [OUTER] | INNER] JOIN FETCH 조인 경로
```
JPQL로 아래와 같이 작성했다고 해 보자.
```sql
select m from Member m join fetch m.team
```
이 때, 발생하는 SQL은 아래와 같다.
```sql
SELECT M.*, T.* FROM MEMBER M INNER JOIN TEAM T ON M.TEAM_ID=T.ID
```
### 예시
```java
Team teamA = new Team();  
teamA.setName("팀A");  
em.persist(teamA);  
  
Team teamB = new Team();  
teamB.setName("팀B");  
em.persist(teamB);  
  
Member member1 = new Member();  
member1.setUsername("회원1");  
member1.changeTeam(teamA);  
em.persist(member1);  
  
Member member2 = new Member();  
member2.setUsername("회원2");  
member2.changeTeam(teamA);  
em.persist(member2);  
  
Member member3 = new Member();  
member3.setUsername("회원3");  
member3.changeTeam(teamB);  
em.persist(member3);  
  
em.flush();  
em.clear();
```
위의 예시에서 살펴보자.
Member와 Team의 FetchType은 LAZY라고 가정한다.

#### 1) 단순 조회
```java
String query = "select m from Member m";  
List<Member> result = em.createQuery(query, Member.class)  
       .getResultList();  
  
for (Member member : result) {  
    System.out.println("member = " + member.getUsername() + ", " + member.getTeam().getName());  
}
```

```sql
Hibernate: 
    /* select
        m 
    from
        Member m */ select
            m1_0.id,
            m1_0.age,
            m1_0.team_id,
            m1_0.username 
        from
            member m1_0
Hibernate: 
    select
        t1_0.id,
        t1_0.name 
    from
        team t1_0 
    where
        t1_0.id=?
member = 회원1, 팀A
member = 회원2, 팀A
Hibernate: 
    select
        t1_0.id,
        t1_0.name 
    from
        team t1_0 
    where
        t1_0.id=?
member = 회원3, 팀B
```
member를 조회하였지만 `member.getTeam().getName()`에 의해서 Team을 조회하는 SQL이 추가로 발생한다.

만약, member가 100명이라고 조회해보자. 그러면 Member를 조회하는 쿼리 뿐만 아니라 각 Member 별 Team을 조회하는 쿼리가 추가로 발생한다. 
이를 **N+1 문제**라고 한다.
#### 2) fetch join
```java
String query = "select m from Member m join fetch m.team";
```

```sql
Hibernate: 
    /* select
        m 
    from
        Member m 
    join
        
    fetch
        m.team */ select
            m1_0.id,
            m1_0.age,
            t1_0.id,
            t1_0.name,
            m1_0.username 
        from
            member m1_0 
        join
            team t1_0 
                on t1_0.id=m1_0.team_id
member = 회원1, 팀A
member = 회원2, 팀A
member = 회원3, 팀B
```
select 쿼리가 추가로 발생하지 않고 연관 관계를 모두 가져왔다.

반대로 컬렉션을 패치 조인할 때도 sql 한번에 연관 관계를 모두 가져올 수 있다.
하지만, 컬렉션을 패치 조인할 때에는 조심할 점이 있다.
![](https://i.imgur.com/6dHhc8J.png)
join을 할 경우 db 상 데이터가 많아지기 때문에 중복되어서 데이터가 발생할 수 있다.
이 경우 DISTINCT로 중복을 제거할 수 있는데, 하이버네이트6부터는 DISTINCT 명령어를 사용하지 않더라도 애플리케이션에서 중복 제거가 자동으로 적용된다.

애플리케이션에서 중복 제거가 된다는 말을 유의깊게 보자.
![](https://i.imgur.com/1UlZYK2.png)
distinct로 중복이 제거가 되려면 데이터가 완전히 동일해야 한다.
하지만 member와 team을 조인한 결과를 보면 결과는 동일하지 않다. 

그래서 jpa는 같은 식별자를 가진 Team 엔티티를 애플리케이션 단위에서 중복 제거를 하게 된다.
### vs 일반 join
일반 join의 경우 결과를 반환할 때 연관관계를 고려하지 않는다.
fetch join의 경우 연관된 엔티티도 SQL 한번에 조회하여 Eager Loading처럼 동작한다.
### 한계
- fetch join 대상에는 별칭을 줄 수 없다. 하이버네이트에서는 가능하지만 가급적 사용하지 않는다.
- 둘 이상의 컬렉션은 fetch join 할 수 없다.
- 컬렉션을 fetch join하면 페이징 API를 사용할 수 없다.
	- 데이터의 정합성이 깨진 것처럼 보일 수 있기 때문이다.
	- `@OneToOne`, `@ManyToOne` 같은 단일 값 연관 필드들은 fetch join 해도 페이징이 가능하다.
	- 하이버네이트는 경고 로그를 남기고 메모리에서 페이징을 한다. 하지만 매우 위험하다.
### 정리
fetch join은 객체 그래프를 유지하면서 성능 최적화에 상당히 유용하다.
하지만 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 한다면 억지로 fetch join을 사용하기 보다는 여러 테이블에서 필요한 필드들만 조회해서 DTO로 반환하는 것이 더 효과적이다.
## 9. Named 쿼리
미리 정의해서 이름을 부여해두고 사용하는 정적 쿼리이다.
따라서 애플리케이션 로딩 시점에 쿼리를 검증한다.
### 사용법
```java
@Entity  
@NamedQuery(  
       name = "Member.findByUsername",  
       query = "select m from Member m where m.username = :username"  
)  
public class Member {
}
```
```java
List<Member> result = em.createNamedQuery("Member.findByUsername", Member.class)  
       .setParameter("username", "회원1")  
       .getResultList();
```
어노테이션에 설정했지만, 이 외에도 XML에 정의할 수 있다.
XML이 항상 우선권을 가진다.
## 10. 벌크 연산
쿼리 한 번으로 여러 테이블 로우가 변경된다.

벌크 연산은 영속성 컨테이너를 무시하고 데이터베이스에 직접 쿼리가 실행된다.
이를 해결하기 위해서는 두 가지 방법이 있다.
1. 벌크 연산을 먼저 실행한다.
2. 벌크 연산 수행 후 영속성 컨텍스트를 초기화한다.