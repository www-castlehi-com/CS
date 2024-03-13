# WebClient란?
- [[WebFlux]]에서 작동하는 HTTP Client
- 논블로킹
- 스트림 지원
- 콘텐츠 인코딩, 디코딩 코덱이 동일
> `implementation 'org.springframework.boot:spring-boot-starter-webflux'` -> Netty 자동 포함
# 특징
## 싱글 스레드
![](https://i.imgur.com/BRlax0A.png)

## Asynchronous Non-Blocking
![](https://i.imgur.com/6Gp5ZQY.png)

1. 각 요청은 event loop 내에 job으로 등록
2. job을 제공자에게 요청한 후 응답 결과를 기다리지 않고 다른 job 처리
3. callback을 통해 제공자에게 응답 결과가 오면 그 결과를 요청자에게 제공
# 메소드
## create 혹은 build
```java
static WebClient create() {
	return new DefaultWebClientBuilder().build();
}

public WebClient build() {
	ClientHttpConnector connectorToUse =
			(this.connector != null ? this.connector : initConnector());

	ExchangeFunction exchange = (this.exchangeFunction == null ?
			ExchangeFunctions.create(connectorToUse, initExchangeStrategies()) :
			this.exchangeFunction);

	ExchangeFilterFunction filterFunctions = (this.filters != null ? this.filters.stream()
			.reduce(ExchangeFilterFunction::andThen)
			.orElse(null) : null);

	HttpHeaders defaultHeaders = copyDefaultHeaders();

	MultiValueMap<String, String> defaultCookies = copyDefaultCookies();

	return new DefaultWebClient(exchange,
			filterFunctions,
			initUriBuilderFactory(),
			defaultHeaders,
			defaultCookies,
			this.defaultRequest,
			this.statusHandlers,
			this.observationRegistry,
			this.observationConvention,
			new DefaultWebClientBuilder(this));
}
```
1️⃣ **create**
- 내부적으로 `build()` 호출
2️⃣ **build**
- **ClientHttpConnector** : HTTP 요청을 실제로 네트워크를 통해 전송 (저수준)
- **ExchangeFunction** : `ClientHttpConnector`를 통해 전송된 HTTP 요청에 대한 응답을 처리 (고수준)
- **ExchangeFilterFunction** : 요청을 전송하기 전이나 받은 응답을 처리하기 전에 실행될 필터 목록
- **defaultCookies** : 기본 HTTP 헤더와 쿠키를 저장하며 모든 요청에 기본적으로 적용 -> 인증 토큰, 콘텐츠 타입 등
### 사용 예시
```java
public static void main(String[] args) { 
	WebClient webClientByCreate = WebClient.create("http://jsonplaceholder.typicode.com"); 

	WebClient webClientByBuilder = WebClient.builder()
				.build();
}
```
## Request 메소드
### 1️⃣ GET
#### Mono
```java
Mono<User> userMono = webClient.get() 
	.uri("/users/{id}", 1) 
	.retrieve() 
	.bodyToMono(User.class);
```
GET /users/{id}
Request : Single user by id
#### Flux
```java
Flux<User> usersFlux = webClient.get()
	.uri("/users") 
	.retrieve() 
	.bodyToFlux(User.class);
```
GET /users
Request : collection of employees
### 2️⃣ POST
#### Mono
```java
Mono<User> createdUser = webClient.post() 
	.uri("/users") 
	.body(Mono.just(new User("John Doe", "john@example.com")), User.class) 
	.retrieve() 
	.bodyToMono(User.class);
```
POST /users
Request : creates a new user from request body
Response : returns the created user
#### Flux
```java
Flux<User> createdUsers = webClient.post()
	.uri("/users/batch")
	.body(Flux.just(new User("Alice"), new User("Bob")), User.class)
	.retrive()
	.bodyToFlux(User.class);
```

> patch, delete, put의 경우 위와 유사
## Response 메소드
### 1️⃣ retrieve
#### return Entity
```java
Mono<ResponseEntity<User>> entityMono = client.get()
     .uri("/users/1")
     .retrieve()
     .toEntity(User.class);
```
#### return Mono, Flux
```java
Mono<Person> entityMono = client.get()
    .uri("/users/1")
    .retrieve()
    .bodyToMono(User.class);
```
### 2️⃣ exchangeToMono
```java
Mono<User> userMono = webClient.get() 
	.uri("/users/{id}", 1) 
	.exchangeToMono(response -> { if (response.statusCode().is2xxSuccessful()) { 
		return response.bodyToMono(User.class); 
	} else if (response.statusCode().is4xxClientError()) { 
		return Mono.error(new RuntimeException("API not found")); 
	} else { 
		return Mono.error(new RuntimeException("Unknown error"));
	} 
});
```
### 3️⃣ exchangeToFlux
```java
Flux<User> userFlux = webClient.get() 
	.uri("/users/{id}", 1) 
	.exchangeToFlux(response -> { if (response.statusCode().is2xxSuccessful()) { 
		return response.bodyToFlux(User.class); 
	} else if (response.statusCode().is4xxClientError()) { 
		return Mono.error(new RuntimeException("API not found")); 
	} else { 
		return Mono.error(new RuntimeException("Unknown error"));
	} 
});
```

> exchange는 deprecated
## 원리
### 1. HTTP method -> RequestBodyUriSpec
```java
Mono<String> createdUser = client.get()
	//...
```
WebClient.{HTTP method}
```java
@Override
public RequestHeadersUriSpec<?> get() {
	return methodInternal(HttpMethod.GET);
}

private RequestBodyUriSpec methodInternal(HttpMethod httpMethod) {
	return new DefaultRequestBodyUriSpec(httpMethod);
}
```
`DefaultWebClient`에서  `methodInternal`을 호출하며 해당 함수에서 httpMethod를 설정한 `RequestBodyUriSpec` 반환
### 2. body -> Mono\<T>
요청 응답에 body가 있는 경우
```java
Mono<String> createdUser = client.post() 
			.uri("/users") 
			.body(Mono.just(new String("John Doe")), String.class)
			//...
```

```java
@Override
public <T, P extends Publisher<T>> RequestHeadersSpec<?> body(P publisher, Class<T> elementClass) {
	this.inserter = BodyInserters.fromPublisher(publisher, elementClass);
	return this;
}
```
- **publisher** : 본문 데이터를 비동기적으로 제공
- **elementClass** : 제공 받은 데이터 타입
- **BodyInserter** : HTTP 요청 실행 시 실제 본문 데이터를 HTTP 메시지로 씀
```java
public static <T, P extends Publisher<T>> BodyInserter<P, ReactiveHttpOutputMessage> fromPublisher(
			P publisher, Class<T> elementClass) {

	Assert.notNull(publisher, "'publisher' must not be null");
	Assert.notNull(elementClass, "'elementClass' must not be null");
	return (message, context) ->
			writeWithMessageWriters(message, context, publisher, ResolvableType.forClass(elementClass), null);
	}
```
- `BodyInserter`의 정적 팩토리 메소드
- `BodyInserter` 생성
- `writeWithMessageWriters` 호출하여 실제 쓰기 작업 수행할 람다 생성 -> `BodyInserter`에 의해 사용
```java
private static <M extends ReactiveHttpOutputMessage> Mono<Void> writeWithMessageWriters(
			M outputMessage, BodyInserter.Context context, Object body, ResolvableType bodyType, @Nullable ReactiveAdapter adapter) {

	Publisher<?> publisher;
	if (body instanceof Publisher<?> publisherBody) {
		publisher = publisherBody;
	}
	else if (adapter != null) {
		publisher = adapter.toPublisher(body);
	}
	else {
		publisher = Mono.just(body);
	}
	MediaType mediaType = outputMessage.getHeaders().getContentType();
	for (HttpMessageWriter<?> messageWriter : context.messageWriters()) {
		if (messageWriter.canWrite(bodyType, mediaType)) {
			HttpMessageWriter<Object> typedMessageWriter = cast(messageWriter);
			return write(publisher, bodyType, mediaType, outputMessage, context, typedMessageWriter);
		}
	}
	return Mono.error(unsupportedError(bodyType, context, mediaType));
}
```
- **outputMessage** : HTTP 요청이나 응답 메시지
- **context** : 쓰기 작업을 수행하는데 필요한 컨텍스트 정보
- `publisher`로부터 데이터를 읽고, `context`에 맞는 `HttpMessageWriter` 목록을 사용해 데이터를 HTTP 메시지로 변환 -> `outputMessage`에 작성
```java
private static <T> Mono<Void> write(Publisher<? extends T> input, ResolvableType type,
			@Nullable MediaType mediaType, ReactiveHttpOutputMessage message,
			BodyInserter.Context context, HttpMessageWriter<T> writer) {

	return context.serverRequest()
			.map(request -> {
				ServerHttpResponse response = (ServerHttpResponse) message;
				return writer.write(input, type, type, mediaType, request, response, context.hints());
			})
			.orElseGet(() -> writer.write(input, type, mediaType, message, context.hints()));
}
```
- `ReactiveHttpOutputMessage`의 구현체인 `ServerHttpResponse`에 담아 해당 구현체의 `write` 호출
```java
// 구현체 중 하나인 EncoderHttpMessageWriter

@Override
public Mono<Void> write(Publisher<? extends T> inputStream, ResolvableType elementType,
		@Nullable MediaType mediaType, ReactiveHttpOutputMessage message, Map<String, Object> hints) {

	MediaType contentType = updateContentType(message, mediaType);

	Flux<DataBuffer> body = this.encoder.encode(
			inputStream, message.bufferFactory(), elementType, contentType, hints);

	if (inputStream instanceof Mono) {
		return body
				.singleOrEmpty()
				.switchIfEmpty(Mono.defer(() -> {
					message.getHeaders().setContentLength(0);
					return message.setComplete().then(Mono.empty());
				}))
				.flatMap(buffer -> {
					Hints.touchDataBuffer(buffer, hints, logger);
					message.getHeaders().setContentLength(buffer.readableByteCount());
					return message.writeWith(Mono.just(buffer)
							.doOnDiscard(DataBuffer.class, DataBufferUtils::release));
				})
				.doOnDiscard(DataBuffer.class, DataBufferUtils::release);
	}

	if (isStreamingMediaType(contentType)) {
		return message.writeAndFlushWith(body.map(buffer -> {
			Hints.touchDataBuffer(buffer, hints, logger);
			return Mono.just(buffer).doOnDiscard(DataBuffer.class, DataBufferUtils::release);
		}));
	}

	if (logger.isDebugEnabled()) {
		body = body.doOnNext(buffer -> Hints.touchDataBuffer(buffer, hints, logger));
	}
	return message.writeWith(body);
}
```
- Mono, Flux로 나누어 처리하며 Mono일 경우 단일 버퍼에 담음
- `return message.writeWith(body)`를 통해 데이터 버퍼 body를 응답 메시지에 작성
### 3. retrieve -> ResponseSpec
응답값을 추출
```java
Mono<ResponseEntity<Person>> entityMono = client.get()
      .uri("/persons/1")
      .accept(MediaType.APPLICATION_JSON)
      .retrieve()
      .toEntity(Person.class);

Mono<Person> entityMono = client.get()
      .uri("/persons/1")
      .accept(MediaType.APPLICATION_JSON)
      .retrieve()
      .bodyToMono(Person.class);

Mono<Person> entityMono = client.get()  
       .uri("/person/1")  
       .accept(MediaType.APPLICATION_JSON)  
       .retrieve()  
       .onStatus(httpStatusCode -> httpStatusCode.is4xxClientError(), response -> {  
          return Mono.error(new IllegalAccessException("4xx client error occurred"));  
       })  
       .bodyToMono(Person.class);
```
- **toEntity** : status를 포함한 header, body를 가지고 있는 `ResponseEntity`를 추출
- **bodyToMono, bodyToFlux** : body만 추출할 경우
-  **onStatus** : status code로 분기를 설정할 경우
```java
@Override
public ResponseSpec retrieve() {
	return new DefaultResponseSpec(
			this.httpMethod, initUri(), exchange(), DefaultWebClient.this.defaultStatusHandlers);
}
```

```java
DefaultResponseSpec(HttpMethod httpMethod, URI uri, Mono<ClientResponse> responseMono,
			List<StatusHandler> defaultStatusHandlers) {

	this.httpMethod = httpMethod;
	this.uri = uri;
	this.responseMono = responseMono;
	this.statusHandlers.addAll(defaultStatusHandlers);
	this.statusHandlers.add(DEFAULT_STATUS_HANDLER);
	this.defaultStatusHandlerCount = this.statusHandlers.size();
}
```
`retrieve` 메소드는 `ResponseSpec` 인터페이스의 구현체인 `DefaultResponseSpec` 객체 반환
-> `bodyToMono`, `bodyToFlux`와 같은 응답 처리에 필요한 메서드 제공

```java
@SuppressWarnings("deprecation")
@Override
public Mono<ClientResponse> exchange() {
	ClientRequest.Builder requestBuilder = initRequestBuilder();
	ClientRequestObservationContext observationContext = new ClientRequestObservationContext(requestBuilder);
	return Mono.deferContextual(contextView -> {
		Observation observation = ClientHttpObservationDocumentation.HTTP_REACTIVE_CLIENT_EXCHANGES.observation(observationConvention,
				DEFAULT_OBSERVATION_CONVENTION, () -> observationContext, observationRegistry);
		observation
				.parentObservation(contextView.getOrDefault(ObservationThreadLocalAccessor.KEY, null))
				.start();
		ExchangeFilterFunction filterFunction = new ObservationFilterFunction(observationContext);
		if (filterFunctions != null) {
			filterFunction = filterFunctions.andThen(filterFunction);
		}
		ClientRequest request = requestBuilder
				.attribute(ClientRequestObservationContext.CURRENT_OBSERVATION_CONTEXT_ATTRIBUTE, observationContext)
				.build();
		observationContext.setUriTemplate((String) request.attribute(URI_TEMPLATE_ATTRIBUTE).orElse(null));
		observationContext.setRequest(request);
		Mono<ClientResponse> responseMono = filterFunction.apply(exchangeFunction)
				.exchange(request)
				.checkpoint("Request to " +
						WebClientUtils.getRequestDescription(request.method(), request.url()) +
						" [DefaultWebClient]")
				.switchIfEmpty(NO_HTTP_CLIENT_RESPONSE_ERROR);
		if (this.contextModifier != null) {
			responseMono = responseMono.contextWrite(this.contextModifier);
		}
		final AtomicBoolean responseReceived = new AtomicBoolean();
		return responseMono
				.doOnNext(response -> responseReceived.set(true))
				.doOnError(observationContext::setError)
				.doFinally(signalType -> {
					if (signalType == SignalType.CANCEL && !responseReceived.get()) {
						observationContext.setAborted(true);
					}
					observation.stop();
				})
				.contextWrite(context -> context.put(ObservationThreadLocalAccessor.KEY, observation));
	});
}
```
- 실제 HTTP 요청 실행
```java
private ClientRequest.Builder initRequestBuilder() {
	if (defaultRequest != null) {
		defaultRequest.accept(this);
	}
	ClientRequest.Builder builder = ClientRequest.create(this.httpMethod, initUri())
			.headers(this::initHeaders)
			.cookies(this::initCookies)
			.attributes(attributes -> attributes.putAll(this.attributes));
	if (this.httpRequestConsumer != null) {
		builder.httpRequest(this.httpRequestConsumer);
	}
	if (this.inserter != null) {
		builder.body(this.inserter);
	}
	return builder;
}
```
HTTP 요청에 대한 정보를 객체에 저장
- HTTP 메소드, URI 설정
- 헤더, 쿠키 초기화
- 속성 설정
- body 설정
```java
public ClientRequestObservationContext(ClientRequest.Builder request) {
	super(ClientRequestObservationContext::setRequestHeader);
	setCarrier(request);
	setRequest(request.build());
}
```
요청 실행 과정에서 발생하는 이벤트 추적, 관련 정보 저장
```java
Mono.deferContextual(contextView ->
```
리액터가 리액티브 컨텍스트에 접근
**Context** : 어떠한 상황에서 그 상황을 처리하기 위해 필요한 정보
> 예시
> - ServletContext : Servlet이 Setvlet Container와 통신하기 위해 필요한 정보를 제공하는 인터페이스
> - ApplicationContext : 애플리케이션의 정보를 제공하는 인터페이스
> - SecurityContext : 애플리케이션 사용자의 인증 정보를 제공하는 인터페이스

ReactiveContext 
- Reactor 간의 구성 요소 간에 전파되는 key/value 형태의 저장소
- Subscriber와 매핑되어 구독이 발생할 때마다 해당 구독과 연결된 하나의 Context가 생성
- **전파** : Downstream -> Upstream으로 컨텍스트가 전달되며 모든 Operator가 컨텍스트의 정보를 동일하게 이용할 수 있음
	- Downstream : 현재 작업으로부터 이후에 발생하는 작업들
	- Upstream : 현재 작업 이전에 발생하는 작업
```java
Observation observation = ClientHttpObservationDocumentation.HTTP_REACTIVE_CLIENT_EXCHANGES.observation(observationConvention,
		DEFAULT_OBSERVATION_CONVENTION, () -> observationContext, observationRegistry);
observation
		.parentObservation(contextView.getOrDefault(ObservationThreadLocalAccessor.KEY, null))
		.start();
```
reative context가 `Observation` 인스턴스를 시작하여 HTTP 요청 및 응답에 대한 세부 정보 수집하고 모니터링함
부모 관찰 시작 -> 현재 요청이 독립적인 요청인지, 상위 요청에 의해 발생되었는지 확인
```java
ExchangeFilterFunction filterFunction = new ObservationFilterFunction(observationContext);  
if (filterFunctions != null) {  
    filterFunction = filterFunctions.andThen(filterFunction);  
}  

ClientRequest request = requestBuilder  
       .attribute(ClientRequestObservationContext.CURRENT_OBSERVATION_CONTEXT_ATTRIBUTE, observationContext)  
       .build();

observationContext.setUriTemplate((String) request.attribute(URI_TEMPLATE_ATTRIBUTE).orElse(null));  
observationContext.setRequest(request);
```
filter, clientRequest에 관찰 컨텍스트를 저장
관찰 컨텍스트에 요청 url과 clientRequest 객체 정보 저장
```java
Mono<ClientResponse> responseMono = filterFunction.apply(exchangeFunction)
	.exchange(request) 
	.checkpoint("Request to " + WebClientUtils.getRequestDescription(request.method(), request.url()) + " [DefaultWebClient]")
	.switchIfEmpty(NO_HTTP_CLIENT_RESPONSE_ERROR);
```
비동기 요청 처리 과정에서, 결과로 받은 응답을 처리함
- **apply** : 필터 적용 -> 요청이나 응답 수정 혹은 추가적인 사이드 이펙트를 발생시킬 수 있는 로직 실행됨
- **exchange** : `clientRequset` 객체를 매개변수로 전달하고 HTTP 요청을 실행하여 비동기적으로 응답 처리 후, 응답을 리엑터 형태로 반환
- **switchIfEmpty** : 에러로 인해 응답을 받지 못해 Mono가 비어있을 때 에러 반환
	```java
	private static final Mono<ClientResponse> NO_HTTP_CLIENT_RESPONSE_ERROR = Mono.error(  
       () -> new IllegalStateException("The underlying HTTP client completed without emitting a response."));
	```

```java
final AtomicBoolean responseReceived = new AtomicBoolean();  
return responseMono  
       .doOnNext(response -> responseReceived.set(true))  
       .doOnError(observationContext::setError)  
       .doFinally(signalType -> {  
          if (signalType == SignalType.CANCEL && !responseReceived.get()) {  
             observationContext.setAborted(true);  
          }  
          observation.stop();  
       })  
       .contextWrite(context -> context.put(ObservationThreadLocalAccessor.KEY, observation));
```
- **responseReceived** : 응답 수신 여부를 조사하면 초기값은 false
- **doOnNext** : 스트림에서 응답이 방출될 때마다 지정된 작업 수행
	>응답이 성공적으로 수신되면 `responseReceived`를 true로 설정
- **doOnError** : 스트림에서 오류가 발생했을 때 지정된 작업 수행
	> 오류 발생 시 관찰 컨텍스트에 오류 정보 설정
- **doFinally** : 스트림이 완료되면 실행
	> 응답이 취소되고, 도착하지 않았다면 관찰 컨텍스트의 상태를 aborted로 설정
	> 어떤 경우든 관찰 중단
- **contextWrite** : 스트림의 컨텍스트에 관찰 데이터를 포함시켜 다운스트림 시 관찰 컨텍스트에 접근할 수 있도록 함
### 4. exchangeToXXX
```java
Mono<Person> entityMono = client.get()
	.uri("/persons/1")
	.accept(MediaType.APPLICATION_JSON)
	.exchangeToMono(response -> {
	  if (response.statusCode().equals(HttpStatus.OK)) {
		  return response.bodyToMono(Person.class);
	  }
	  else {
		  return response.createError();
	  }
	});

Flux<Person> entityFlux = client.get()
	.uri("/persons")
	.accept(MediaType.APPLICATION_JSON)
	.exchangeToFlux(response -> {
	  if (response.statusCode().equals(HttpStatus.OK)) {
		  return response.bodyToFlux(Person.class);
	  }
	  else {
		  return response.createError().flux();
	  }
	});
```
`clientResponse`에 대한 접근을 가능하게 하며, 응답 상태에 따라 응답을 다르게 디코딩할 수 있음

1) `exchangeToMono`
	```java
	@Override  
	public <V> Mono<V> exchangeToMono(Function<ClientResponse, ? extends Mono<V>> responseHandler) {  
	    return exchange().flatMap(response -> {  
	       try {  
	          return responseHandler.apply(response)  
	                .flatMap(value -> releaseIfNotConsumed(response).thenReturn(value))  
	                .switchIfEmpty(Mono.defer(() -> releaseIfNotConsumed(response).then(Mono.empty())))  
	                .onErrorResume(ex -> releaseIfNotConsumed(response, ex));  
	       }  
	       catch (Throwable ex) {  
	          return releaseIfNotConsumed(response, ex);  
	       }  
	    });  
	}
	```
	- **flatMap(~ releaseIfNotConsumed)** : response가 읽혔다면 연결 리소스를 해제하고, `responseHandler`에서 반환된 결과 값을 그대로 반환
	- **switchIfEmpty** : `responseHandler`가 빈 Mono를 반환하는 경우, 연결 리소스를 해제하고 빈 Mono를 반환
	- **onErrorResume** : 처리 과정에서 오류 발생 시 연결 리소스 해제 후 해당 오류 처리
2) `exchangeToFlux`
	```java
	@Override  
	public <V> Flux<V> exchangeToFlux(Function<ClientResponse, ? extends Flux<V>> responseHandler) {  
	    return exchange().flatMapMany(response -> {  
	       try {  
	          return responseHandler.apply(response)  
	                .concatWith(Flux.defer(() -> releaseIfNotConsumed(response).then(Mono.empty())))  
	                .onErrorResume(ex -> releaseIfNotConsumed(response, ex));  
	       }  
	       catch (Throwable ex) {  
	          return releaseIfNotConsumed(response, ex);  
	       }  
	    });  
	}
	```

>Mono와 Flux의 차이점은 `flatMap`, `flatMapMany`에 있음
## 정리
```
WebClient.builder() -> WebClient 생성
  |
  v
HTTP 메서드 선택 (.get(), .post(), ...) -> RequestHeadersUriSpec
  |
  v
URI 설정 (.uri()) -> RequestHeadersSpec
  |
  v
헤더/본문 설정 (.header(), .body(), ...) -> RequestHeadersSpec
  |
  v
요청 실행 (.retrieve() / .exchange()) -> ResponseSpec / ClientResponse
  |
  v
응답 처리 (.bodyToMono(), .bodyToFlux(), .onStatus(), .onErrorMap(), ...)
```
