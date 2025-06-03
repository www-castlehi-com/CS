# 등장 이유
## 과거
![](https://i.imgur.com/gtfkoYM.png)
- 애플리케이션을 개발할 때 중요한 데이터는 대부분 데이터베이스에 보관
![](https://i.imgur.com/5DLap8n.png)
1. **커넥션 연결** : TCP/IP를 사용해 연결
2. **SQL 전달** : 애플리케이션 서버 -> DB 전달
3. **결과 응답** : DB -> 애플리케이션 서버
![](https://i.imgur.com/xxQ40aw.png)
- 각각의 데이터베이스마다 커넥션을 연결하는 방법, SQL을 전달하는 방법, 결과를 응답 받는 방법이 모두 다름
- 데이터베이스를 다른 종류의 데이터베이스로 변경할 경우 데이터베이스 사용 코드도 함께 변경해야 함
- 각각의 데이터베이스에 연결하는 방법, SQL을 전달하는 방법, 결과를 응답 받는 방법을 새로 학습해야 함
## JDBC 표준 인터페이스
### JDBC란
- JDBC (Java Database Connectivity)는 자바에서 데이터베이스에 접속할 수 있도록 하는 자바 API
- 데이터베이스에서 자료를 쿼리하거나 업데이트하는 방법 제공
![](https://i.imgur.com/XGVsPD3.png)
- `java.sql.Connection` : 연결
- `java.sql.Statement` : SQL을 담은 내용
- `java.sql.ResultSet` : SQL 요청 응답
### JDBC 드라이버
- JDBC 인터페이스를 각각의 DB 벤더(회사) 에서 자신의 DB에 맞도록 구현해서 라이브러리로 제공한 것
	- MySQL JDBC 드라이버 : MySQL DB에 접근
	- Oracle JDBC 드라이버 : Oracle DB에 접근
#### MySQL JDBC 드라이버
![](https://i.imgur.com/LYCInhG.png)
#### Oracle JDBC 드라이버
![](https://i.imgur.com/xEr4tCy.png)
### 장점
- 애플리케이션 로직은 JDBC 표준 인터페이스에만 의존
- 데이터베이스를 변경할 경우 JDBC 구현 라이브러리만 변경
- JDBC 표준 인터페이스 사용법만 학습
### 한계
- 각각의 데이터베이스마다 SQL, 데이터 타입 등의 일부 사용법이 다름
- ANSI SQL 표준이 있지만 일반적인 부분만 공통화했기 때문에 한계가 있음
	- ex) 페이징 SQL
> JPA (Java Persistence API)를 사용할 경우 해당 부분 다수 해결 가능
# 최신 데이터 접근 기술
## 기술 종류
### 1️⃣ 직접 사용
![](https://i.imgur.com/xeFp2BW.png)
### 2️⃣ SQL Mapper
![](https://i.imgur.com/7enJRaS.png)
#### 장점
- JDBC를 편리하게 사용하도록 도와줌
- SQL 응답 결과를 객체로 변환
- JDBC의 반복 코드 제거
#### 단점
- 개발자가 SQL을 직접 작성
#### 대표 기술
- Jdbc Template
- MyBatis
### 3️⃣ ORM 기술
![](https://i.imgur.com/8zqIYgY.png)
- 객체를 관계형 데이터베이스 테이블과 매핑해주는 기술
#### 장점
- 반복적인 SQL을 직접 작성하지 않고, ORM 기술이 개발자 대신에 SQL을 동적으로 만들어 실행
- 각각의 데이터베이스마다 다른 SQL을 사용하는 문제도 중간에서 해결
#### 대표 기술
- JPA 
- 하이버네이트
- 이클립스링크
## 비교
### SQL Mapper vs ORM
- SQL Mapper
	- 개발자가 SQL만 직접 작성
- ORM
	- SQL 작성 X
	- 사용성 어려움
# JDBC Driver Manager
- 라이브러리에 등록된 DB 드라이버들을 관리하고, 커넥션 획득
## 커넥션 인터페이스
![](https://i.imgur.com/fiqpWAL.png)
- JDBC는 `java.sql.Connection` 표준 커넥션 인터페이스 정의
- H2 데이터베이스 드라이버는 JDBC Connection 인터페이스를 구현한 `org.h2.jdbc.JdbcConnection` 구현체 제공
## 요청 흐름
![](https://i.imgur.com/QZOsjI2.png)
1. 애플리케이션 로직에서 커넥션 필요 요청 -> `DrierManager.getConnection()` 호출
2. `DriverManager`는 라이브러리에 등록된 드라이버 목록을 자동으로 인식하며, 드라이버들에게 순서대로 커넥션을 획득할 수 있는지 확인
	- URL
	- 이름, 비밀번호 등 접속에 필요한 추가 정보
3. 찾은 커넥션 구현체를 클라이언트에게 반환
# 개발
## 커넥션 획득
- `getConnection()` : 데이터베이스 커넥션 획득
## SQL 전달
- `sql` : 데이터베이스에 전달할 SQL을 정의
	- `statement`를 사용할 경우 사용자가 악의적으로 `1=1`을 주입해 SQL Injection이 발생할 수 있음 -> `prepareStatment` 사용
- `con.prepareStatement(sql)`
	- 데이터베이스에 전달할 SQL과 파라미터로 전달할 데이터 준비
	- `pstmt.setString()`, `pstmt.setInt()` : SQL의 인자값에 파라미터 준비
- `pstmt.executeUpdate()`
	- `Statement`를 통해 준비된 SQL을 커넥션을 통해 실제 데이터베이스에 전달
	- 영향받은 DB row 수를 `int`로 반환
##  리소스 정리
- 쿼리 실행 후 리소스 역순으로 정리
- 항상 수행되어야 하므로 `finally` 구문에 작성
	- 리소스 누수 유의
## 쿼리 실행
- `executeQuery()`
	- 데이터 조회 시 사용
	- 결과를 `ResultSet`에 담아 반환
## ResultSet
![](https://i.imgur.com/6LVmwAW.png)
- 위와 같은 데이터 구조
- select 쿼리의 결과가 순서대로 입력
![](https://i.imgur.com/Q92IWCf.png)
- 내부에 있는 커서 (`cursor`)를 이용해 데이터 조회
- `rs.next()`
	- 호출 시 커서가 다음으로 이동
	- 최초 커서는 데이터를 가리키고 있지 않기 때문에 최초 한번은 호출해야 데이터 조회 가능
	- `true` : 커서의 이동 결과 데이터가 있음
	- `false` : 더이상 커서가 가리키는 데이터가 없음S