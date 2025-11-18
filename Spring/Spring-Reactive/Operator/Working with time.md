## 1️⃣ delayElements
![](https://i.imgur.com/qHQkDYE.png)
Upstream 데이터(`onNext` signals)의 emit을 파라미터로 입력한 시간 동안 지연시킨다
Upstream에서 emit 되는 데이터가 없거나(`empty` signals) 에러 발생(`onError` signals)시에는 지연되지 않는다

> 데이터의 emit을 지연시키는 거지 signal 자체를 지연시키는 것은 아니다
#### 예시
```java
public static void main(String[] args) {  
    Flux  
          .range(1, 10)  
          .doOnNext(num -> Logger.doOnNext(num))  
          .delayElements(Duration.ofMillis(500))  
          .subscribe(Logger::onNext);  
  
    TimeUtils.sleep(6000);  
}
```
![](https://i.imgur.com/turCWnJ.png)
`delayElements()`가 `doOnNext()`의 아래 쪽에 위치하므로 `doOnNext()`는 데이터 emit을 즉시 받고, `onNext`는 0.5초의 딜레이 시간을 가진다

---
```java
public static void main(String[] args) {  
    Flux  
          .range(1, 10)  
          .delayElements(Duration.ofMillis(500))  
          .doOnNext(num -> Logger.doOnNext(num))  
          .subscribe(Logger::onNext);  
  
    TimeUtils.sleep(6000);  
}
```
![](https://i.imgur.com/EmsSPVC.png)
`delayElements()`가 `doOnNext()`보다 위 쪽에 위치하므로 `doOnNext()` 또한 0.5 초의 딜레이 타임을 가진다

## 2️⃣ delaySubscription vs 3️⃣ delaySequence
![](https://i.imgur.com/KTelBZw.png)
파라미터로 입력받은 시간만큼 구독이 지연된다
![](https://i.imgur.com/3nCHi5O.png)
구독은 즉시 이루어지지만 파라미터로 입력한 시간 만큼 최초 데이터 emit이 지연된다

#### 예시
**DelaySubscription**
```java
public static void main(String[] args) {  
    Flux  
          .range(1, 10)  
          .doOnSubscribe(subscription -> Logger.info("# doOnSubscribe > upstream"))  
          .delaySubscription(Duration.ofSeconds(2))  
          .doOnSubscribe(subscription -> Logger.info("# doOnSubscribe > downstream"))  
          .subscribe(Logger::onNext);  
  
    TimeUtils.sleep(2500);  
}
```
![](https://i.imgur.com/6RsFR1j.png)
Upstream의 구독이 지연되어 2초 느린 것을 확인할 수 있다

**DelaySequence**
```java
public static void main(String[] args) {  
    Flux  
          .range(1, 10)  
          .doOnSubscribe(subscription -> Logger.doOnSubscribe())  
          .delaySequence(Duration.ofSeconds(2))  
          .doOnSubscribe(subscription -> Logger.doOnSubscribe())  
          .subscribe(Logger::onNext);  
  
    TimeUtils.sleep(2500);  
}
```
![](https://i.imgur.com/1YorSF7.png)
구독은 즉시 이루어지지만 첫 데이터 emit이 2초 지연되어 구독과 데이터 emit의 시간 차이가 있다

## 4️⃣ timeout
![](https://i.imgur.com/NXfsnF8.png)
파라미터로 입력 한 시간 동안 emit되는 데이터가 없으면 `TimeoutException`이 발생한다
즉, `onError()` signal이 발생하면서 구독이 취소된다

#### 예시
```java
public static void main(String[] args) {  
    requestToServer()  
          .timeout(Duration.ofSeconds(2))  
          .subscribe(response -> Logger.onNext(response),  
                error -> Logger.onError(error));  
  
    TimeUtils.sleep(3500);  
}  
  
private static Mono<String> requestToServer() {  
    return Mono.just("complete to process request from client successfully")  
          .delayElement(Duration.ofSeconds(3));  
}
```
![](https://i.imgur.com/6PEmEo2.png)
