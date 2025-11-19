## 정의
Spring 5부터 지원하는 리액티브 웹 프레임워크이다
비동기 Non-Blocking I/O 방식으로 적은 수의 스레드를 사용한다
Reactive Streams의 구현체 중에 하나인 Reactor에 의존하여 비동기 로직을 구성하고 리액티브 스트림을 제공한다
Reactor 기반이지만 RxJava 등 다른 리액티브 확장 라이브러리를 쉽게 적용할 수 있다
## 기술 스택
![](https://velog.velcdn.com/images/www-castlehi-com/post/43097908-c6f0-4d64-b936-8f15d3b1c736/image.png)
- Reactive Stack과 Servlet Stack이 존재한다
  - Servlet 위에서도 Reactive Streams를 사용할 수 있으나 fully non-blocking I/O 방식을 구현할 순 없다
- Webflux는 Netty 등의 서버 엔진에서 동작한다
- 리액티브 스트림즈 어댑터를 통해 API를 지원한다
- Spring MVC에서는 Spring Security가 서블릿 컨테이너와 통합되는 반면에 Spring Webflux는 WebFilter를 이용한다
- Spring Webflux는 RDBMS를 지원하지 않는다
![](https://velog.velcdn.com/images/www-castlehi-com/post/ab0d33d6-c038-4675-ad6d-f9053ae3395f/image.png)
## 요청 처리 흐름
아래 요청 처리 흐름은 Spring MVC를 사용하지 않는 완전한 Webflux 환경일 때에만 해당한다
만약 Spring MVC를 사용한다면 Spring MVC의 흐름을 타게 된다
다만, Mono나 Flux를 반환하는 컨트롤러의 경우 `HttpMessageConverter`가 이들을 POJO로 보지 않기 때문에 JSON 변환을 시도할 때 그대로 serialize한다
![](https://velog.velcdn.com/images/www-castlehi-com/post/f0ba477a-efae-4031-b706-ab66ec0888c2/image.png)
1. 클라이언트 요청이 들어오면 Netty 등의 서버 엔진을 거쳐 HttpHandler가 들어오는 요청을 전달 받는다. `ServerHttpRequest`와 `ServerHttpResponse`를 포함하는 `ServerWebExchange`를 생성하고, `WebFilter` 체인으로 전달한다
2. `WebFilter` 체인에서 `ServerWebExchange`를 전처리한다. `WebHandler` 인터페이스의 구현체인 `DispatcherHandler`에게 전달된다.
3. `DispatcherHandler`는 `HandlerMapping` List를 원본 Flux의 소스로 전달받는다.
4. `ServerWebExchange`를 처리할 핸들러를 조회한다.
5. 조회할 핸들러의 호출을 `HandlerAdapter`에게 위임한다.
6. `HandlerAdapter`는 `ServerWebExchange`를 처리할 핸들러를 호출한다.
7. `Controller`와 같은 핸들러에서 요청을 처리한 후, 응답 데이터를 리턴한다.
8. 핸들러로부터 리턴받은 응답 데이터를 처리할 `HandlerResultHandler`를 조회한다.
9. 조회한 `HandlerResultHandler`가 응답 데이터를 적절하게 처리하여 response로 리턴한다.
> 실제 핸들러에서 리턴되는 것은 응답 데이터를 포함하고 있는 Flux또는 Mono sequence이기 때문에 메서드 호출을 통해 리턴된 Reactor Sequence가 즉시 어떤 작업을 수행한다는 의미는 아니다

## 프로세스
![](https://velog.velcdn.com/images/www-castlehi-com/post/cb258069-1a53-4c64-b87c-24f3036f8ae6/image.png)
> **스레드** 
CPU 코어를 효율적으로 사용하기 위해 논리적으로 쪼갠 것

1. 요청이 들어오면 요청 핸들러는 이벤트 루프에 요청 이벤트를 푸시한다
2. 이벤트 루프는 푸시된 요청 이벤트에 대해 콜백을 등록한다
3. 작업이 완료가 되면 완료 이벤트를 이벤트 루프에 푸시한다
4. 콜백을 호출하여 처리 결과를 전달한다

CPU 코어 개수만큼의 스레드가 있으면 하나의 스레드로 대량의 요청을 처리할 수 있다
따라서 적은 수의 고정된 크기의 스레드를 생성한다
CPU 코어의 개수가 4보다 더 적은 경우 최소 4개의 워커 스레드를 생성하고, 4보다 더 많다면 코어 개수만큼 스레드를 생성한다

## WebClient
Non-Blocking HTTP request를 위한 리액티브 웹 클라이언트이다
함수형 기반의 향상된 API를 제공한다

내부적으로 HTTP 클라이언트 라이브러리에게 HTTP request를 위임한다
기본 HTTP 클라이언트 라이브러리는 Reactor Netty이다

> Spring MVC 기반에서의 RestTemplate이라고 생각하면 된다

## 적합한 시스템
- Blocking I/O 방식으로 처리하는데 한계가 있는 대량의 요청 트래픽이 발생하는 시스템
- 마이크로 서비스 기반 시스템
- 스트리밍 시스템 또는 실시간 시스템
- 네트워크 접속이 느린 클라이언트의 요청 처리