# DataSource란
![](https://i.imgur.com/kvUaNrd.png)
- `java.sql.DataSource`
- **커넥션을 획득하는 방법을 추상화** 하는 인터페이스
	- `DriverManager`를 사용하다가 `HikariCP`와 같은 [[커넥션 풀]]을 사용할 경우 애플리케이션 코드가 변경되어야 함
- 핵심 기능 : **커넥션 조회**
# 인터페이스
```java
public interface Datasource {
	Connection getConnection() throws SQLException;
}
```

> - `DriverManager`는 `DataSource` 인터페이스를 사용하지 않음
> - 스프링은 `DriverManager`도 `DataSource`를 통해서 사용할 수 있도록 `DriverManagerDataSource`라는 `DataSource`를 구현한 클래스 제공
# 종류
## 1️⃣ DriverManager
### vs DataSource
#### DriverManager
```java
DriverManager.getConnection(URL, USERNAME, PASSWORD);
DriverManager.getConnection(URL, USERNAME, PASSWORD);
```
- 커넥션을 획득할 때마다 `URl`, `USERNAME`, `PASSWORD`같은 파라미터를 계속 전달
#### DataSource
```java
void dataSourceDriverManager() throws SQLException {
	DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);

	dataSource.getConnection();
	dataSource.getConnection();
}
```
- 처음 객체를 생성할 때만 필요한 파라미터를 넘기고, 커넥션 획득 시에는 단순 호출

> **설정과 사용 분리**
> - **설정** : 설정과 관련된 속성들은 생성 시에만 사용하는 등 한 곳에 응집
> - **사용** : 설정은 신경쓰지 않고, `getConnection()`만 호출해서 사용
## 2️⃣ 커넥션 풀
### 사용
```java
@Test  
void dataSourceConnectionPool() throws SQLException, InterruptedException {  
    // 커넥션 풀링  
    HikariDataSource dataSource = new HikariDataSource();  
    dataSource.setJdbcUrl(URL);  
    dataSource.setUsername(USERNAME);  
    dataSource.setPassword(PASSWORD);  
    dataSource.setMaximumPoolSize(10);  
    dataSource.setPoolName("MyPool");  
  
    useDataSource(dataSource);  
    Thread.sleep(1000);  
}
```
- 애플리케이션 실행 속도에 영향을 주지 않기 위해 별도의 쓰레드에서 작동
### 실행 결과
```log
#커넥션 풀 초기화 정보 출력

HikariConfig - MyPool - configuration:
HikariConfig - maximumPoolSize................................10
HikariConfig - poolName................................"MyPool"

#커넥션 풀 전용 쓰레드가 커넥션 풀에 커넥션을 10개 채움

[MyPool connection adder] MyPool - Added connection conn0: url=jdbc:h2:..user=SA
[MyPool connection adder] MyPool - Added connection conn1: url=jdbc:h2:..user=SA
[MyPool connection adder] MyPool - Added connection conn2: url=jdbc:h2:..user=SA
[MyPool connection adder] MyPool - Added connection conn3: url=jdbc:h2:..user=SA
[MyPool connection adder] MyPool - Added connection conn4: url=jdbc:h2:..user=SA
...
[MyPool connection adder] MyPool - Added connection conn9: url=jdbc:h2:..user=SA

#커넥션 풀에서 커넥션 획득1

ConnectionTest - connection=HikariProxyConnection@446445803 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class=class
com.zaxxer.hikari.pool.HikariProxyConnection

#커넥션 풀에서 커넥션 획득2

ConnectionTest - connection=HikariProxyConnection@832292933 wrapping conn1: url=jdbc:h2:tcp://localhost/~/test user=SA, class=class
com.zaxxer.hikari.pool.HikariProxyConnection

MyPool - After adding stats (total=10, active=2, idle=8, waiting=0)
```
#### HikariConfig
- `HikariCP` 관련 설정 확인
#### connection adder
- 별도의 쓰레드를 사용해서 커넥션 풀에 최대 커넥션 풀 수까지 커넥션을 채움

> 커넥션 풀을 다 채울 때까지 대기하고 있으면 애플리케이션 실행 시간이 늦어지기 때문에 별도의 쓰레드에서 처리
#### 커넥션 획득
- 커넥션 2개를 획득하고 반환하지 않을 경우 10개의 커넥션 중에 2개를 가지고 있는 상태(`active=2`)가 되며, 대기 상태인 커넥션은 8개(`idle=8`)
```log
20:49:17.046 [MyPool:connection-adder] DEBUG com.zaxxer.hikari.pool.HikariPool --MyPool - After adding stats (total=10/10, idle=0/10, active=10, waiting=1)
20:49:17.047 [MyPool:connection-adder] DEBUG com.zaxxer.hikari.pool.HikariPool --MyPool - Connection not added, stats (total=10/10, idle=0/10, active=10, waiting=1)
```
- 커넥션 풀을 초과할 경우 `waiting` 상태 돌입
# DI
```java
public class MemberRepositoryV1 {  
  
    private final DataSource dataSource;  
  
    public MemberRepositoryV1(DataSource dataSource) {  
       this.dataSource = dataSource;  
    }
}
```
```java
class MemberRepositoryV1Test {  
  
    MemberRepositoryV1 repository;  
  
    @BeforeEach  
    void setUp() {  
       // 기본 DriverManager - 항상 새로운 커넥션을 획득  
       // DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);  
  
       // 커넥션 풀링  
       HikariDataSource dataSource = new HikariDataSource();  
       dataSource.setJdbcUrl(URL);  
       dataSource.setUsername(USERNAME);  
       dataSource.setPassword(PASSWORD);  
  
       repository = new MemberRepositoryV1(dataSource);  
    }
}
```
- `DriverManagerDataSource` -> `HikariDataSource`로 변경해도 내부 구현 코드는 `DataSource`에만 의존하기 때문에 변경 X