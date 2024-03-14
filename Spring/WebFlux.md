# WebFlux란?
- 웹 애플리케이션을 개발하기 위한 모델
- [[MSA]]의 통신 지원
- 스프링 5.0부터 추가
- 반응 스트림 back pressure를 지원
- Netty, underow 및 서블릿 컨테이너와 같은 서버에서 실행
## 반응형 스트림
- 논블로킹 back pressure 사용하여 비동기식 스트림 처리에 대한 표준을 제공
- 다른 스레드나 스레드 풀로 요청을 전달할 때 수신 측이 임의의 양의 데이터를 버퍼링하지 않도록 보장
### Back Pressure
- 소비자가 처리할 수 있는 속도보다 데이터 생산자가 데이터를 더 빠르게 생성할 경우 발생하는 상황을 관리
- 데이터의 흐름을 제어하여 소비자가 데이터를 처리할 수 있는 속도를 초과하지 않도록 **생산자의 데이터 생성 속도 조절**
- 데이터 처리 시스템이 과부하되는 것을 방지, 시스템의 안전성과 효율성 유지하는데 도움
> ❗조절 불가 하다면,
> (1) 버퍼에 담거나, (2) 버리거나, (3) 실패 처리
### 반응형이란?
1. 변화에 반응함
	> 네트워크 컴포넌트가 I/O 이벤트에 반응, UI 컨트롤러가 마우스 이벤트에 반응 등
2. 논브로킹 : 함수가 완료되었거나 데이터가 사용 가능할 때 알 수 있음
3. back pressure : 생성 데이터 양 조절 
# Reactive API
- WebFlux는 `Reactor`를 라이브러리로 채택
## Reactor
### 1️⃣ MONO
- 0 또는 1개의 요소를 포함할 수 있는 타입
- 최대 한 개의 값, 혹은 에러를 방출할 수 있음
> 데이터베이스에서 단일 레코드를 조회하거나, 단일 결과를 반환하는 API 호출 시 사용

```java
Mono<String> monoString = Mono.just("Hello, Mono!"); 
monoString.subscribe(System.out::println);
```
### 2️⃣ FLUX
- 0개 이상의 요소를 포함할 수 있는 타입
> 데이터베이스에서 여러 레코드를 조회하거나, 스트림을 통해 여러 이벤트를 처리할 시 사용

```java
Flux<String> fluxStrings = Flux.just("Hello", "Reactive", "World"); fluxStrings.subscribe(System.out::println);
```
### 인터페이스
![](https://i.imgur.com/yJx6st6.png)
#### 1. Publisher
```java
public interface Publisher<T> {
    public void subscribe(Subscriber<? super T> s);
}
```
- 데이터 스트림 생성, 발행
- 구독 요청을 받으면 데이터 항목을 `Subscriber`에게 전송
- 데이터의 생산과 소비가 동시에 이루어지지 않아도 됨 -> 비동기
#### 2. Subscriber
```java
public interface Subscriber<T> {
    public void onSubscribe(Subscription s);
    public void onNext(T t);
    public void onError(Throwable t);
    public void onComplete();
}
```
- `Publisher`를 구독하고 그로부터 데이터 수신
- 데이터 항목 (`onNext`), 스트림 완료 (`onComplete`), 에러 (`onError`) 처리
#### 3. Subscription
```java
public interface Subscription {
    public void request(long n);
    public void cancel();
}
```
- `Publisher`와 `Subscriber` 사이의 구독 관계
- `Subscriber`가 데이터 요청의 양을 제어 `request` 하거나 구독을 취소 (`cancel`)
#### 4. Processor
```java
public static interface Processor<T,R> extends Subscriber<T>, Publisher<R> {
}
```
- `Publisher`와 `Subscriber`의 역할 모두 수행
- 입력 스트림을 받아 처리하고, 결과를 새로운 스트림으로 발행

1) **구독** : `Subscriber`가 `Publisher`에 구독 요청
2) **데이터 요청** : `Publisher`로부터 `onSubscribe` 콜백을 통해 `Subscription` 객체를 받은 `Subscriber`는 이 객체를 통해 필요한 데이터 항목의 양 요청
3) **데이터 전송** : `Publisher`는 `Subscriber` 요청에 따라 데이터 항목들을 `onNext` 메소드를 통해 전송
4) **스트림 완료** 혹은 **에러 처리** : 데이터 전송이 성공적일 경우 `onComplete` 이벤트를 통해 스트림 완료를 알리거나, 문제가 발생한 경우 `onError` 이벤트로 에러 알림

```java
DirectProcessor<String> processor = DirectProcessor.create();
processor.subscribe(System.out::println); // Subscriber 역할
processor.onNext("Hello"); // Publisher 역할
processor.onNext("Reactive Streams");
processor.onComplete();
```
