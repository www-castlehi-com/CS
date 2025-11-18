## 1️⃣ Filter
![](https://i.imgur.com/xJgtVuy.png)
Upstream에서 emit된 데이터 중에서 조건에 일치하는 데이터만 Downstream으로 emit한다
filter operator의 파라미터로 입력받은 Predicate의 return 값이 true인 데이터만 Downstream으로 emit한다
> **Predicate**
> 리턴값이 boolean인 자바 함수형 프로그래밍

#### 예시
```java
public static void main(String[] args) {
    Flux
        .range(1, 20)
        .filter(num -> num % 2 == 0)
        .subscribe(Logger::onNext);
}
```

---
```java
public static void main(String[] args) {
    Flux
        .fromIterable(SampleData.coronaVaccineNames)
        .filterWhen(vaccine -> isGreaterThan(vaccine, 3_000_000))
        .subscribe(Logger::onNext);

    TimeUtils.sleep(1000);
}
```
`filterWhen`은 비동기로 필터링하고 싶을 때 사용하며 반환값은 `Publisher<Predicate>`이다

## 2️⃣ Skip
![](https://i.imgur.com/gQF2UCs.png)
Upstream에서 emit된 데이터 중에서 파라미터로 입력받은 숫자만큼 건너뛴 후, 나머지 데이터를 Downstream으로 emit한다
![](https://i.imgur.com/fziY9aQ.png)
파라미터로 입력받은 시간 내에 emit된 데이터를 건너뛴 후, 나머지 데이터를 emit한다

#### 예시
```java
public static void main(String[] args) {
    Flux
        .interval(Duration.ofSeconds(1))
        .doOnNext(Logger::doOnNext)
        .skip(3)
        .subscribe(Logger::onNext);

    TimeUtils.sleep(5000L);
}
```
![](https://i.imgur.com/OLbp5KG.png)
`doOnNext()`를 통해 0~4의 데이터가 emit 되었지만, `doNext()`에서는 3개가 버려지고 3, 4만 처리했음을 알 수 있다

> `sleep(5000L)`의 이유는 `interval()`이 별도의 스레드에서 실행되는 operator이기 때문이다

---
```java
public static void main(String[] args) {
    Flux
        .interval(Duration.ofSeconds(1))
        .skip(Duration.ofMillis(2500))
        .subscribe(Logger::onNext);

    TimeUtils.sleep(5000L);
}
```
![](https://i.imgur.com/NWDavvW.png)
0, 1, 2, 3, 4는 각각 1초, 2초, 3초, 4초, 5초에 emit된다. 
`skip()`으로 인해 2.5초까지 버려지므로 1과 2사이까지 버려지고, 2부터 처리하는 것을 알 수 있다

## 3️⃣ Take
![](https://i.imgur.com/FC2ckPw.png)
Upstream에서 emit되는 데이터 중에서 파라미터로 입력받은 숫자만큼만, Downstream으로 emit한다
![](https://i.imgur.com/WD6iC5v.png)
Upstream에서 emit되는 데이터 중에서 파라미터로 입력받은 시간 내에 emit된 데이터만 Downstream으로 emit한다

#### 예시
```java
 public static void main(String[] args) {
    Flux
        .interval(Duration.ofSeconds(1))
        .doOnNext(Logger::doOnNext)
        .take(3)
        .subscribe(Logger::onNext);

    TimeUtils.sleep(5000L);
}
```
![](https://i.imgur.com/uHV0zSM.png)

---
```java
public static void main(String[] args) {
    Flux
        .interval(Duration.ofSeconds(1))
        .doOnNext(Logger::doOnNext)
        .take(Duration.ofSeconds(2))
        .subscribe(Logger::onNext);

    TimeUtils.sleep(4000L);
}
```
![](https://i.imgur.com/WqT95WU.png)

> `skip()`과 반대로 동작하지만 `doOnNext()`와 `onNext()`가 동일한 이유는 `take()`가 그 기점까지만 스트림을 열어 두기 때문이다
> 즉, 0~4가 emit되는 것이 아니라 `take()`에 따라서 0, 1까지만 스트림을 열어둔다

## 4️⃣ next
![](https://i.imgur.com/4Th5gQv.png)
Upstream에서 emit되는 데이터 중에서 첫 번째 데이터만 Downstream으로 emit한다
Upstream에서 emit되는 데이터가 empty라면 Downstream으로 empty Mono를 emit한다

#### 예시
```java
public static void main(String[] args) {
    Flux
        .fromIterable(SampleData.btcTopPricesPerYear)
        .doOnNext(Logger::doOnNext)
        .filter(tuple -> tuple.getT1() == 2015)
        .next()
        .subscribe(tuple -> Logger.onNext(tuple.getT1(), tuple.getT2()));
}
```
![](https://i.imgur.com/kT6dp0E.png)
`doOnNext()`를 통해서 여러 데이터가 emit되었지만, `next()`에 의해 첫 번째 데이터만 emit이 된다
그 이후에는 구독을 취소하기 때문에 남은 데이터는 emit 되지 않고 종료된다

> **`next()`와 `filter()`의 차이점**
> `filter()`에 의해 조건에 맞는 데이터가 1개 있을 경우, `next()`가 존재하지 않고 `filter()`만 있을 때에는 구독은 취소 되지 않는다

