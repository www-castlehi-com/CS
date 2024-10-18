# 일반 헤더
## 분류
#### 1️⃣ General 헤더
- 메시지 전체에 적용되는 정보
- `Connection: close`
#### 2️⃣ Request 헤더
- 요청 정보
- `User-Agent: Mozilla/5.0`
#### 3️⃣ Response 헤더
- 응답 정보
- `Server: Apache`
#### 4️⃣ Entity 헤더
- 엔티티 바디 정보
- 엔티티 본문의 데이터를 해석할 수 있는 정보 제공
- `Content-Type: text/html, Content-Length: 3423`
> **RFC 723x 변화**
> - 엔티티 -> 표현
> - Representation = Representation Metadata + Representation Data
> - 표현 = 표현 메타데이터 + 표현 데이터
> ![](https://i.imgur.com/CgkblUg.png)
> - 메시지 본문 = payload
> - 표현 헤더는 표현 데이터를 해석할 수 있는 정보 제공
## 표현
전송, 응답 둘 다 사용
#### 1️⃣ Content-Type
- 표현 데이터의 형식
- 미디어 타입, 문자 형식
> 예시
> - `text/html; charset=utf-8`
> - `application/json`
> - `image/png`
#### 2️⃣ Content-Encoding
- 표현 데이터의 압축 방식
- 데이터를 전달하는 곳에서 압축 후 인코딩 헤더 추가
- 데이터를 읽는 쪽에서 인코딩 헤더의 정보로 압축 해제
> 예시
> - gzip
> - deflate
> - identity
#### 3️⃣ Content-Language
- 표현 데이터의 자연 언어
> 예시
> - ko
> - en
> - en-US
#### 4️⃣ Content-Length
- 표현 데이터의 길이
- 바이트 단위
- Transfer-Encoding (전송 코딩)을 사용하면 Content-Length 사용 X
## 협상
클라이언트가 선호하는 표현 요청
요청 시에만 사용
### 종류
- **Accept** : 클라이언트가 선호하는 미디어 타입 전달
- **Accept-Charset** : 클라이언트가 선호하는 문자 인코딩
- **Accept-Encoding** : 클라이언트가 선호하는 압축 인코딩
- **Accept-Language** : 클라이언트가 선호하는 자연 언어
### 우선순위 1
- **Quality Values(q)** 값 사용
- 0 ~ 1, 클수록 높은 우선순위
- 생략하면 1
> `Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7`
> 1. ko-KR;q=1
> 2. ko;q=0.9
> 3. en-US;q=0.8
> 4. en;q=0.7
### 우선순위 2
- 구체적인 것이 우선
> `Accept: text/*, text/plain, text/plain;format=flowed, */*`
> 1. text/plain;format=flowed
> 2. text/plain
> 3. text/*
> 4. \*/\*
## 전송
### 전송 방식
#### 1️⃣ 단순 전송
- **Content-Length
- 한 번에 전송하고 한 번에 수신
![](https://i.imgur.com/ZbwjrMz.png)
#### 2️⃣ 압축 전송
- **Content-Encoding**
![](https://i.imgur.com/8mTjIlq.png)
#### 3️⃣ 분할 전송
- **Transfer-Encoding**
- chunk 단위 사용
- Content-Length 사용 X
![](https://i.imgur.com/C2Kjbt5.png)
#### 4️⃣ 범위 전송
- **Content-Range**
- 범위와 끝길이 전송
![](https://i.imgur.com/pmIDVbq.png)
## 일반 정보
##### 1️⃣ Form
- 유저 에이전트의 이메일 정보
- 일반적으로 잘 사용 X
- 검색 엔진 등에서 주로 사용
- 요청 시 사용
#### 2️⃣ Referer
- 이전 웹 페이지 주소
- A -> B로 이동하는 경우 B를 요청할 때 Referer: A를 포함해서 요청
- 유입 경로 분석 가능
- 요청 시 사용
> referrer의 오타
#### 3️⃣ User-Agent
- 유저 에이전트 애플리케이션 정보
- `user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7 AppleWebKit/537.36 (KHTML, Like Gecko) Chrome/86.0.4240.183 Safari/537.36`
- 통계 정보
- 어떤 종류의 브라우저에서 장애가 발생하는지 파악 가능
- 요청 시 사용
#### 4️⃣ Server
- 요청을 처리하는 ORIGIN 서버의 소프트웨어 정보
- `Server: Apache/2.2.22 (Debian)`, `Server: nginx`
- 응답 시 사용
#### 5️⃣ Date
- 메시지가 발생한 날짜와 시간
- `Date: Tue, 15 Nov 1994 08:12:31 GMT`
- 응답 시 사용
## 특별한 정보
#### 1️⃣ Host
- 요청한 호스트 정보
- 요청 시 사용
- **필수**
- 하나의 서버가 여러 도메인을 처리해야 할 때
- 하나의 IP 주소에 여러 도메인이 적용되어 있을 때
![](https://i.imgur.com/cNkaJlr.png)
#### 2️⃣ Location
- 페이지 리다이렉션
- **201 (Created)** : 요청에 의해 생성된 리소스의 URI
- **3xx (Redirection)** : 요청을 자동으로 리다이렉션 하기 위한 대상 리소스를 가리킴
#### 3️⃣ Allow
- 허용 가능한 HTTP 메서드
- **405 (Method Not Allowed)** 에서 응답에 포함해야 함
- `Allow: GET, HEAD, PUT`
#### 4️⃣ Retry-After
- 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간
- **503 (Service Unavailable)** : 서비스가 언제까지 불능인지 알려줌

1. 날짜 표기
	- `Retry-After: Fri, 31 Dec 1999 23:59:59 GMT`
2. 초단위 표기
	- `Retry-After: 120`
## 인증
#### 1️⃣ Authorization
-  클라이언트 인증 정보를 서버에 전달
- `Authorization: Basic xxxxxxxxxxx`
#### 2️⃣ WWW-Authenticate
- 리소스 접근 시 필요한 인증 방법 정의
- **401 Unauthorized** 응답과 함께 사용
- `WWW-Authenticate: Newauth realm="apps", type=1, title="Login to \"apps\", Basic realm="simple"`
## 쿠키
### Stateless
- HTTP는 무상태 프로토콜
- 클라이언트와 서버가 요청과 응답을 주고 받으면 연결이 끊어짐
- 클라이언트가 다시 요청하면 서버는 이전 요청을 기억하지 못함
- 클라이언트와 서버는 서로 상태를 유지하지 않음
### 사용
![](https://i.imgur.com/N00YgK0.png)
![](https://i.imgur.com/2MesnWo.png)
### 특징
`set-cookie: sessionId=abcde1234; expires=Sat, 26-Dec-2020 00:00:00 GMT; path=/; domain=.google.com; Secure`
#### 사용처
- 사용자 로그인 세션 관리
- 광고 정보 트래킹
#### 항상 서버에 전송
- 네트워크 트래픽 추가 유발
- 최소한의 정보만 사용 (session id, 인증 token)
- 서버에 전송하지 않고, 웹 브라우저 내부에 데이터를 저장하고 싶을 경우 웹 스토리지 (LocalStorage, sessionStorage) 참고
> ⚠️ 보안에 민감한 데이터는 저장하면 안됨
### 생명주기
`Set-Cookie: expires=Sat, 26-Dec-2020 04:39:21 GMT`
- 만료일이 되면 쿠키 삭제
`Set-Cookie: max-age=3600 (3600초)`
- 0이나 음수를 지정할 경우 쿠키 삭제

1. **세션 쿠키** : 만료 날짜 생략 시 브라우저 종료 시까지만 유지
2. **영속 쿠키** : 만료 날짜 입력 시 해당 날짜까지 유지
### 도메인
#### 1️⃣ 명시
- 명시한 문서 기준 도메인 + 서브 도메인 포함
> `domain=example.org` 지정 후 쿠키 생성
> -  `example.org` 쿠키 접근 O
> - `dev.example.org` 쿠키 접근 O
#### 2️⃣ 생략
- 현재 문서 기준 도메인만 적용
> `example.org` 쿠키 생성 후 `domain 지정 생략
> - `example.org` 쿠키 접근 O
> - `dev.example.org` 쿠키 접근 X
### 경로
- 경로를 포함한 하위 경로 페이지만 쿠키 접근 O
- 일반적으로 `path=/` 루트로 지정
> `path=/home` 지정
> - `/home`: O
> - `/home/level1` : O
> - `/home/level1/level2` : O
> - `/hello` : X
### 보안
#### 1️⃣ Secure
- 기본 : http, https를 구분하지 않고 전송
- 적용 : **https**인 경우에만 전송
#### 2️⃣ HttpOnly
- XSS (Cross-Site Scripting) 공격 방지
- 자바 스크립트에서 접근 불가
- HTTP 전송에만 사용
#### 3️⃣ SameSite
- XSRF (Cross-Site Request Forgery) 공격 방지
- 요청 도메인과 쿠키에 설정된 도메인이 같은 경우만 쿠키 전송
# 캐시 & 조건부 요청
## 캐시 기본 동작
### 캐시가 없을 경우
- 데이터가 변경되지 않아도 계속 네트워크를 통해서 데이터를 다운받아야 함
- 매우 느리고 비쌈
- 브라우저 로딩 속도가 느림 -> 느린 사용자 경험
### 캐시가 있을 경우
![](https://i.imgur.com/7tCuIP5.png)
- 캐시 가능 시간 (**Cache-Control**) 동안 네트워크 사용 X
- 비싼 네트워크 사용량 감소
- 브라우저 로딩 속도가 빨라짐 -> 빠른 사용자 경험
### 캐시 시간 초과
- 캐시 유효 시간 (**Cache-Control**)이 초과하면, 서버를 통해 데이터를 재조회하고, 캐시를 갱신
- 네트워크 다운로드가 재발생함
