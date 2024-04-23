## 개념
- Cross Origin Resource Sharing : 교차 출처 리소스 공유
- **동일 출처 정책** : 한 웹 페이지(출처 A)에서 다른 웹 페이지(출처 B)의 데이터를 직접 불러오는 것을 제한
- 한 웹 페이지가 다른 출처의 리소스에 접근할 수 있도록 '허가'를 구하는 방법
- ***브라우저***가 다른 웹 페이지의 요청을 대신해서 해당 데이터를 사용해도 되는지 다른 출처에게 물어봄
- 브라우저가 클라이언트(A)의 요청 헤더와 서버(B)의 응답 헤더를 비교해서 최종 응답 결정
	- **Protocol**, **Host**, **Port** 비교
### 동일 출처 기준

| URL                                     | 동일 출처 여부 | 근거             |
| --------------------------------------- | -------- | -------------- |
| https://security.io/auth?username=user1 | O        | 스킴, 호스트, 포트 동일 |
| https://user:password@security.io       | O        | 스킴, 호스트, 포트 동일 |
| http://security.io                      | X        | 스킴 다름          |
| https://api.security.io                 | X        | 호스트 다름         |
| https://security.io:8000                | ?        | 브라우저 구현에 따라 다름 |

## 종류
### 1️⃣ Simple Request
- 예비 요청 (Preflight) 과정 없이 자동으로 CORS 작동하여 서버에 요청
- 서버가 응답 헤더에 Access-Control-Allow-Origin과 같은 값을 전송하면 브라우저가 비교 후 CORS 정책 위반 여부 검사
#### 제약사항
- GET, POST, HEAD만 가능
- 헤더는 Accept, Accept-Language, Content-Language, Content-Type, DPR, Downlink, Save-Data, Viewport-Width만 가능하고 커스텀 헤더 적용 X
- Content-type은 application/x-www-form-urlencoded, multipart/form-data, text/plain만 가능
#### 흐름
![](https://i.imgur.com/7vnOGPB.png)
### 2️⃣ Preflight Request
- 브라우저가 요청을 한번에 보내지 않음
- 예비 요청(Preflight)과 본 요청으로 나누어 서버에 전달
- 예비 요청 메소드에 OPTIONS 사용
#### 흐름
![](https://i.imgur.com/A2ERucX.png)
## 서버에서 Access-Control-Allow-* 세팅
- **Access-Control-Allow-Origin**
	- 헤더에 작성된 출처만 브라우저가 리소스를 접근할 수 있음
	- \* https://security.io
- **Access-Control-Allow-Methods**
	- Preflight request에 대한 응답으로 실제 요청 중에 사용할 수 있는 메서드
	- 기본값 : GET, POST, HEAD, OPTIONS, \*
- **Access-Control-Allow-Headers**
	- Preflight request에 대한 응답으로 실제 요청 중에 사용할 수 있는 헤더 필드 이름
	- 기본값 : Origin, Accept, X-Requested-With, Content-Type, Access-Control-Request-Method, Access-Control-Request-Headers, Custom Header, \*
- **Access-Control-Allow-Credentials**
	- 실제 요청에 쿠키나 인증 등의 자격 증명이 포함될 수 있음
	- Client의 `credentials:include` 옵션일 경우 true 필수
- **Access-Control-Max-Age**
	- Preflight 요청 결과를 캐시 할 수 있는 시간을 나타낸 것으로 해당 시간 동안은 preflight 요청을 하지 않음
## API
- CORS의 pre-flight request에는 쿠키(JSESSIONID)가 포함되어 있지 않기 때문에 [[Spring Security]] 이전에 처리
- 사전 요청에 쿠키가 없고 Spring Security가 먼저 처리되면 사용자가 인증되지 않았다고 판단하고 거부 가능
- CORS를 먼저 처리하기 위해 CorsFilter 사용 & CorsConfigurationSource를 제공하여 Spring Security와 통합

```java
@Bean
SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
	http.cors(cors -> cors.configurationSource(corsConfigurationSource()));

	return http.build();
}

@Bean
public CorsConfigurationSource corsConfigurationSource() {
	CorsConfiguration configuration = new CorsConfiguration();
	configuration.addAllowedOrigin("https://example.com");
	configuration.addAllowedMethod("GET", "POST");
	configuration.setAllowCredentials(true);
	UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
}
```