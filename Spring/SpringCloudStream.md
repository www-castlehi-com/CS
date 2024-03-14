# Spring Cloud란?
![](https://i.imgur.com/y5e6raQ.png)
[[MSA]]를 구축하기 위한 도구와 라이브러리 집합 (프레임워크)
## RAD 방법론
- 빠르게 애플리케이션을 개발하기 위한 방법론
- Spring Cloud는 서비스 디스커버리, 로드밸런싱, 구성 관리, 회로 차단, 분석 추적 등의 기능을 제공하여 개발자가 비즈니스 로직에 더 집중할 수 있도록 도와주기 때문에 RAD 방법론 사용
## vs SpringBoot
- SpringBoot는 단일 애플리케이션 개발을 위한 프레임워크
- SpringCloud는 SpringBoot에서 제공하는 기능 + MSA에서 필요한 다양한 기능

> 💡 SpringCloud라는 환경 하에 각각의 단일 애플리케이션은 Spring Boot를 이용하여 개발
# Spring Cloud Stream이란?
![](https://i.imgur.com/wrjlU4y.png)
- Spring Cloud 중 하나로 이벤트 중 MSA 어플리케이션을 위한 프레임워크
- 메시징 시스템 ([[Kafka]])에 대한 구체적인 지식이 없어도 추상화 계층 제공 -> 비즈니스 로직을 Kafka나 RabitMQ 등에 연결
## 특징
### 바인딩
- 입/출력을 Kafka와 같은 미들웨어에 연결하기 위한 Bridge
- 개발자가 브로커의 세부 사항을 몰라도 메시지를 송수신할 수 있도록 함
### Consumer group
![](https://i.imgur.com/NBaYQJS.png)
- 하나의 Consumer는 하나 이상의 Consumer Group에 속해 있어야 함
- 그룹 단위로 데이터를 읽으며 하나의 Group에 한 번의 데이터 전달만 하면 됨
### 파티셔닝
![](https://i.imgur.com/dSc9jPm.png)
- 한 개 이상의 Producer가 다수의 Consuer에게 데이터를 보내도 해당 데이터를 인지하고 계속해서 같은 Consumer가 해당 데이터를 처리하도록 보장
- 파티셔닝을 사용하는 Kafka 뿐만 아니라 이를 제공하지 않는 RabbitMP에서도 파티셔닝 사용 가능
## 예시
### 1️⃣ Kafka, Stream 바인딩 설정
```yaml
spring:
  cloud:
    stream:
      kafka:
        binder:
          brokers: localhost:9092
      bindings:
        inputChannel:
          destination: test-topic
          group: test-group
        outputChannel:
          destination: test-topic
```
### 2️⃣ 채널 정의
```java
public interface MyMessageProcessor {
    String INPUT = "inputChannel";
    String OUTPUT = "outputChannel";

    @Input(MyMessageProcessor.INPUT)
    SubscribableChannel inputChannel();

    @Output(MyMessageProcessor.OUTPUT)
    MessageChannel outputChannel();
}
```
### 3️⃣ 리스너 생성
```java
@EnableBinding(MyMessageProcessor.class)
public class MyMessageListener {

    @StreamListener(MyMessageProcessor.INPUT)
    @SendTo(MyMessageProcessor.OUTPUT)
    public String processMessage(String message) {
        System.out.println("Received message: " + message);
        return "Processed message: " + message;
    }
}
```
### 4️⃣ 컨트롤러 생성
```java
@RestController
public class MyMessageController {

    private final MessageChannel outputChannel;

    public MyMessageController(MyMessageProcessor myMessageProcessor) {
        this.outputChannel = myMessageProcessor.outputChannel();
    }

    @GetMapping("/send")
    public String sendMessage(@RequestParam("msg") String message) {
        outputChannel.send(MessageBuilder.withPayload(message).build());
        return "Message sent: " + message;
    }
}
```

1. **메시지 발송 요청**
	- `/send?msg=msg_exaple` 엔드포인트로 HTTP GET 요청 
	- kafka토픽에 msg_example 발행 시도
2. **REST 컨트롤러 처리**
	- `MyMessageController`의 `sendMessage` 메서드가 호출
	- SpringCloudStream의 출력 채널인 `outputChannel`로 메시지 보냄
	- `MessageBuilder`를 사용하여 메시지를 빌드하고, `send` 메서드를 통해 메시지 발송
3. **메시지 발행**
	- `outputChannel`은 `MyMessageProcessor` 인터페이스에 정의된 출력 채널과 바인딩
	- `application.yml` 설정에 따라 Kafka의 `test-topic` 토픽으로 메시지 발행
4. **메시지 수신, 처리**
	- `MyMessageListener` 클래스에 정의된 `processMessage` 메서드가 메시지 수신
	- `@EnableBinding` 어노테이션을 사용하여 `MyMessageProcessor` 인터페이스에 바인딩
	- `@StreamListener(MyMessageProcessor.INPUT)` 어노테이션을 통해 입력 채널(`inputChannel`)이 `test-topic` 토픽에서 메시지 수신

ex) 주문 정보를 Kafka를 통해 output하고, 재고 서비스에서 재고를 input함
