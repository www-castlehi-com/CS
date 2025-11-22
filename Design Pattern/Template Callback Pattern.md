## 1. 정의
공통 흐름(템플릿)과 변하는 부분(콜백)을 분리하여 재사용성과 유연성을 높이는 패턴이다.
GoF 디자인 패턴이 아니며 GoF 디자인 패턴의 템플릿 메서드 패턴과 전략 패턴 기반이다.

### 템플릿 메서드 패턴이란?
부모 클래스가 전체 흐름(템플릿)을 정의하고 자식 클래스가 변하는 부분만 오버라이딩하는 패턴이다.
상속 기반이며, `abstract` 메서드를 오버라이드 해야 한다.

여러 파일을 읽어주는 서비스를 만든다고 가정하자.
서비스는 파일을 열고 닫을 수 있어야하고, 읽을 수 있어야 한다.
지원하고자 하는 파일은 텍스트 파일과 xml 파일이다.
이 때, 텍스트 파일과 xml파일의 열고 닫는 것은 공통적으로 동일하지만, 파일을 한 줄 한 줄 읽는 방법은 다르다.

```java
abstract class FileProcessor {

    public final void process() {
        open();
        read();
        close();
    }

    protected void open() {
        System.out.println("파일 열기");
    }

    protected abstract void read();

    protected void close() {
        System.out.println("파일 닫기");
    }
}
```
위의 조건에 맞게 부모가 될 abstract class를 만들었다.
이 abstract class가 템플릿이 된다.
이 템플릿은 파일을 열고 닫는 공통적인 부분을 처리하지만, 파일을 읽는 부분은 `abstract` 메서드로 만들어 행위를 정의하지 않았다.

```java
class TextFileProcessor extends FileProcessor {
    @Override
    protected void read() {
        // 텍스트 파일 읽기
    }
}

class XmlFileProcessor extends FileProcessor {
	@Override
	protected void read() {
		// XML 파일 읽기
	}
}
```
서비스는 텍스트 파일과 xml 파일을 지원하니 관련 클래스들을 자식 클래스로 만들었다.
그리고 각자 파일을 읽는 부분의 행위를 정의했다.

### 전략 패턴이란?
변하는 부분(전략)을 인터페이스로 분리하고, 객체를 주입해서 실행하는 패턴이다.
조립이 가능하고, 상속 대신 Composition(구성)을 기반으로 한다.

마찬가지로 파일 서비스를 지원하는 서비스를 예시로 들어보자.
우리는 파일을 읽는 행위를 지원해야 하는데, 텍스트 파일인지 xml 파일인지에 따라 구체적인 행위가 달라진다.

```java
interface ReadStrategy {
    void read();
}
```
파일을 읽는 행위를 인터페이스로 구현했다.

```java
class TextReadStrategy implements ReadStrategy {
    @Override
    public void read() {
        // 텍스트 파일 읽기
    }
}

class XmlReadStrategy implements ReadStrategy {
	@Override
	public void read() {
		//XML 파일 읽기
	}
}
```
파일의 종류마다 이 인터페이스를 구현하여 자신들만의 구체적인 행위를 정의한다.

```java
class FileProcessor {

    private final ReadStrategy strategy;

    public FileProcessor(ReadStrategy strategy) {
        this.strategy = strategy;
    }

    public void process() {
        System.out.println("파일 열기");
        strategy.read(); // 전략 실행
        System.out.println("파일 닫기");
    }
}

public class Main {
    public static void main(String[] args) {
        FileProcessor processor = new FileProcessor(new TextReadStrategy());
        processor.process();
    }
}
```
사용할 때에는 이 전략들을 적절히 조립하여 사용한다.

---
자, 다시 템플릿 콜백 패턴으로 돌아오자.
템플릿 콜백 패턴은 템플릿 메서드 패턴과 전략 패턴을 합친 것이다.

앞서 살펴봤듯이 템플릿 메서드  패턴은 흐름이 고정적이다.
부모 클래스가 정의한대로 흐름은 고정되어 있다.
자식 클래스는 그 흐름 내에서 일부 메서드의 내용만 변경했을 뿐이다.

그리고 전략 패턴은 변화되는 것을 주입한다.
매 호출마다 변경하기 용이하다는 것이다.

즉, 템플릿 콜백 패턴은 정해진 흐름(템플릿)대로 작업을 처리하는데, 그 작업을 외부에서 주입(콜백)받는다.

템플릿 콜백 패턴도 파일 서비스 예시로 알아보자.

```java
class FileTemplate {

    public <T> T execute(Supplier<T> callback) {
        System.out.println("파일 열기");
        try {
            return callback.get();  // 콜백 실행 (변하는 부분)
        } finally {
            System.out.println("파일 닫기");
        }
    }
}
```
템플릿은 파일 열기 -> 작업 -> 파일 닫기 순으로 흐름이 정의되었다.
그리고 파일 연 후 닫기 전까지 작업은 함수를 주입받아 실행한다.

```java
public class Main {
    public static void main(String[] args) {
        FileTemplate template = new FileTemplate();

        String result = template.execute(() -> {
            System.out.println("텍스트 파일 읽기");
            return "완료";
        });

        System.out.println(result);
    }
}
```
callback은 위의 예시처럼 텍스트 파일을 읽는 것일 수도 있고, 파일에 직접 내용을 추가하도록 할 수도 있다.

열기 -> 작업 -> 닫기의 흐름은 고정되어 있으나, 작업은 외부에 의해 주입되어 달라질 수 있다.

## 2. 사용 이유
### 1️⃣ 중복 제거
비슷한 흐름인데 특정 부분만 다를 경우 메서드를 모두 작성하면 중복이 굉장히 많아진다.

파일 서비스 예시에서도 파일의 열고 닫는 부분을 텍스트 파일, xml 파일 처리기에 모두 작성할 경우 중복되는 코드가 많아진다.

템플릿 콜백 패턴을 사용하면 중복되는 부분이 없이 템플릿에만 작성하면 되므로 중복 코드가 제거된다.

### 2️⃣ 공통 정책 중앙화
공통으로 처리할 수 있는 부분이 템플릿 클래스에서 중앙 집중적으로 관리되므로 유지 보수성이 좋아진다.

### 3️⃣ OCP
템플릿 코드는 고정이라 변경이 쉽지 않지만 콜백만 바꾸면 새로운 행동이 되므로 확장에는 열려 있는 구조이다.

## 3. 예시
템플릿 콜백 패턴은 실제로 스프링에서도 많이 사용하는 패턴이다.
### 1️⃣ JdbcTemplate
```java
public <T> T execute(ConnectionCallback<T> action) throws DataAccessException {  
    Assert.notNull(action, "Callback object must not be null");  
    Connection con = DataSourceUtils.getConnection(this.obtainDataSource());  
  
    Object var11;  
    try {  
        Connection conToUse = this.createConnectionProxy(con);  
        var11 = action.doInConnection(conToUse);  
    } catch (SQLException var8) {  
        SQLException ex = var8;  
        String sql = getSql(action);  
        DataSourceUtils.releaseConnection(con, this.getDataSource());  
        con = null;  
        throw this.translateException("ConnectionCallback", sql, ex);  
    } finally {  
        DataSourceUtils.releaseConnection(con, this.getDataSource());  
    }  
  
    return var11;  
}
```
JdbcTemplate은 SQL 커넥션을 연결하고, 트랜잭션 처리를 하고, 예외 처리하는 부분을 공통 템플릿으로 사용하고 실제 어떤 sql을 수행할 것인지는 콜백으로 받는다.

### 2️⃣ TransactionTemplate
```java
public <T> T execute(TransactionCallback<T> action) throws TransactionException {  
    Assert.state(this.transactionManager != null, "No PlatformTransactionManager set");  
    PlatformTransactionManager var3 = this.transactionManager;  
    if (var3 instanceof CallbackPreferringPlatformTransactionManager cpptm) {  
        return cpptm.execute(this, action);  
    } else {  
        TransactionStatus status = this.transactionManager.getTransaction(this);  
  
        Object result;  
        try {  
            result = action.doInTransaction(status);  
        } catch (Error | RuntimeException var6) {  
            Throwable ex = var6;  
            this.rollbackOnException(status, ex);  
            throw ex;  
        } catch (Throwable var7) {  
            Throwable ex = var7;  
            this.rollbackOnException(status, ex);  
            throw new UndeclaredThrowableException(ex, "TransactionCallback threw undeclared checked exception");  
        }  
  
        this.transactionManager.commit(status);  
        return result;  
    }  
}
```
트랜잭션을 시작, 커밋, 롤백하는 것은 공통 템플릿에서 관리하고, 어떤 비즈니스 로직에 트랜잭션을 적용할지 콜백으로 받는다.

### 3️⃣ WebClient
```java
webClient.get()
    .uri("/hello")
    .retrieve()
    .bodyToMono(String.class)
    .map(body -> body.toUpperCase());
```
WebClient는 내부적으로 HTTP Request를 생성하고, Netty로 비동기 송수신을 하고 응답 결과에 따라 분기처리를 하며 수신한 ByteBuf를 DataBuffer로 변환한다. 이 DataBuffer의 내용을 디코딩하고 리소스를 정리한다.
이 과정은 WebClient 내부에서 결정되는 고정 흐름이므로 템플릿 역할을 한다.
이 때 사용자는 응답을 어떻게 처리할지 콜백을 보낸다.