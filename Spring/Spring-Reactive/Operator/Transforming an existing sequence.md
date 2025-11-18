## 1️⃣ map
![](https://i.imgur.com/KfNuOgm.png)
Upstream에서 emit된 데이터를 mapper Function을 사용하여 변환한 후, Downstream으로 emit한다
map Operator 내부에서 에러 발생 시, sequence가 종료되지 않고 계속 진행 되도록 하는 것이 가능하다

#### 예시
```java
public static void main(String[] args) {
    Flux
        .just("Green-Circle", "Yellow-Circle", "Blue-Circle")
        .map(circle -> circle.replace("Circle", "Rectangle"))
        .subscribe(Logger::onNext);
}
```
![](https://i.imgur.com/YWRviHB.png)

## 2️⃣ flatMap
![](https://i.imgur.com/IMpyFf2.png)
Upstream에서 emit된 데이터를 inner Publisher를 통해 변환한다
Upstream에서 들어오는 데이터가 하나이더라도, operator 내부에서 데이터가 여러 개가 생길 수 있다
비동기로 동작할 경우, emit되는 순서를 보장하지 않는다

> map이 1:1 매핑인 반면 flatMap은 1:n이다

#### 예시 코드
```java
public static void main(String[] args) {
    Flux
        .just(3)
        .flatMap(dan -> Flux.range(1, 9).map(n -> dan + " * " + n + " = " + dan * n))
        .subscribe(Logger::onNext);
}
```
외부 Publisher에서 단을 emit하고 `flatMap()`의 내부 Publisher에서 1부터 9까지 반복하여 구구단을 출력한다

---
```java
public static void main(String[] args) {
    Flux
        .range(2, 8)
        .flatMap(dan -> Flux
                            .range(1, 9)
                            .publishOn(Schedulers.parallel())
                            .map(n -> dan + " * " + n + " = " + dan * n))

        .subscribe(Logger::onNext);

    TimeUtils.sleep(200L);
}
```
![](https://i.imgur.com/G43qg1y.png)
내부 Publisher의 스레드가 다르므로 순서가 보장되지 않는다

> 비동기로 동작하는 이유는 내부 publisher의 순서가 다르기 때문만은 아니다
> `flatMap()`이 다수의 innerPublisher들을 구독하고 동시에 실행시키는 것이기 때문에 스레드가 달라서 비동기가 되는 것이다
>
> `flatMapSequential()`을 사용하면 순서대로 동작한다

## 3️⃣ concat
![](https://i.imgur.com/7PoxuHx.png)
파라미터로 입력되는 Publisher sequence들을 연결해서 데이터를 순차적으로 emit한다
**Subscribed to lazily** : 먼저 입력된 Publisher sequence의 emit이 종료될 때까지 나머지 Publisher Sequence들은 대기한다

#### 예시
```java
public static void main(String[] args) {
    Flux
        .concat(Flux.just(1, 2, 3), Flux.just(4, 5))
        .subscribe(Logger::onNext);
}
```
![](https://i.imgur.com/xKfxsyp.png)
## 4️⃣ merge
![](https://i.imgur.com/xYJf25A.png)
왼쪽부터 오른쪽으로  갈 수록 emit 되는 시간이 느리다
파라미터로 입력되는 Publisher sequence들에게서 emit된 데이터를 인터리빙 방식으로 병합한다

> **인터리빙**
> 비동기에서 여러 개의 작업이 번갈아가면서 실행이 된다

**Subscribed to eagerly** : 모든 Publisher sequence들을 즉시 구독한다

#### 예시
```java
public static void main(String[] args) {
    Flux
        .merge(
                Flux.just(1, 2, 3).delayElements(Duration.ofMillis(300L)),
                Flux.just(4, 5, 6).delayElements(Duration.ofMillis(500L))
        )
        .subscribe(Logger::onNext);

    TimeUtils.sleep(3500L);
}
```
![](https://i.imgur.com/p1c2o8a.png)
0.3초에 1
0.5초에 4
0.6초에 2
0.9초에 3
1.0초에 5
1.5초에 6이 emit된다

## 5️⃣ zip
![](https://i.imgur.com/OdKM9Lt.png)
파라미터로 입력되는 Publisher sequence들에게서 emit된 데이터를 결합한다
각 Publisher들이 데이터를 하나씩 emit할 때까지 기다렸다가 결합한다. 즉, 위 쪽에서 A, B가 emit되더라도 아래 쪽에서 1이 emit되어야만 emit될 수 있다
결합되지 않은 데이터는 emit 하지 않는다. 즉, e는 emit하지 않는다

#### 예시
```java
public static void main(String[] args) {
    Flux
            .zip(
                    Flux.just(1, 2, 3).delayElements(Duration.ofMillis(300L)),
                    Flux.just(4, 5, 6).delayElements(Duration.ofMillis(500L))
            )
            .subscribe(tuple2 -> Logger.onNext(tuple2));

    TimeUtils.sleep(2500L);
}
```
![](https://i.imgur.com/k5uGTCk.png)

---
```java
public static void main(String[] args) {
    Flux
            .zip(
                    Flux.just(1, 2, 3).delayElements(Duration.ofMillis(300L)),
                    Flux.just(4, 5, 6).delayElements(Duration.ofMillis(500L)),
                    (n1, n2) -> n1 * n2
            )
            .subscribe(Logger::onNext);

    TimeUtils.sleep(2500L);
}
```
`Zip()`의 세 번째 파라미터는 BiFunction을 이용한 두 Publisher에서 emit한 데이터의 처리값을 emit한다
Downstream 또한 tuple을 받는 것이 아니라 실제 값을 받는다
![](https://i.imgur.com/xoueqXn.png)

## 6️⃣ and
![](https://i.imgur.com/SFFARNG.png)
Mono의 Complete Signal과 파라미터로 입력된 Publisher의 Complete Signal을 결합하여 새로운 Mono\<Void\>로 변환한다
즉, Mono와 파라미터로 입력된 Publisher의 Sequence가 모두 종료 되었다는 사실만을 알릴 수 있다
하나의 Sequence만 전달 할 수 있다
선 작업이 모두 성공했을 때 다음 작업을 수행하고 싶을 때 사용한다

#### 예시
```java
public static void main(String[] args) {
    Mono
        .just("Okay")
        .delayElement(Duration.ofSeconds(1))
        .doOnNext(Logger::doOnNext)
        .and(
            Flux
                .just("Hi", "Tom")
                .delayElements(Duration.ofSeconds(2))
                .doOnNext(Logger::doOnNext)
        )
        .subscribe(
            Logger::onNext,
            Logger::onError,
            Logger::onComplete
        );

    TimeUtils.sleep(5000);
 }
```
![](https://i.imgur.com/Ta1zib8.png)

## 7️⃣ when
![](https://i.imgur.com/MpWnXO5.png)
파라미터로 입력된 Publisher들의 Complete Signal을 결합하여 새로운 Mono\<Void\>로 반환한다
즉, 파라미터로 입력된 Publisher들의 Sequence가 모두 종료 되었음을 알릴 수 있다
여러 개의 Sequence를 전달받을 수 있다

#### 예시
```java
public static void main(String[] args) {
    Mono
        .when(
            Flux
                .just("Hi", "Tom")
                .delayElements(Duration.ofSeconds(2))
                .doOnNext(Logger::doOnNext),
            Flux
                .just("Hello", "David")
                .delayElements(Duration.ofSeconds(1))
                .doOnNext(Logger::doOnNext)
        )
        .subscribe(
            Logger::onNext,
            Logger::onError,
            Logger::onComplete
        );

    TimeUtils.sleep(5000);
}
```
![](https://i.imgur.com/3eN8JQq.png)

## 8️⃣ then
![](https://i.imgur.com/nnYW6o4.png)
Mono의 complete signal과 error signal만 새로운 Mono\<Void\>로 반환한다
즉, Mono Sequence가 모두 종료되었음을 알릴 수 있다

#### 예시
```java
public static void main(String[] args) {
    restartApplicationServer()
        .then()
        .subscribe(
                Logger::onNext,
                Logger::onError,
                () -> Logger.onComplete("Send an email to Administrator: " +
                        "Application Server is restarted successfully")
        );

    TimeUtils.sleep(3000L);
}

private static Mono<Void> restartApplicationServer() {
    return Mono
            .just("Application Server was restarted successfully.")
            .delayElement(Duration.ofSeconds(2))
            .doOnNext(Logger::doOnNext)
            .flatMap(notUse -> Mono.empty());
}
```
![](https://i.imgur.com/jGCDhSj.png)

## 9️⃣ collectList
![](https://i.imgur.com/9bj9BQJ.png)
Flux에서 emit된 데이터를 모아서 List로 변환 후, 변환 된 List를 emit하는 Mono를 반환한다
만약 Upstream Sequence가 비어 있다면 비어 있는 list를 downstream으로 emit한다

#### 예시
```java
public static void main(String[] args) {  
    Flux  
          .just(1, 3, 5)  
          .map(number -> multiply(number))  
          .collectList()  
          .subscribe(list -> Logger.onNext(list));  
}  
  
public static Integer multiply(int number) {  
    return number * 2;  
}
```
![](https://i.imgur.com/kRwTNDx.png)

## 1️⃣0️⃣ collectMap
![](https://i.imgur.com/QK7yD9U.png)
Flux에서 emit된 데이터를 기반으로 key와 value를 생성하여 Map의 Element로 추가한 후, 최종적으로 Map을 emit하는 Mono를 반환한다
만약 Upstream Sequence가 비어 있다면 비어 있는 Map을 Downstream으로 emit한다

> 내부적으로 HashMap을 사용하기 때문에 순서가 보장되지 않는다

#### 예시
```java
public static void main(String[] args) {  
    Flux  
          .range(0, 26)  
          .collectMap(key -> key, value -> value * 2)  
          .subscribe(map -> Logger.onNext(map));  
}
```
![](https://i.imgur.com/y0zaBgi.png)
