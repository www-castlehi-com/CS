일반적으로 리액터에서는 throw를 사용해서 에러를 처리하는 방법을 사용하지 않는다
예외가 발생하면 Subscriber에서 `onError` 시그널을 통해 전달받을 수 있다
Subscriber가 받기 전에 업스트림 쪽에서 에러를 적절히 처리할 수 있다
예외가 발생해도 시퀀스 자체를 종료하지 않고 남은 시퀀스를 진행하게 할 수 있다
## 1️⃣ error
![](https://i.imgur.com/QJMrMOP.png)
파라미터로 지정된 에러로 종료하는 Mono 또는 Flux를 생성한다

>Java의 throw 키워드를 사용해 예외를 의도적으로 throw하는 것과 유사하다

#### 예시
```java
public static void main(String[] args) {  
    Flux  
          .just('a', 'b', 'c', '3', 'd')  
          .flatMap(letter -> {  
             return convert(letter);  
          })  
          .subscribe(Logger::onNext,  
                Logger::onError);  
}  
  
private static Mono<String> convert(char ch) {  
    if (!Character.isAlphabetic(ch)) {  
       return Mono.error(new DataFormatException("Not Alphabetic"));  
    }  
    return Mono.just("Converted to " + Character.toUpperCase(ch));  
}
```
`error()`를 통해 Exception 객체를 전달한다
![](https://i.imgur.com/7g4uxLp.png)
## 2️⃣ onErrorReturn
![](https://i.imgur.com/cbhWMoB.png)
에러가 발생했을 때, onError signal을 Downstream으로 전파하지 않고, 대체 값을 emit한다

> Java에서 try ~  catch 문의 catch 블록에서 발생한 예외 대신에 대체 값을 리턴 하는 방식과 유사

#### 예시
```java
public static void main(String[] args) {
    getBooks()
            .map(book -> book.getPenName().toUpperCase())
            .onErrorReturn(NullPointerException.class, "no pen name")
            .onErrorReturn(IllegalFormatException.class, "Illegal pen name")
            .subscribe(Logger::info, Logger::onError);
}

public static Flux<Book> getBooks() {
    return Flux.fromIterable(SampleData.books);
}
```
여러 `onErrorReturn` 시그널을 사용할 수 있지만 예외가 발생하면 operator를 더 이상 수행하지 않기 때문에 먼저 발생한 exception에 대한 대체값만 리턴한다
![](https://i.imgur.com/r24b3KX.png)

## 3️⃣ onErrorResume
![](https://i.imgur.com/MuSMBwE.png)
에러가 발생했을 때, onError signal을 Downstream으로 전파하지 않고 대체 Publisher를 리턴한다
즉, 에러로 인해 중단된 Sequence를 다시 재개한다

#### 예시
```java
public static void main(String[] args) {
    final String keyword = "DDD";
    getBooksFromCache(keyword)
            .onErrorResume(error -> getBooksFromDatabase(keyword))
            .subscribe(data -> Logger.onNext(data.getBookName()),
                    Logger::onError);
}
```
cache에 없으면 예외가 발생하는데, `onErrorResume`을 통해서 데이터베이스에서 이어 찾는 것으로 설계할 수 있다

## 4️⃣ onErrorContinue
![](https://i.imgur.com/r6sVckZ.png)
에러가 발생했을 때 에러가 발생한 데이터는 제거하고 후속 데이터를 계속 emit한다
데이터의 흐름이 일반적인 흐름과 반대로 동작하기 때문에 의도하지 않은 동작을 일으킬 수 있어 권장 방식은 아니다

#### 예시
```java
public static void main(String[] args) {  
    Flux  
          .just(1, 2, 4, 0, 6, 12)  
          .map(num -> 12 / num)  
          .onErrorContinue((error, num) -> Logger.onError("error: {}, num: {}", error, num))  
          .subscribe(Logger::onNext,  
                Logger::onError);  
}
```
![](https://i.imgur.com/R4gUMsx.png)

## 5️⃣ onErrorMap
![](https://i.imgur.com/NnoWgmY.png)
첫번째 `onErrorMap`은 에러가 발생하면 타입과 상관없이 다른 타입의 에러로 변환해서 Downstream으로 전달한다

![](https://i.imgur.com/pjZbtJE.png)
에러가 발생했을 때, 파라미터로 지정한 에러 타입과 일치하면 다른 타입의 에러로 변환해서 Downstream 으로 전달한다
예외 타입이 일치하지 않으면 원래 발생한 에러를 Downstream으로 전달한다

#### 예시
```java
public static void main(String[] args) {  
    Flux.just(1, 3, 0, 6, 8)  
          .filter(num -> num % 3 == 0) // 3으로 나누어 떨어지는 숫자만 필터링 하기 위한 작업. 0도 포함된다.  
          .doOnNext(Logger::doOnNext)  
          .map(num -> (num * 2) / num) // 0으로 나눌 수 없으므로 ArithmeticException이 발생한다.  
          .onErrorMap(error -> new CannotDivideByZeroException(error.getMessage()))  
          .subscribe(  
                Logger::onNext,  
                Logger::onError  
          );  
  
}
```
![](https://i.imgur.com/iFiYPue.png)

```java
public static void main(String[] args) {  
    RestTemplate restTemplate = new RestTemplate();  
    HttpHeaders headers = new HttpHeaders();  
    headers.setAccept(Collections.singletonList(MediaType.APPLICATION_JSON));  
  
    Mono.fromSupplier(() ->  
                restTemplate.exchange(WORLD_TIME_URI, HttpMethod.GET, new HttpEntity<String>(headers), String.class)  
          )  
          .onErrorMap(HttpClientErrorException.class, (HttpClientErrorException ex) -> {  
             if (ex.getStatusCode() == HttpStatus.NOT_FOUND) {  
                return new TimezoneNotFoundException(ex.getResponseBodyAsString());  
             }  
             return new HttpClientErrorException(ex.getStatusCode());  
          })  
          .map(response -> {  
             DocumentContext jsonContext = JsonPath.parse(response.getBody());  
             String dateTime = jsonContext.read("$.datetime");  
             return dateTime;  
          })  
          .subscribe(Logger::onNext,  
                Logger::onError,  
                Logger::onComplete);  
}
```
`onErrorMap()`에서 status에 따라 에러를 다른 에러로 변환한다

## 6️⃣ retry
![](https://i.imgur.com/JbgWwtG.png)
데이터 emit 중 에러가 발생하면 파라미터로 입력한 횟수만큼 원본 Publisher의 Sequence를 다시 구독한다
즉, 원본 Publisher의 데이터를 처음부터 다시 emit한다

#### 예시
```java
public static void main(String[] args) throws InterruptedException {  
    final int[] count = {1};  
    Flux  
          .range(1, 3)  
          .delayElements(Duration.ofSeconds(1))  
          .map(num -> {  
             if (num == 3 && count[0] == 1) {  
                count[0]++;  
                TimeUtils.sleep(1000);  
             }  
  
             return num;  
          })  
          .timeout(Duration.ofMillis(1500))  
          .retry(1)  
          .subscribe(Logger::onNext,  
                Logger::onError,  
                Logger::onComplete);  
  
    TimeUtils.sleep(7000);  
}
```
![](https://i.imgur.com/GNaUYIO.png)
