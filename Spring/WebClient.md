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
