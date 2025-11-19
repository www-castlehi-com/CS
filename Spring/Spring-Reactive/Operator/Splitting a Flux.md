## 1️⃣ window
![](https://i.imgur.com/cECsUhk.png)
Upstream에서 emit되는 첫번째 데이터부터 maxSize 숫자만큼의 데이터를 포함하는 새로운 Flux(윈도우)로 분할한다
데이터 개수 = Downstream에서 요청한 개수 (`request()`) * maxSize

#### 예시
```java
public static void main(String[] args) {  
    Flux  
          .range(1, 11)  
          .window(3)  
          .flatMap(flux -> {  
             Logger.info("======================");  
             return flux;  
          })  
          .subscribe(new BaseSubscriber<>() {  
             @Override  
             protected void hookOnSubscribe(Subscription subscription) {  
                subscription.request(2);  
             }  
  
             @Override  
             protected void hookOnNext(Integer value) {  
                Logger.onNext(value);  
                request(2);  
             }  
          });  
}
```
![](https://i.imgur.com/6EkeGDB.png)

## 2️⃣ buffer
![](https://i.imgur.com/wZrpgYk.png)
Upstream에서 emit되는 데이터가 maxSize 숫자만큼 버퍼에 채워지면 버퍼를 비운다
버퍼에서 비워진 데이터는 List 형태로 한번에 Downstream으로 emit된다
버퍼 크기가 너무 크면 스트림이 끝날 때까지 무한정 대기할 수 있다

#### 예시
```java
public static void main(String[] args) {  
    Flux  
          .range(1, 95)  
          .buffer(10)  
          .subscribe(buffer -> Logger.onNext(buffer));  
}
```
![](https://i.imgur.com/49lFF88.png)

## 3️⃣ bufferTimeout
![](https://i.imgur.com/IB0vmOR.png)
Upstream에서 emit된 데이터가 버퍼에 채워질 때 maxTime에 도달하면 버퍼를 비운다
maxTime에 도달하기 전에 maxSize 만큼의 데이터가 버퍼에 채워지면 maxTime까지 기다리지 않고 버퍼를 비운다
#### 예시
```java
public static void main(String[] args) {  
    Flux  
          .range(1, 20)  
          .map(num -> {  
             try {  
                if (num < 10) {  
                   Thread.sleep(100L);  
                } else {  
                   Thread.sleep(300L);  
                }  
             } catch (InterruptedException e) {}  
             return num;  
          })  
          .bufferTimeout(3, Duration.ofMillis(400L))  
          .subscribe(buffer -> Logger.onNext(buffer));  
}
```
![](https://i.imgur.com/YYiYYCi.png)
## 4️⃣ groupBy
![](https://i.imgur.com/XFWDGaE.png)
`groupBy(keyMapper)`는 emit되는 데이터를 keyMapper로 생성한 key를 기준으로 그룹화 한 GroupedFlux를 리턴한다
![](https://i.imgur.com/m5NetRM.png)
`groupBy(keyMapper, valueMapper)`는 emit되는 데이터를 keyMapper로 생성한 key를 기준으로 그룹화한 후에 valueMapper를 통해 그룹화된 데이터를 다른 형태로 가공 처리한 후, Downstream으로 emit한다

#### 예시
```java
public static void main(String[] args) {
    Flux
        .fromIterable(SampleData.books)
        .groupBy(book -> book.getAuthorName())
        .flatMap(groupedFlux ->
                groupedFlux
                        .map(book -> book.getBookName() +
                                "(" + book.getAuthorName() + ")")
                        .collectList()
        )
        .subscribe(bookByAuthor ->
                Logger.onNext(bookByAuthor));
}
```

```java
public static void main(String[] args) {
    Flux
        .fromIterable(SampleData.books)
        .groupBy(book -> book.getAuthorName(),
                book -> book.getBookName() + "(" + book.getAuthorName() + ")")
        .flatMap(groupedFlux -> groupedFlux.collectList())
        .subscribe(booksByAuthor ->
                Logger.onNext(booksByAuthor));
}
```
두 함수의 결과는 동일하다