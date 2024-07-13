## DataSource 설정
```YAML
spring.application.name=Spring_Introduction  
spring.datasource.url=jdbc:h2:tcp://localhost/~/Study/Programming/h2/test  
spring.datasource.driver-class-name=org.h2.Driver
```
JDBC를 이용하기 위해서는 `application.properties`에 Data Source 설정이 선행되어야 함

```java
public class JdbcMemberRepository implements MemberRepository {  
  
    private final DataSource dataSource;  
  
    public JdbcMemberRepository(DataSource dataSource) {  
       this.dataSource = dataSource;  
    }
}
```
JDBC를 이용할 때에는 DataSource를 선언하고, 생성자로 주입받음
이 때, 생성자로 주입해주는 것은 SpringFramework이며, `application.properties`에 설정한 DataSource 내용이 주입됨
## DataSource 연결 / 종료
```java
private Connection getConnection() {  
    return DataSourceUtils.getConnection(dataSource);  
}   
  
private void close(Connection conn) throws SQLException {  
    DataSourceUtils.releaseConnection(conn, dataSource);  
}
```
DataSource 이전에 트랜잭션이 걸릴 수 있음
데이터베이스 연결은 항상 같은 connection으로 연결해야 하기 때문에 `DataSource.getConnection()`보다 `DataSourceUtils`를 사용함
## Config 설정
```java
@Configuration  
public class SpringConfig {   
    @Autowired  
    private DataSource dataSource;
}

@Configuration  
public class SpringConfig {  

	private DataSource  dataSource;

	@Autowired
	public SpringConfig(DataSource dataSource) {  
	    this.dataSource = dataSource;  
	}
}
```
선언부 혹은 생성자에 `@Autowired`를 설정한다면 Configuration을 할 때 SpringFramework에서 주입받을 수 있음
## 테스트
```java
@SpringBootTest  
@Transactional  
class MemberServiceIntegrationTest {
}
```
Database 연결이 필요하므로 `@SpringBootTest`가 필요함
테스트 후에 Database에 영향을 주면 안되므로 `@Transactional`을 걸어 DB에 반영이 되지 않도록 해야함
> 통합테스트도 필요한 테스트이긴 하지만, 단위 테스트의 비중이 훨씬 커야함
## JDBC Template
```java
public class JdbcTemplateMemberRepository implements MemberRepository {  
  
    private final JdbcTemplate jdbcTemplate;  
  
    public JdbcTemplateMemberRepository(DataSource dataSource) {  
       jdbcTemplate = new JdbcTemplate(dataSource);  
    }
}
```
기존 JDBC에서 템플릿 메소드를 통해 반복적인 코드를 개선하여 만든 것