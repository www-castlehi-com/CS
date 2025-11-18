## Operator
- `doOnSubscribe` : Publisher가 구독 중일 때 트리거 되는 동작을 추가한다
- `doOnRequest` : Publisher가 요청을 수신할 때 트리거 되는 동작을 추가한다
- `doOnNext` : Publisher가 데이터를 emit할 때 트리거 되는 동작을 추가한다
- `doOnComplete` : Publisher가 성공적으로 완료되었을 때 트리거 되는 동작을 추가한다
- `doOnError` : Publisher가 에러가 발생한 상태로 종료되었을 때 트리거 되는 동작을 추가한다
- `doOnCancel` : Publisher가 취소되었을 때 트리거 되는 동작을 추가한다
- `doOnTerminate` : Publisher가 성공적으로 완료되었을 때 또는 에러가 발생한 상태로 종료되었을 때 트리거 되는 동작을 추가한다
- `doOnEach` : `doOnNext()`, `doOnComplete()`, `doOnError()`의 역할을 한다
- `doOnDiscard` : Upstream에 있는 전체 Operator 체인의 동작 중에서 Operator에 의해 폐기되는 요소를 조건부로 정리한다
- `doAfterTerminate` : Sequence가 성공적으로 완료된 직 후 또는 에러가 발생하여 Seqeuence가 종료된 직 후에 트리거 되는 동작을 추가한다
- `doFirst` : Publisher가 구독 되기 전에 트리거 되는 동작을 추가한다
- `doFinally` : 에러를 포함해서 어떤 이유이든 간에 Publisher가 종료된 후 트리거 되는 동작을 추가한다

#### 예시
```java
public static void main(String[] args) {  
    Flux  
          .range(1, 5)  
          .doFinally(signalType -> Logger.info("doFinally() > range"))  
          .doOnNext(data -> Logger.doOnNext("range", data))  
          .doOnRequest(n -> Logger.info("doOnRequest > range: {}", 1))  
          .doOnSubscribe(subscription -> Logger.info("doOnSubscribe() > range"))  
          .doFirst(() -> Logger.info("doFirst() > range"))  
          .doOnComplete(() -> Logger.info("doOnComplete() > range"))  
          .filter(num -> num % 2 == 1)  
          .doOnNext(data -> Logger.doOnNext("filter", data))  
          .doOnRequest(n -> Logger.info("doOnRequest > filter: {}", 1))  
          .doOnSubscribe(subscription -> Logger.info("doOnSubscribe() > filter"))  
          .doFinally(signalType -> Logger.info("doFinally() > filter"))  
          .doOnComplete(() -> Logger.info("doOnComplete() > filter"))  
          .doFirst(() -> Logger.info("doFirst() > filter"))  
          .subscribe(new BaseSubscriber<>() {  
             @Override  
             protected void hookOnSubscribe(Subscription subscription) {  
                request(1);  
             }  
  
             @Override  
             protected void hookOnNext(Integer value) {  
                Logger.info("# hookOnNext: {}", value);  
                request(1);  
             }  
          });  
}
```
![](https://i.imgur.com/p9tHhMa.png)

---
```java
public static void main(String[] args) {  
    Flux  
          .just("HI", "HELLO")  
          .filter(data -> data.equals("HI"))  
          .doOnTerminate(() -> Logger.doOnTerminate("filter"))  
          .doAfterTerminate(() -> Logger.doAfterTerminate("filter"))  
          .map(data -> data.toLowerCase())  
          .doOnTerminate(() -> Logger.doOnTerminate("map"))  
          .doAfterTerminate(() -> Logger.doAfterTerminate("map"))  
          .subscribe(  
                data -> Logger.onNext(data),  
                error -> {},  
                () -> Logger.doOnComplete());  
}
```
![](https://i.imgur.com/KeH3nBn.png)
