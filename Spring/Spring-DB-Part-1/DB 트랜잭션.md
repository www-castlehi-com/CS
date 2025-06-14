# 데이터베이스 연결 구조와 DB 세션
![](https://i.imgur.com/Qt0btxn.png)
- 사용자는 WAS나 DB 접근 툴 같은 클라이언트를 사용해서 데이터베이스 서버에 접근
- 클라이언트는 데이터베이스 서버에 연결을 요청하고 커넥션을 맺음
- 데이터베이스는 내부에 세션을 생성함
- 해당 커넥션을 통한 모든 요청은 생성한 세션을 통해서 실행
- 세션은 [[트랜잭션]]을 시작하고, 커밋 또는 로백을 통해 트랜잭션 종료하며 이후에 새로운 트랜잭션을 다시 시작
- 사용자가 커넥션을 닫거나, 세션을 강제로 종료하면 세션이 종료됨
![](https://i.imgur.com/DCR0cjY.png)
- 커넥션의 개수마다 세션이 만들어짐
# 동작
- 데이터 변경 쿼리를 실행하고 결과를 반영하려면 `commit`을 호출하고, 결과를 반영하고 싶지 않으면 `rollback`을 호출
![](https://i.imgur.com/dx8IYXT.png)
- 커밋을 호출하기 전까지는 임시로 데이터를 저장
- 해당 트랜잭션을 시작한 세션에게만 변경 데이터가 보이고 다른 세션에게는 변경 데이터가 보이지 않음
	- 격리 수준이 `READ UNCOMMITTED`라면 다른 세션에도 보임
- 다른 세션에 보일 경우 추가한 세션에서 `rollback`을 할 경우 모든 데이터가 처음 상태로 복구되어 데이터 정합성에 문제가 발생함
# 커밋
## 자동 커밋
- 각각의 쿼리 실행 직후에 자동으로 `commit` 호출
- `commit`이나 `rollback`을 직접 호출하지 않아도 됨
## 수동 커밋
- 대부분 자동 커밋 모드가 기본
- 수동 커밋 모드로 설정한는 것을 트랜잭션을 시작한다고 표현
- `commit`, `rollback` 호출
# DB 락
- 트랜잭션을 시작하고 아직 커밋을 수행하지 않았는데 다른 세션에서 같은 데이터를 수정하게 될 경우 격리성이 깨지게 됨
## 동작
![](https://i.imgur.com/2uCqZmg.png)
1. 세션1이 트랜잭션 시작
2. 세션1이 변경하려는 로우의 락을 획득 시도하며, 언락 상태일 경우 락을 얻음
3. 락을 획득하고 해당 로우에 update sql 수행
![](https://i.imgur.com/bWwOasu.png)
4. 세션2가 트랜잭션 시작
5. 세션2는 해당 로우의 락이 없으므로 락이 돌아올 때 까지 대기

> 락을 무한정 대기하지 않음
> 락 대기 시간을 넘어갈 경우 락 타임 오류 발생
![](https://i.imgur.com/ac1TuQy.png)
6. 세션1이 커밋을 수행하고, 트랜잭션이 종료되어 락 반납
![](https://i.imgur.com/MjxDRnM.png)
![](https://i.imgur.com/oGUMYGN.png)
7. 락을 대기하던 세션2가 락 획득
8. update sql 수행
9. 세션2가 커밋을 수행하고 트랜잭션이 종료되면 락 반납
## 조회 락
- 일반적으로 조회 락은 사용하지 않음
- 세션1에서 데이터를 조회한 다음에 이 데이터를 이용하여 애플리케이션에서 계산을 수행하여아 할 때 조회 시에 락 획득
![](https://i.imgur.com/75Z4Yd0.png)
- `select for update` 구문 사용
# 적용
## DB 트랜잭션 사용
### 원리
![](https://i.imgur.com/7RgceMc.png)
- 트랜잭션은 비즈니스 로직이 있는 서비스 계층에서 시작
- 비즈니스 로직이 잘못될 경우 해당 비즈니스 로직으로 인해 문제가 되는 부분을 함께 롤백
- 서비스 계층에서 커넥션을 만들고, 트랜잭션 커밋 이후에 커넥션 종료
- 트랜잭션을 사용하는 동안 같은 커넥션 유지
### 사용
```java
public Member findById(Connection con, String memberId) throws SQLException {  
    //...
  
    try {  
       pstmt = con.prepareStatement(sql);  
       //...
    } catch (SQLException e) {  
       log.error("db error", e);  
       throw e;  
    } finally {  
       // connection은 여기서 닫지 않는다.  
       JdbcUtils.closeResultSet(rs);  
       JdbcUtils.closeStatement(pstmt);  
    }  
}  
  
public void update(Connection con, String memberId, int money) throws SQLException {  
    String sql = "update member set money = ? where member_id = ?";  
  
    PreparedStatement pstmt = null;  
  
    try {  
       pstmt = con.prepareStatement(sql);  
       //...
    } catch (SQLException e) {  
       log.error("db error", e);  
       throw e;  
    } finally {  
       // connection은 여기서 닫지 않는다.  
       JdbcUtils.closeStatement(pstmt);  
    }  
}
```
- `connection` 외부 주입 -> `con = getConnection()` 코드 제외
- 리포지토리에서 커넥션 종료 X
```java
public void accountTransfer(String fromId, String toId, int money) throws SQLException {  
    Connection con = dataSource.getConnection();  
    try {  
       con.setAutoCommit(false); // 트랜잭션 시작  
  
       bizLogic(con, fromId, toId, money);  
  
       con.commit();  
    } catch (Exception e) {  
       con.rollback();  
       throw new IllegalStateException(e);  
    } finally {  
       release(con);  
    }  
}
```
- 서비스 단에서 커넥션 생성 후 repository 단에 동일한 커넥션 전달
- `commit`, `rollback` 이후 커넥션 종료
	- 커넥션 풀에 autoCommit을 false로 설정하고 전달할 경우, 다음에 해당 커넥션을 얻을 때 false 옵션이 적용되어 옵션 되돌리기
### 단점
- 서비스 계층이 지저분
- 코드 복잡
- 커넥션 유지하도록 코드 변경하는 것이 번거로움
### 문제점
#### 1️⃣ 트랜잭션 문제
- JDBC 구현 기술이 서비스 계층에 누수되는 문제
	- 구현 기술을 변경해도 서비스 계층 코드는 최대한 유지할 수 있어야 함
	- 서비스 계층은 특정 기술에 종속되지 않아야 함
- 트랜잭션 동기화 문제
	- 같은 트랜잭션을 유지하기 위해 커넥션을 파라미터로 넘겨야함
	- 똑같은 기능도 트랜잭션 적용 기능과 트랜잭션 비적용 기능으로 분리해야 함
- 트랜잭션 적용 반복 문제
	- `try`, `catch`, `finally` 반복
#### 2️⃣ 예외 누수
- JDBC 구현 기술 예외가 서비스 계층으로 전파
- 향후 데이터 접근 기술이 변경되면, 그에 맞는 예외로 변경해야 하여 서비스 코드도 수정해야 함
#### 3️⃣ JDBC 반복 문제
- 유사 코드 반복
