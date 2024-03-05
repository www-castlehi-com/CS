# 상속
`InterceptingHttpAccessor` 상속 -> `HttpAccessor` 상속
## HTTPAccessor란?
모든 HTTP 요청을 위한 기본 설정을 관리하는 역할
HTTP의 요청 직접적으로 처리 X -> HTTP 통신을 위한 기본적인 구성 요소인 `ClientHttpRequestFactory`를 설정하고 관리
### InterceptingHTTPAccessor란?
HTTPAccessor 상속
HTTP 요청을 수행하기 전후 추가적인 처리를 할 수 있는 **인터셉터** 기능 추가
> ❗**인터셉터**
> - 요청, 응답 조작
> - 로깅
> - 인증 토큰 추가
> - 매트릭 측정 등

```java
public abstract class HttpAccessor {
	//...
	
	private ClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
	
	//...
}
```
ClientHttpRequestFactory interface에 대하여 기본적으로 `SimpleClientHttpRequestFactory` 사용
>**SimpleClientHttpRequestFactory**
>- Java 표준 `HttpURLConnection` 클래스 사용
>- 외부 라이브러리 의존 없이 HTTP 요청 수행 가능
>  
>=> 복잡한 요구사항, 연결 풀링, keep-alive, 멀티파트 등의 요청의 경우 다른 ClientHttpRequestFactory를 사용하는 것이 좋음 
# 특징
## Multi-thread
![](https://i.imgur.com/u8OQzGA.png)
- Thread pool은 어플리케이션 구동 시에 미리 만듦
- Request는 queue에 쌓이고, 가용 스레드가 있다면 그 스레드에 할당되어 처리 -> 1 요청 당 1 스레드
## Synchronous Blocking
![](https://i.imgur.com/CPCCAN0.png)
할당된 스레드에서 Blocking이 되어 응답이 돌아올 때까지 다른 요청에 할당 불가능

>비동기 논블로킹 요청의 경우 [[WebClient]]에서 사용됨
## HTTP 요청, 응답 자동 변환 & 역직렬화
- **MessageConverter** 사용
```java
public RestTemplate() {
	this.messageConverters.add(new ByteArrayHttpMessageConverter());
	this.messageConverters.add(new StringHttpMessageConverter());
	this.messageConverters.add(new ResourceHttpMessageConverter(false));
	
	if (!shouldIgnoreXml) {
		try {
			this.messageConverters.add(new SourceHttpMessageConverter<>());
		}
		catch (Error err) {
		// Ignore when no TransformerFactory implementation is available
		}
	}
	
	this.messageConverters.add(new AllEncompassingFormHttpMessageConverter());
	
	if (romePresent) {
		this.messageConverters.add(new AtomFeedHttpMessageConverter());
		this.messageConverters.add(new RssChannelHttpMessageConverter());
	}
	
	if (!shouldIgnoreXml) {
		if (jackson2XmlPresent) {
			this.messageConverters.add(new MappingJackson2XmlHttpMessageConverter());
		}
		else if (jaxb2Present) {
			this.messageConverters.add(new Jaxb2RootElementHttpMessageConverter());
		}
	}
	
	if (jackson2Present) {
		this.messageConverters.add(new MappingJackson2HttpMessageConverter());
	}
	else if (gsonPresent) {
		this.messageConverters.add(new GsonHttpMessageConverter());
	}
	else if (jsonbPresent) {
		this.messageConverters.add(new JsonbHttpMessageConverter());
	}
	else if (kotlinSerializationJsonPresent) {
		this.messageConverters.add(new KotlinSerializationJsonHttpMessageConverter());
	}
	
	if (jackson2SmilePresent) {
		this.messageConverters.add(new MappingJackson2SmileHttpMessageConverter());
	}
	if (jackson2CborPresent) {
		this.messageConverters.add(new MappingJackson2CborHttpMessageConverter());
	}
	this.uriTemplateHandler = initUriTemplateHandler();
}
```
# 동작원리
![](https://i.imgur.com/P6bZ0gZ.png)
1. 애플리케이션이 RestTeamplate 생성하고, URI, HTTP 메소드 등을 헤더에 담아 요청
2. RestTemplate의 HttpMessageConverter가 Java 객체를 Json으로 변환
3. ClientHttpRequestFactory가 ClientHttpRequest 인스턴스 생성 -> 실제로 서버에 HTTP 요청을 보내는데 사용
4. ClientHttpRequest를 사용하여 설정된 HTTP 메소드와 URI를 가진 HTTP 요청을 REST API에 전송
5. RestTemplate은 ResponseErrorHandler를 통해 오류 확인 (4xx, 5xx)
6. ResponseErrorHandler는 오류가 있다면 ClientHttpResponse에서 응답 데이터를 가져와 처리
7. HttpMessageConverter를 이용해 응답 메시지를 Java 객체로 변환
8. 애플리케이션에 response 반환
# 메소드
## doExcute
제공된 URI 실행
```java
protected <T> T doExecute(URI url, @Nullable HttpMethod method, @Nullable RequestCallback requestCallback,
		@Nullable ResponseExtractor<T> responseExtractor) throws RestClientException {
	
	Assert.notNull(url, "URI is required");
	Assert.notNull(method, "HttpMethod is required");
	ClientHttpResponse response = null;
	try {
		ClientHttpRequest request = createRequest(url, method);
		if (requestCallback != null) {
			requestCallback.doWithRequest(request);
		}
		response = request.execute();
		handleResponse(url, method, response);
		return (responseExtractor != null ? responseExtractor.extractData(response) : null);
	}
	catch (IOException ex) {
		String resource = url.toString();
		String query = url.getRawQuery();
		resource = (query != null ? resource.substring(0, resource.indexOf('?')) : resource);
		throw new ResourceAccessException("I/O error on " + method.name() + 
			" request for \"" + resource + "\": " + ex.getMessage(), ex);
	}
	finally {
		if (response != null) {
			response.close();
		}
	}
}
```
### \<ClientHttpRequest> createRequest
```java
@Override
public ClientHttpRequest createRequest(URI uri, HttpMethod httpMethod) throws IOException {
	HttpURLConnection connection = openConnection(uri.toURL(), this.proxy);
	prepareConnection(connection, httpMethod.name());
	if (this.bufferRequestBody) {
		return new SimpleBufferingClientHttpRequest(connection, this.outputStreaming);
	}
	else {
		return new SimpleStreamingClientHttpRequest(connection, this.chunkSize, this.outputStreaming);
	}
}

protected HttpURLConnection openConnection(URL url, @Nullable Proxy proxy) throws IOException {
	URLConnection urlConnection = (proxy != null ? url.openConnection(proxy) : url.openConnection());
	if (!(urlConnection instanceof HttpURLConnection)) {
		throw new IllegalStateException(
			"HttpURLConnection required for [" + url + "] but got: " + urlConnection);
	}
	return (HttpURLConnection) urlConnection;
}

protected void prepareConnection(HttpURLConnection connection, String httpMethod) throws IOException {
	if (this.connectTimeout >= 0) {
		connection.setConnectTimeout(this.connectTimeout);
	}
	if (this.readTimeout >= 0) {
		connection.setReadTimeout(this.readTimeout);
	}
	
	boolean mayWrite =
		("POST".equals(httpMethod) || "PUT".equals(httpMethod) ||
			"PATCH".equals(httpMethod) || "DELETE".equals(httpMethod));
	
	connection.setDoInput(true);
	connection.setInstanceFollowRedirects("GET".equals(httpMethod));
	connection.setDoOutput(mayWrite);
	connection.setRequestMethod(httpMethod);
}
```
#### openConnection
- 주어진 URI에 대한 `HTTPURLConnection` 객체 생성
- proxy가 있을 경우 이를 사용
#### prepareConnection
- 객체 커넥션, 읽기 타임아웃 적용
- `POST`, `PUT`, `PATCH`, `DELETE`의 경우, 요청 본문에 데이터를 쓸 수 있어야 하므로 `mayWrite = true` -> `doOutput = true`
- `Get`의 경우, 자동으로 리다이렉션을 따르도록 함 `instanceFollowRedirects = true`
- 모든 요청이 응답 데이터를 읽을 수 있어야 하므로 `doInput = true`
- HTTP 메소드를 `HTTPURLConnection` 객체에 설정
#### CreateRequest
- Buffering 혹은 Streaming 방식 사용
> **Buffering**
> - 모든 데이터를 메모리에 저장한 후, 요청 본문 전체를 한 번에 서버로 전송
> **Streaming**
> - 요청 본문을 작은 조각(chunk)로 나누어 서버로 바로 전송

### \<RestTemplate-HttpEntityRequestCallback-doWithRequest>
```java
public void doWithRequest(ClientHttpRequest httpRequest) throws IOException {
	super.doWithRequest(httpRequest);
	Object requestBody = this.requestEntity.getBody();
	if (requestBody == null) {
		HttpHeaders httpHeaders = httpRequest.getHeaders();
		HttpHeaders requestHeaders = this.requestEntity.getHeaders();
		if (!requestHeaders.isEmpty()) {
			requestHeaders.forEach((key, values) -> httpHeaders.put(key, new ArrayList<>(values)));
		}
		if (httpHeaders.getContentLength() < 0) {
			httpHeaders.setContentLength(0L);
		}
	}
	else {
		Class<?> requestBodyClass = requestBody.getClass();
		Type requestBodyType = (this.requestEntity instanceof RequestEntity ?
			((RequestEntity<?>)this.requestEntity).getType() : requestBodyClass);
		HttpHeaders httpHeaders = httpRequest.getHeaders();
		HttpHeaders requestHeaders = this.requestEntity.getHeaders();
		MediaType requestContentType = requestHeaders.getContentType();
		for (HttpMessageConverter<?> messageConverter : getMessageConverters()) {
			if (messageConverter instanceof GenericHttpMessageConverter) {
				GenericHttpMessageConverter<Object> genericConverter =
					(GenericHttpMessageConverter<Object>) messageConverter;
				if (genericConverter.canWrite(requestBodyType, requestBodyClass, requestContentType)) {
					if (!requestHeaders.isEmpty()) {
						requestHeaders.forEach((key, values) -> httpHeaders.put(key, new ArrayList<>(values)));
					}
					logBody(requestBody, requestContentType, genericConverter);
					genericConverter.write(requestBody, requestBodyType, requestContentType, httpRequest);
					return;
				}
			}
			else if (messageConverter.canWrite(requestBodyClass, requestContentType)) {
				if (!requestHeaders.isEmpty()) {
					requestHeaders.forEach((key, values) -> httpHeaders.put(key, new ArrayList<>(values)));
				}
				logBody(requestBody, requestContentType, messageConverter);
				((HttpMessageConverter<Object>) messageConverter).write(
					requestBody, requestContentType, httpRequest);
				return;
			}
		}
		String message = "No HttpMessageConverter for " + requestBodyClass.getName();
		if (requestContentType != null) {
			message += " and content type \"" + requestContentType + "\"";
		}
		throw new RestClientException(message);
	}
}
```
1️⃣ **요청 본문이 없는 경우**
- `HttpHeaders` 객체에 요청 헤더 복사
- `Content-Length`가 설정되지 않았다면 (`< 0`), 0으로 설정
2️⃣ **요청 본문이 있는 경우**
- 요청 본문 타입, 클래스, Content-type 결정
- 사용 가능한 `HttpMessageConverter` 목록을 순회하며, 변환기 찾고 해당 변환기의 `write` 메소드를 호출
### \<AbstractGenericHttpMessageConverter> write
```java
public final void write(final T t, @Nullable final Type type, @Nullable MediaType contentType,
		HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
	
	final HttpHeaders headers = outputMessage.getHeaders();
	addDefaultHeaders(headers, t, contentType);
	if (outputMessage instanceof StreamingHttpOutputMessage) {
		StreamingHttpOutputMessage streamingOutputMessage = (StreamingHttpOutputMessage) outputMessage;
		streamingOutputMessage.setBody(outputStream -> writeInternal(t, type, new HttpOutputMessage() {
			@Override
			public OutputStream getBody() {
				return outputStream;
			}
			@Override
			public HttpHeaders getHeaders() {
				return headers;
			}
		}));
	}
	else {
		writeInternal(t, type, outputMessage);
		outputMessage.getBody().flush();
	}
}
```
스트림으로 body 내용 전송
`ClientHttpResponse.getBody()`를 통해 `InputStream` 형태로 얻을 수 있음

사용자가 `execute` 메소드 호출 시 내부적으로 `doExecute` 호출
`SimpleBufferingClientHttpRequest` 혹은 `SimplestreamingCientHttpRequest`가 `ClientHttpRequest`의 구현체이므로 해당 구현체의 `executeInternal`이 내부적으로 실행 
-> 이들이 구현하는 `AbstractClientHttpRequest`의 `execute`가 `exeucteInternal`을 사용하며 해당 메소드는 추상메소드로 구현체에서 정의
### \<SimpleBufferingClientHttpRequest> executeInternal
```java
protected ClientHttpResponse executeInternal(HttpHeaders headers, byte[] bufferedOutput) throws IOException {
	addHeaders(this.connection, headers);
	
	// JDK <1.8 doesn't support getOutputStream with HTTP DELETE
	if (getMethod() == HttpMethod.DELETE && bufferedOutput.length == 0) {
		this.connection.setDoOutput(false);
	}
	
	if (this.connection.getDoOutput() && this.outputStreaming) {
		this.connection.setFixedLengthStreamingMode(bufferedOutput.length);
	}
	this.connection.connect();
	if (this.connection.getDoOutput()) {
		FileCopyUtils.copy(bufferedOutput, this.connection.getOutputStream());
	}
	else {
	// Immediately trigger the request in a no-output scenario as well
		this.connection.getResponseCode();
	}
	return new SimpleClientHttpResponse(this.connection);
}
```
`AbstractClientHttpRequest`의 구현체 중 하나인 `SimpleBufferingClientHttpRequest`의 `executeInternal`
- 헤더 추가(`addHeaders`)
- 서버 연결(`HttpURLConnection.connect()`)
- 응답 코드 가져오기(`HttpURLConnection.getResponseCode()`)
## getForObject
HTTP GET  요청 후 결과는 객체로 반환
```java
public <T> T getForObject(String url, Class<T> responseType, Object... uriVariables) throws RestClientException {
	RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
	HttpMessageConverterExtractor<T> responseExtractor = 
		new HttpMessageConverterExtractor<>(responseType, getMessageConverters(), logger);
	return execute(url, HttpMethod.GET, requestCallback, responseExtractor, uriVariables);
}
```
### 사용 예시
```java
String result = restTemplate.getForObject("http://example.com/resource", String.class);
```
## getForEntity
HTTP GET 요청 후 결과는 ResponseEntity로 반환
```java
public <T> ResponseEntity<T> getForEntity(URI url, Class<T> responseType) throws RestClientException {
	RequestCallback requestCallback = acceptHeaderRequestCallback(responseType);
	ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
	return nonNull(execute(url, HttpMethod.GET, requestCallback, responseExtractor));
}
```
### 사용 예시
```java
ResponseEntity<String> response = restTemplate.getForEntity("http://example.com/resource", String.class);
```
## postForLocation
HTTP Post 요청 후 결과 헤더에 저장된 URI를 반환
```java
public URI postForLocation(String url, @Nullable Object request, Object... uriVariables)
		throws RestClientException {
	RequestCallback requestCallback = httpEntityCallback(request);
	HttpHeaders headers = execute(url, HttpMethod.POST, requestCallback, headersExtractor(), uriVariables);
	return (headers != null ? headers.getLocation() : null);

}
```
### 사용 예시
```java
URI location = restTemplate.postForLocation("http://example.com/resource", newResource);
```
## postForObject
HTTP Post 요청 후 결과는 객체로 반환
```java
public <T> T postForObject(String url, @Nullable Object request, Class<T> responseType,
	Object... uriVariables) throws RestClientException {
	
	RequestCallback requestCallback = httpEntityCallback(request, responseType);
	HttpMessageConverterExtractor<T> responseExtractor =
		new HttpMessageConverterExtractor<>(responseType, getMessageConverters(), logger);
	return execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables);
}
```
### 사용 예시
```java
String result = restTemplate.postForObject("http://example.com/resource", newResource, String.class);
```
## postForEntity
HTTP Post 요청 후 결과는 ResponseEntity로 반환
```java
public <T> ResponseEntity<T> postForEntity(String url, @Nullable Object request,
	Class<T> responseType, Object... uriVariables) throws RestClientException {
	
	RequestCallback requestCallback = httpEntityCallback(request, responseType);
	ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
	return nonNull(execute(url, HttpMethod.POST, requestCallback, responseExtractor, uriVariables));
}
```
### 사용 예시
```java
ResponseEntity<String> response = restTemplate.postForEntity("http://example.com/resource", newResource, String.class);
```
## delete
HTTP DELETE 요청
```java
public void delete(String url, Object... uriVariables) throws RestClientException {
	execute(url, HttpMethod.DELETE, null, null, uriVariables);
}
```
### 사용 예시
```java
restTemplate.delete("http://example.com/resource/{id}", 1);
```
## headForHeaders
HTTP HEAD 요청 후 헤더 정보 반환
```java
public HttpHeaders headForHeaders(String url, Object... uriVariables) throws RestClientException {
	return nonNull(execute(url, HttpMethod.HEAD, null, headersExtractor(), uriVariables));
}
```
#### 사용 예시
```java
HttpHeaders headers = restTemplate.headForHeaders("http://example.com/resource");
```
## put
HTTP PUT 요청
```java
public void put(String url, @Nullable Object request, Object... uriVariables)
		throws RestClientException {
	
	RequestCallback requestCallback = httpEntityCallback(request);
	execute(url, HttpMethod.PUT, requestCallback, null, uriVariables);
}
```
#### 사용 예시
```java
restTemplate.put("http://example.com/resource/{id}", updatedResource, 1);
```
## patchForObject
HTTP PATCH 요청 후 결과는 객체로 반환
```java
public <T> T patchForObject(String url, @Nullable Object request, Class<T> responseType,
		Object... uriVariables) throws RestClientException {
	
	RequestCallback requestCallback = httpEntityCallback(request, responseType);
	HttpMessageConverterExtractor<T> responseExtractor =
		new HttpMessageConverterExtractor<>(responseType, getMessageConverters(), logger);
	return execute(url, HttpMethod.PATCH, requestCallback, responseExtractor, uriVariables);
}
```
#### 사용 예시
```java
String result = restTemplate.patchForObject("http://example.com/resource/{id}", updatedPart, String.class, 1);
```
## exchange
원하는 HTTP 메소드 요청 후 결과는 ResponseEnntity로 반환
```java
public <T> ResponseEntity<T> exchange(String url, HttpMethod method,
	@Nullable HttpEntity<?> requestEntity, Class<T> responseType, Map<String, ?> uriVariables)
		throws RestClientException {
	
	RequestCallback requestCallback = httpEntityCallback(requestEntity, responseType);
	ResponseExtractor<ResponseEntity<T>> responseExtractor = responseEntityExtractor(responseType);
	return nonNull(execute(url, method, requestCallback, responseExtractor, uriVariables));
}
```
#### 사용 예시
```java
String result = restTemplate.execute("http://example.com/resource", HttpMethod.GET, requestCallback, responseExtractor);
```