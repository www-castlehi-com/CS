## 1️⃣ publish
![](https://i.imgur.com/l01oXpV.png)
구독 시점에 즉시 데이터를 emit하지 않고, `connect()`를 호출하는 시점에 데이터를 emit한다
HotSequence로 변환되기 때문에 구독 시점 이후에 emit된 데이터만 전달받을 수 있다

#### 예시
```java
public static void main(String[] args) {  
    ConnectableFlux<Integer> flux =  
          Flux  
                .range(1, 5)  
                .delayElements(Duration.ofMillis(300L))  
                .publish();  
  
    TimeUtils.sleep(500L);  
    flux.subscribe(data -> Logger.onNext("subscriber1: ", data));  
  
    TimeUtils.sleep(200L);  
    flux.subscribe(data -> Logger.onNext("subscriber2: ", data));  
  
    flux.connect();  
  
    TimeUtils.sleep(1000L);  
    flux.subscribe(data -> Logger.onNext("subscriber3: ", data));  
  
    TimeUtils.sleep(2000L);  
}
```
![](https://i.imgur.com/wu88qBw.png)

## 2️⃣ autoConnect
![](https://i.imgur.com/Gz2ZuOm.png)
파라미터로 지정하는 숫자만큼의 구독이 발생하는 시점에 Upstream 소스에 자동으로 connect 된다
즉, `publish()`처럼 `connect()`를 직접 호출할 필요가 없다

#### 예시
```java
public static void main(String[] args) throws InterruptedException {  
    Flux<String> publisher =  
          Flux  
                .just("Concert part1", "Concert part2", "Concert part3")  
                .delayElements(Duration.ofMillis(300L))  
                .publish()  
                .autoConnect(2);  
  
    Thread.sleep(500L);  
    publisher.subscribe(data -> Logger.info("# audience 1 is watching {}", data));  
  
    Thread.sleep(500L);  
    publisher.subscribe(data -> Logger.info("# audience 2 is watching {}", data));  
  
    Thread.sleep(500L);  
    publisher.subscribe(data -> Logger.info("# audience 3 is watching {}", data));  
  
    Thread.sleep(1000L);  
}
```
![](https://i.imgur.com/XAD0yaH.png)

## 3️⃣ refCount
![](https://i.imgur.com/PpXcN6O.png)
파라미터로 지정한 숫자만큼의 구독이 발생하는 시점에 Upstream 소스에 자동으로 connect된다
파라미터로 지정한 숫자만큼의 구독이 취소되거나 Upstream의 데이터 emit이 종료되면 연결이 해제된다

#### 예시
```java
public static void main(String[] args) throws InterruptedException {  
    Flux<Long> publisher =  
          Flux  
                .interval(Duration.ofMillis(500))  
                .publish().refCount(1);  
    Disposable disposable =  
          publisher.subscribe(data -> Logger.info("# subscriber 1: {}", data));  
  
    Thread.sleep(2100L);  
    disposable.dispose();  
  
    publisher.subscribe(data -> Logger.info("# subscriber 2: {}", data));  
  
    Thread.sleep(2500L);  
}
```
![](https://i.imgur.com/7hRdV0A.png)
subscriber1이 구독 취소 후 upstream과의 연결이 해제되어 subscriber2가 구독할 때 다시 연결된다
만약, `autoConnect()`였다면 연결이 해제되지 않아 다시 연결되는 것이 아니라 이미 실행하고 있던 signal을 그대로 이어 받는다

## 4️⃣ replay
![](https://i.imgur.com/XagImxE.png)
구독 시점에 즉시 데이터를 emit하지 않고 `connect()`를 호출하는 시점에 emit된다
구독 시점 이전에 emit된 데이터를 다시 전달받을 수 있다

#### 예시
```java
public static void main(String[] args) {  
    ConnectableFlux<Integer> flux =  
          Flux  
                .range(1, 5)  
                .delayElements(Duration.ofMillis(300L))  
                .replay();  
  
    TimeUtils.sleep(500L);  
    flux.subscribe(data -> Logger.onNext("subscriber1: ", data));  
  
    TimeUtils.sleep(200L);  
    flux.subscribe(data -> Logger.onNext("subscriber2: ", data));  
  
    flux.connect();  
  
    TimeUtils.sleep(1000L);  
    flux.subscribe(data -> Logger.onNext("subscriber3: ", data));  
  
    TimeUtils.sleep(2000L);  
}
```
![](https://i.imgur.com/BdE7DSQ.png)

```java
public static void main(String[] args) {  
    ConnectableFlux<Integer> flux =  
          Flux  
                .range(1, 5)  
                .delayElements(Duration.ofMillis(300L))  
                .replay(2);  
  
    TimeUtils.sleep(500L);  
    flux.subscribe(data -> Logger.onNext("subscriber1: ", data));  
  
    TimeUtils.sleep(200L);  
    flux.subscribe(data -> Logger.onNext("subscriber2: ", data));  
  
    flux.connect();  
  
    TimeUtils.sleep(1000L);  
    flux.subscribe(data -> Logger.onNext("subscriber3: ", data));  
  
    TimeUtils.sleep(2000L);  
}
```
![](https://i.imgur.com/lvmh83s.png)
`replay()`에 전달한 파라미터 숫자 만큼의 가장 마지막에 emit한 데이터를 전달받는다