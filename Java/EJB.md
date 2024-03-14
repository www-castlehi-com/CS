# EJB란?
![](https://i.imgur.com/L8umsS6.png)
- Enterprise JavaBeans
- 자바 EE(Enterprise Edition) 애플리케이션을 개발하기 위한 서버측 컴포넌트 모델
- 애플리케이션의 업무 로직을 가지고 있음
- 분산 환경에서도 안정적으로 실행 가능
# EJB 컴포넌트
## 1️⃣ Session Beans
### Stateless Session Beans
```java
import javax.ejb.Stateless;

@Stateless
public class CalculatorService {

    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }
}
```
- 클라이언트 요청마다 새로운 인스턴스가 생성될 필요가 없음
- 상태 정보 유지 X
- 각 요청은 독립적으로 처리
### Stateful Session Beans
```java
import javax.ejb.Stateful;

@Stateful
public class ShoppingCartBean {

    private List<String> items = new ArrayList<>();

    public void addItem(String item) {
        items.add(item);
    }

    public List<String> getItems() {
        return items;
    }

    public void checkout() {
        items.clear();
    }
}
```
- 특정 클라이언트 세션과 연결되어 상태 정보 유지
- 클라이언트와 빈 사이의 대화 상태 유지
- 복잡한 트랜잭션 처리에 적합
## 2️⃣ Message-Driven Beans
```java
import javax.ejb.MessageDriven;
import javax.jms.Message;
import javax.jms.MessageListener;

@MessageDriven(mappedName = "jms/queue/test")
public class SimpleMessageBean implements MessageListener {

    public void onMessage(Message message) {
        System.out.println("Received message: " + message);
    }
}
```
- 비동기 메시지 (***JMS : Java Message Service***) 처리
- 메시지가 도착하면 반응하여 해당 메시지를 기반으로 비즈니스 로직 수행
## 3️⃣ Entity Beans
- 데이터베이스의 데이터를 관리
- Insert, Update, Delete, Select
- DB관련 쿼리가 자동으로 만들어지고 개발자는 비즈니스 로직에만 집중할 수 있음
- DB가 수정되면 코드 수정 없이 다시 배포
- JavaEE 6부터 일반적으로 JPA를 사용
# EJB 컨테이너
## 트랜잭션 관리
```java
import javax.ejb.Stateless;
import javax.ejb.TransactionAttribute;
import javax.ejb.TransactionAttributeType;

@Stateless
public class AccountService {

    @TransactionAttribute(TransactionAttributeType.REQUIRED)
    public void deposit(Long accountId, Double amount) {
        // 계좌 입금 로직
    }

    @TransactionAttribute(TransactionAttributeType.REQUIRES_NEW)
    public void transfer(Long fromAccountId, Long toAccountId, Double amount) {
        // 계좌 이체 로직
    }
}
```
트랜잭션이 자동 처리되며, 분산 환경에서는 분산 트랜잭션을 지원

```java
public enum TransactionAttributeType {  
     MANDATORY,  
     REQUIRED,
     REQUIRES_NEW,
     SUPPORTS,
     NOT_SUPPRTED,
     NEVER
}
```
### 1️⃣ MANDATORY
```java
@Stateless
public class TransferService {

    @TransactionAttribute(TransactionAttributeType.MANDATORY)
    public void withdraw(Long accountId, double amount) {
        // 출금 로직
    }

    @TransactionAttribute(TransactionAttributeType.MANDATORY)
    public void deposit(Long accountId, double amount) {
        // 입금 로직
    }

    public void transfer(Long fromAccountId, Long toAccountId, double amount) {
        // 이체 처리 로직: 이 메소드는 외부에서 이미 시작된 트랜잭션 내에서 호출되어야 함
        withdraw(fromAccountId, amount);
        deposit(toAccountId, amount);
    }
}
```
- 메소드가 호출될 때 이미 실행 중인 트랜잭션이 있어야 함
- 실행 중인  트랜잭션이 없다면 예외 발생
- 은행 계좌 이체 등 출금과 입금하는 작업은 상위 트랜잭션의 일부로만 실행 가능
### 2️⃣REQUIRED
```java
@TransactionAttribute(TransactionAttributeType.REQUIRED)
public void updateUserDetails(User user) {
    // 사용자 세부 정보 업데이트 로직
}
```
- 현재 실행 중인 트랜잭션이 있으면 그 트랜잭션을 사용하고, 없으면 새로운 트랜잭션 시작
- 기본 설정으로 대부분의 비즈니스 로직에서 사용
### 3️⃣ REQUIRES_NEW
```java
@TransactionAttribute(TransactionAttributeType.REQUIRES_NEW)
public void logTransaction(String action) {
    // 트랜잭션 로깅 로직
}
```
- 메소드를 호출할 때마다 새로운 트랜잭션 시작
- 이미 실행 중인 트랜잭션이 있더라도, 새로운 트랜잭션 시작
- 로그 기록 등
### 4️⃣SUPPORTS
```java
@TransactionAttribute(TransactionAttributeType.SUPPORTS)
public User findUserById(Long id) {
    // 사용자 조회 로직
}
```
- 트랜잭션 내에서 실행될 수도 있고, 트랜잭션이 없는 상태에서도 실행될 수 있음
- 호출하는 측의 트랜잭션 상태에 따라 달라짐
- 트랜잭션이 필수적이지 않은 읽기 전용 작업에 적합
### 5️⃣ NOT_SUPPORTED
```java
@TransactionAttribute(TransactionAttributeType.NOT_SUPPORTED)
public void generateReport() {
    // 트랜잭션 없이 보고서 생성 로직
}
```
- 트랜잭션 없이 실행
- 호출 시점에 실행 중인 트랜잭션이 있다면 트랜잭션 일시 중지
- 트랜잭션 오버헤드를 피하고 싶을 때 사용
### 6️⃣NEVER
```java
@TransactionAttribute(TransactionAttributeType.NEVER) 
public void loadInitialData() { 
	// 초기 구성 데이터 로딩 로직: 이 메소드는 트랜잭션 없이 실행되어야 함 
}
```
- 메소드가 트랜잭션 없이 호출되어야 함
- 초기 설정, 구성 데이터 로딩에 적합
### 보안
```java
import javax.annotation.security.RolesAllowed;
import javax.ejb.Stateless;

@Stateless
public class AdminService {

    @RolesAllowed("ADMIN")
    public void performAdminOperation() {
        // 관리자 작업
    }
}
```
역할 기반의 접근 제어를 쉽게 구현
인증, 권한 부여, 데이터 암호화 등의 보안 기능 제공
### 스레드 관리
EJB 컨테이너는 스레드 풀 사용해서 EJB 컴포넌트에 대한 클라이언트 요청 관리
클라이언트로부터 요청이 들어오면, 스레드 풀에서 스레드를 할당하여 해당 요청 처리하고, 작업 완료 시 스레드 반환
EJB 컨테이너에 의해 자동 관리
#### 비동기 처리
```java
import javax.ejb.Asynchronous;
import javax.ejb.Stateless;

@Stateless
public class EmailService {

    @Asynchronous
    public void sendEmail(String toAddress, String subject, String body) {
        // 이메일 발송 로직 구현
        System.out.println("Sending email to: " + toAddress);
        // 실제 이메일을 보내는 작업을 여기에 구현...
    }
}
```
스레드 관리를 직접 하지 못할 경우 비동기 처리를 위해 `@Asynchronous` 어노테이션 사용
별도의 스레드에서 실행되며 클라이언트는 메소드가 완료될 때까지 기다리지 않고 즉시 반환받음
