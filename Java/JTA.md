# JTA란?
- [[XA]] 인터페이스를 Java 언어로 추상화한 것
- XA 트랜잭션을 자바 프로그램에서 사용할 수 있게 해주는 Java API
## 구현체
- Atomikos
- Bitronix
- Narayana
- Glassfish / Payara Server
- Apache Geronimo
## 구성 요소
### 1️⃣ UserTransaction
- JTA의 인터페이스
- 애플리케이션에서 트랜잭션을 제어할 수 있는 방법 제공
	> ex) `begin`, `commit`, `rollback`
### 2️⃣ TransactionManager
- 백엔드 시스템에서 트랜잭션을 관리하는 컴포넌트
- 트랜잭션 상태 관리, XA 트랜잭션 조정 등
### 3️⃣ JtaTransactionManager
- Spring의 `PlatformTransactionManager` 구현체
- `UserTransaction`과 `TransactionManager` 래핑
- 선언적 트랜잭션인 `@Transactional`과 통합

```java
@Bean
public UserTransaction userTransaction() throws Throwable {
    // Atomikos 또는 Bitronix UserTransaction 구현체 인스턴스화
}

@Bean
public TransactionManager transactionManager() throws Throwable {
    // Atomikos 또는 Bitronix TransactionManager 구현체 인스턴스화
}

@Bean
public JtaTransactionManager jtaTransactionManager() throws Throwable {
    JtaTransactionManager transactionManager = new JtaTransactionManager();
    transactionManager.setUserTransaction(userTransaction());
    transactionManager.setTransactionManager(transactionManager());
    return transactionManager;
}
```


> 💡 JPA의 구현체인 hibernate를 쓰는 것처럼 JTA 또한 사용자가 직접 구현하는 것이 아닌 구현체를 쓰는게 일반적
## 사용
### 1️⃣ 순수 JAVA 환경
```java
import com.atomikos.icatch.jta.UserTransactionManager;
import com.atomikos.jdbc.AtomikosDataSourceBean;
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.PreparedStatement;

public class JavaJTAExample {

    public static void main(String[] args) {
        UserTransactionManager transactionManager = new UserTransactionManager();
        try {
            transactionManager.init();
            
            AtomikosDataSourceBean dataSource = new AtomikosDataSourceBean();
            dataSource.setUniqueResourceName("mysqlXADS");
            dataSource.setXaDataSourceClassName("com.mysql.cj.jdbc.MysqlXADataSource");
            dataSource.setXaProperties(new Properties() {{
                setProperty("user", "dbuser");
                setProperty("password", "dbpassword");
                setProperty("serverName", "localhost");
                setProperty("port", "3306");
                setProperty("databaseName", "mydb");
                setProperty("pinGlobalTxToPhysicalConnection", "true");
            }});

            transactionManager.begin();
            
            try (Connection conn = dataSource.getConnection();
                 PreparedStatement statement = conn.prepareStatement("INSERT INTO your_table (column_name) VALUES (?)")) {
                statement.setString(1, "value");
                statement.executeUpdate();
            }

            transactionManager.commit();
        } catch (Exception e) {
            try {
                transactionManager.rollback();
            } catch (Exception ex) {
                ex.printStackTrace();
            }
            e.printStackTrace();
        } finally {
            transactionManager.close();
        }
    }
}
```
### 2️⃣ Spring 환경
***설정 예시***
```java
@Configuration
public class SpringConfig {

    @Bean(initMethod = "init", destroyMethod = "close")
    public AtomikosDataSourceBean dataSource() {
        AtomikosDataSourceBean dataSource = new AtomikosDataSourceBean();
        dataSource.setUniqueResourceName("mysqlXADS");
        dataSource.setXaDataSourceClassName("com.mysql.cj.jdbc.MysqlXADataSource");
        dataSource.setXaProperties(new Properties() {{
            setProperty("user", "dbuser");
            setProperty("password", "dbpassword");
            setProperty("serverName", "localhost");
            setProperty("port", "3306");
            setProperty("databaseName", "mydb");
            setProperty("pinGlobalTxToPhysicalConnection", "true");
        }});
        return dataSource;
    }

    @Bean
    public JtaTransactionManager transactionManager() {
        UserTransactionManager userTransactionManager = new UserTransactionManager();
        userTransactionManager.init();
        return new JtaTransactionManager(userTransactionManager);
    }
}
```
***사용 예시***
```java
@Service
public class YourService {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Transactional
    public void addYourEntity(String columnValue) {
        jdbcTemplate.update("INSERT INTO your_table (column_name) VALUES (?)", columnValue);
    }
}
```