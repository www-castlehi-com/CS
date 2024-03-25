# Application Layer란?
- 각 서비스의 종단 시스템에서 동작
- 네트워크를 통해 서로 통신
- 서버와 클라이언트 존재
## 구조
### 1️⃣ 서버-클라이언트
#### 서버
- 항상 동작하고 있는 호스트
- 다른 호스트들로부터 서비스 요청을 받음
- 고정 IP와 같은 **잘 알려진 IP**를 주소로 가짐
- 대규모일 경우 **데이터 센터**를 사용
#### 클라이언트
- 서버로 요청을 보내는 호스트
- 서로 직접적으로 통신하지 않음
![](https://i.imgur.com/rgCivDC.png)

1. 클라이언트가 서버에게 객체를 요청함
2. 서버는 클라이언트가 요청한 객체를 반환하며 응답함

> ex) 웹, 파일 전송, 원격 로그인, 전자 메일 등
### 2️⃣ P2P 구조
![](https://i.imgur.com/rE8dK5O.png)
- 피어(peer)라는 호스트 쌍이 서로 직접 통신
- 특정 서버를 통하지 않고 사용자들이 제어하는 peer가 직접 통신하므로 peer-to-peer(**P2P**)라고 함
- **자가확장성** : 객체를 요청한 피어는 응답을 받은 후 해당 파일을 다른 피어들에게 분배하여 작업 부하 감소시킴
> ex) 비트토렌트
#### tit-for-tat
- 자신에게 더 빨리 객체를 제공하는 클라이언트에게 우선 제공함

![](https://i.imgur.com/Nbt7hIP.png)
## 소켓 통신
### 소켓
![](https://i.imgur.com/fUqbmvm.png)
- 호스트의 애플리케이션 계층과 전송 계층 간의 인터페이스
- 애플리케이션 개발자의 경우, 소켓의 모든 통제권을 가짐
- 소켓의 경우, 전송 계층에 대한 통제권을 거의 갖지 못함
	> 1) 전송 프로토콜 선택
	> 2) 최대 버퍼와 최대 세그먼트 크기 등 전송 계층 파라미터 설정
### 프로세스 식별
1. 호스트 주소 (**IP 주소**)
	- 32 비트
	- 송수신 호스트 식별
2. 호스트 내의 수신 프로세스 / 소켓 (**Port 번호**)
	- 인기 있는 애플리케이션의 경우 특정한 포트 번호 할당
		> ex) 웹서버 : 80 | SMTP : 25 등
## application layer가 사용 가능한 transport layer 서비스
### 1️⃣ reliable-data-transfer
- 패킷은 네트워크 내에서 유실될 수 있음
	> ex) 버퍼에서의 오버플로우로 인한 유실, 패킷의 비트가 잘못되었을 시 라우터에 의해 손실
- 송신 프로세스가 데이터를 소켓으로 보내고, 해당 데이터가 오류 없이 수신 프로세스에 도착함을 보장

> 💡 **손실 허용 어플리케이션**(오디오/비디오 등)의 경우, 신뢰적 데이터 전송을 하지 않아도 약간의 데이터 손실을 감수할 수 있음
### 2️⃣ throughput
- 네트워크 경로를 따라 두 프로세스 간의 통신 세션에서 송신 프로세스가 수신 프로세스로 비트를 전달할 수 있는 비율
- 다른 세션들이 함께 대역폭을 공유하기 때문에 시간에 따라 변동됨
	![](https://i.imgur.com/DMcrDy5.png)
- **대역폭 민감 애플리케이션**(음성 / 비디오)은 처리율 요구사항을 가짐
> 💡 **탄력적 애플리케이션**(전자메일, 파일 전송, 웹 전송)은 처리율과 상관없이 이용 가능하지만 다다익선
### 3️⃣ 시간
- 게임, 인터넷 전화, 원격 회의 등의 어플리케이션은 엄격한 시간 제한이 요구됨
### 4️⃣ 보안
- 하나 이상의 보안 서비스 제공 가능
- 암호화(기밀성), 무결성, 인증 등
## Application Layer가 사용 가능한 transport layer [[프로토콜]]
### 1️⃣ TCP
#### 연결 지향형
- 3-hand shaking, 4-hand shaking으로 클라이언트와 서버가 연결-종료를 진행
- 두 프로세스가 서로 동시에 메시지를 보낼 수 있기 때문에 full-duplex 연결이라고 함
#### reliable-data-transfer
- 패킷이 손실되거나 중복되지 않게 전달
#### flow-control
- 수신자의 버퍼 크기에 맞게 데이터 스트림을 조절
#### congestion-control
- 송신자가 데이터 스트림을 조절

> ⚠️ 보안을 제공하지 않기 때문에 현대 네트워크에서는 TCP + TLS를 사용함
> TLS는 transfer layer가 아닌 application layer로서 TCP를 강화하는 애플리케이션
### 2️⃣ UDP
#### 비연결형
- hand-shaking을 진행하지 않음
#### unreliable-data-transfer
- UDP 소켓으로 메시지 전송 시, 그 메시지가 목적지 소켓에 전달하는 것을 보장하지 않음
- 도착 패킷에 대한 순서 보장하지 않음
#### 혼잡 제어 없음
- 원하는 속도로 하위 계층에 전송

> 💡 인터넷 속도의 증가로 많은 애플리케이션이 TCP를 사용하고 있으며 UDP 통신이 실패할 경우, TCP로 전환됨

## Application Layer Protocol
### 1️⃣ HTTP
### 2️⃣ FTP
### 3️⃣ SMTP
### 4️⃣ DNS
### 5️⃣ SSL / TLS
# Web & HTTP
## HTTP란?
브라우저와 웹 서버 사이에서 교환되는 메시지의 format, order 정의
**TCP**를 transport layer protocol로 사용 -> 모든 메시지는 안전하게 서버, 클라이언트에 도착
**Stateless** 프로토콜 : 클라이언트에 대한 정보를 유지하지 않음

1. HTTP 클라이언트가 서버에 TCP 연결 시작
2. 연결 후, 소켓 API를 통해 TCP로 접속
3. 클라이언트는 소켓 API를 통해 요청을 보내고 서버로부터 응답을 받음
## 비지속 연결
- 각 요구/응답 쌍이 분리된 TCP 연결을 통해 전달됨
- HTTP/1.0

예시 : 클라이언트는 서버에게 파일을 요청하며 이 파일에는 10개의 JPEG 이미지가 존재
연결 수행 과정
1. HTTP 포트인 80번을 통해 TCP 연결 시도
2. TCP 소켓을 통해 서버로 HTTP 파일을 요청하는 요청 메시지를 송신
3. HTTP 서버는 저장장치로부터 해당 파일을 추출하며 객체로 캡슐화하여 소켓을 통해 클라이언트에게 전송하여 응답
4. HTTP 서버는 TCP에게 TCP 연결 종료 요청
	> 바로 종료되는 것은 아니며, 클라이언트가 무사히 데이터를 전송 받았는지 확인한 후 종료됨
5. TCP 연결 중단되며, 클라이언트는 파일에서 10개의 JPEG를 발견하고 이에 대한 참조를 찾음
6. 이후 참조되는 JPEG 이미지에 대해서 1 ~ 4 단계 반복

=> 총 11개의 TCP 연결

![](https://i.imgur.com/5yxGHhn.png)


**단점**
- 각 요청 객체에 대한 새로운 연결이 설정되고 유지되어야 함
- 각 객체는 2RTT를 필요로 함(TCP 연결에 1 RTT, 객체 송수신에 1 RTT)
	결국, 총 시간은 2RTT + file transfer time
## 지속 연결
- 응답을 보낸 후, TCP 연결을 그대로 유지 -> 동일한 클라이언트-서버 간의 요청/응답은 같은 TCP 연결 내에서 보내짐
- HTTP/1.1
### HOL 블로킹
- HTTP/1.1에서 하나의 TCP 연결을 이용할 시, 트래픽을 많이 요구하지 않는 작은 객체들이 트래픽을 많이 요구하는 객체 뒤에서 응답을 기다릴 경우 지연이 증가함
- 파이프라이닝을 통해 연속적으로 객체를 받아 해결
	![](https://i.imgur.com/uBJG4aA.png)
## Message Format
### 1️⃣ Request 메시지
![](https://i.imgur.com/xF6NnGm.png)
> 💡 각 줄은 CRLF(`\r\n`)로 이루어져 있음
1. 요청 라인 : \<method> \<url> \<version>
	**method 필드** - GET, POST, DELETE, PUT, HEAD, PATCH ...
2. 헤더 라인 : 객체가 존재하는 호스트 명시
	> TCP 연결이 맺어져 있어 불필요할 수 있지만, 웹 프록시 캐시에서 사용
3. 개체 몸체 : GET은 비워지며, POST 방식에서 사용

예시
```
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```
### 2️⃣ Response 메시지
![](https://i.imgur.com/lYWDj9S.png)
1. 상태 라인
	**status code & 문장** : 요청 결과
		- 200 OK : 요청이 성공했고, 정보가 보내짐
		- 301 Moved Permanently : 요청 객체가 영원히 이동 -> 새로운 URL이 있을 경우 자동으로 추출
		- 400 Bad Request : 서버가 요청을 이해할 수 없음
		- 404 Not Found :  요청 문서가 서버에 존재하지 않음
		- 505 HTTP Version Not Supported : 요청 HTTP 프로토콜 버전을 서버가 지원하지 않음
2. 헤더 라인
3. 개체 몸체

예시
```
HTTP/1.1 200 OK
Connection: close
Date: Tue, 18 Aug 2015 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tue, 18 Aug 2015 15:11:03 GMT
Content-Length: 6821
Content-Type: text/html

(data data data data data ...)
```
## 쿠키
사용자에 따라 콘텐츠를 제공하기 위해 사용자를 확인
사용자의 웹 이용 경험을 간소화시킬 수 있지만, 개인정보가 담겨 있는 쿠키를 제 3자에게 판매할 수 있기 때문에 보안 위험 존재
![](https://i.imgur.com/ERNffsa.png)
1. HTTP 응답 메시지 쿠키 헤더 라인
	-> 웹 사이트 접속 시, 웹사이트가 해당 사용자에 대한 유일한 식별번호를 만듦 `Set-cookie: 1678`
2. 웹사이트의 백엔드 데이터베이스
	-> 해당 식별번호로 인덱싱되는 백엔드 데이터베이스 안에 엔트리를 만듦
3. 사용자의 브라우저에 쿠키 파일
	-> 응답 메시지로 받은 쿠키를 브라우저에 저장
4. HTTP 요청 메시지 쿠키 헤더 라인
	-> 브라우저에서 참조한 쿠키 파일을 HTTP 쿠키 헤더에 담아 전달

ex) 장바구니 저장, 개인정보 저장
## 웹 캐싱 (프록시 서버)
자체의 저장 디스크를 가족 있어 최근 호출된 객체의 사본을 저장, 보존함
사용자의 모든 요청이 웹 캐시에 먼저 전달됨
![](https://i.imgur.com/iuc9Um9.png)
1. 브라우저는 웹 캐시와 TCP 연결을 수립하고 HTTP 요청을 보냄
2. 웹 캐시는 객체의 사본이 본인에게 저장되어 있는지 확인하고 저장되어 있다면 웹 캐시는 클라이언트 브라우저로 HTTP 응답 메시지와 함께 객체 전송
3. 웹 캐시가 객체의 사본을 가지고 있지 않다면, 기점 서버로 TCP 연결 수립 후, HTTP 요청을 하여 기점 서버로부터 HTTP 응답 메시지와 객체를 전달 받음
4. 웹 캐시는 전달받은 객체를 저장장치에 복사하고 클라이언트에게 HTTP 응답 메시지와 객체의 사본 전달
5. 추가적인 요청, 응답이 더 이상 존재하지 않다면 TCP 연결 종료

> 💡 캐시는 서버이자 클라이언트
### 장점
- 클라이언트 요구에 대한 응답 시간 감소
	![](https://i.imgur.com/6Gr5JiO.png)
	: 클라이언트와 기점 서버 간의 대역폭이 클라이언트와 캐시 사이의 대역폭보다 작을 때 효과적
- 웹 트래픽을 대폭 줄일 수 있음
	: 리소스를 기점 서버로부터 다시 받아올 필요가 없기 때문
### 조건부 GET
웹 캐시를 사용할 경우, 캐시가 가지고 있는 객체가 최신 버전이 아닐 수 있기 때문에 객체가 최신 임을 보장하는 방법
#### 조건
1) GET 방식을 사용함
2) `If-Modified-Since` 헤더 라인을 포함함
#### 순서
1. 브라우저의 요청을 대신해 프록시 서버가 기점 서버로 HTTP 요청 메시지를 보냄
	```
	GET /fruit/kiwi.gif HTTP/1.1
	Host: www.exotiquecuisine.com
	```
2. 기점 서버는 프록시 서버에게 객체와 함께 응답 메시지를 보냄
	```
	HTTP/1.1 200 OK
	Date: Sat, 3 Oct 2015 15:39:29
	Server: Apache/1.3.0 (Unix)
	Last-Modified: Wed, 9 Sep 2015 09:23:24
	Content-Type: image/gif
	
	(data data data data data ...)
	```
3. 같은 객체를 캐시에게 요청하면 브라우저는 조건부 GET으로 갱신 조사 수행
	```
	GET /fruit/kiwi.gif HTTP/1.1
	Host: www.exotiquecuisine.com
	If-modified-since: Wed, 9 Sep 2015 09:23:24
	```
4. 변경되지 않았을 경우, 웹 서버는 클라이언트에게 다음과 같은 응답 메시지를 보냄
	```
	HTTP/1.1 304 Not Modified
	Date: Sat, 10 Oct 2015 15:39:29
	Server: Apache/1.3.0 (Unix)
	
	(empty entity body)
	```
## HTTP/2
**목표** : 하나의 TCP 연결상에서 멀티플렉싱 요청 / 응답 지연 시간을 감소
**비변경 사항** : 상태 코드, URL, 헤더 필드 등 HTTP 메소드 자체
**변경 사항** : 클라이언트와 서버 간의 데이터 포맷 방법, 전송 방법

HTTP/1.1의 문제점 
- HOL 블로킹을 피하기 위해 병렬 TCP 연결을 사용했음
- 하나의 웹페이지 전송을 위해 병렬 TCP 연결을 사용할 경우, 일종의 속임수로 다른 객체보다 더 많은 대역폭을 할당 받을 수 있음
변경 사항
- 속임수를 가지는 TCP 병렬을 최대한 줄이거나 없앰
- TCP 병렬을 감소시키면서 생기는 HOL 블로킹을 해결할 메커니즘 모색
### 1️⃣ HTTP/2 프레이밍
각 메시지를 작은 프레임으로 나누고 같은 TCP 연결에서 요청과 응답 메시지를 전달
프레이밍 서브 계층에서 쪼개고 TCP 메시지 전달 후, 프레이밍 서브 계층에서 재조립
![](https://i.imgur.com/iPN4IMi.png)

> 💡 프레이밍 서브 계층은 **데이터 링크 계층**의 일부 기능으로서, 프레임을 바이너리 인코딩함
> => 바이너리 프로토콜 : 파싱 효율적, 작은 프레임 크기, 에러에 강함
### 2️⃣ 메시지 우선순위화
프레이밍 서브 계층이 쪼갠 데이터 스트림에 1~256의 가중치 부여 -> 높은 가치일 수록 높은 우선순위
### 3️⃣ 서버 푸싱
클라이언트의 요청 없이도 추가적인 객체를 클라이언트에게 **푸시**하여 보낼 수 있음
객체에 대한 HTTP 요청을 기다리는 대신, HTML 페이지를 미리 분석하여 필요한 객체들을 식별
해당 객체들에 대한 요청이 도착하기 전에 해당 객체를 클라이언트에 보냄
### 4️⃣ HTTP/3
transport layer 프로토콜인 QUIC(Quick UDP Internet Connections)
독립적인 스트림을 하나의 연결 위에서 병렬로 전송 가능
하나의 스트림에서 손실이 발생했을 때, 다른 스트림은 전혀 영향을 받지 않고 데이터 전송이 계속되므로 HOL 블로킹 문제가 없음
![](https://i.imgur.com/pyls7H4.png)
# 이메일
![](https://i.imgur.com/rjocgxa.png)
1. **사용자 에이전트**
	- 사용자가 메시지를 읽고, 응답, 전달, 저장, 구성하게 해줌
	> ex) 아웃룩, 애플 메일, 지메일 등
2. **메일 서버**
	- 송신자의 사용자 에이전트가 메시지를 전달하면, 송신자의 메일 서버를 거치며 수신자의 메일 서버의 메일 박스에 저장됨
	- 만일 전달할 수 없을 시에는 메시지 큐에 보관하고 이후 다시 시도
3. **SMTP**
	- 어플리케이션 계층 프로토콜
	- TCP 이용
	- 메일 서버가 상대 메일 서버로 메일을 보낼 때 : SMTP 클라이언트
	- 메일 서버가 상대 메일 서버로부터 메일을 받을 때 : SMTP 서버
##  SMTP
 - 7비트 ASCII여야 함 -> 전송 용량 제한
 - 두 메일 서버가 먼 거리를 떨어져 있더라도 중간 메일 서버를 이용하지 않고 직접 연결함
 - persistent 연결을 사용함 -> 메일이 길더라도 모든 메일을 보내는데 같은 TCP 연결을 이용함
### 순서
![](https://i.imgur.com/WbdGl8R.png)
1. Alice의 사용자 에이전트가 Bob의 전자메일 주소를 제공하고, 사용자 에이전트에게 메시지를 보내라고 명령함
2. Alice의 사용자 에이전트는 메일을 Alice의 메일 서버로 보내고 메시지 큐에 저장함
3. Alice의 메일 서버에 있는 SMTP 클라이언트는 메시지를 확인하고, SMTP 서버에게 25번 포트로 TCP 연결 요청
4. hand-shaking 이후, SMTP 클라이언트는 메시지를 TCP 연결로 보냄
5. Bob의 메일 서버 호스트에서 SMTP 서버가 메시지 수신하고, Bob의 메일 서버는 메시지를 Bob의 메일 박스에 놓음
6. Bob은 사용자 에이전트를 통해 메일(POP3, IMAP 프로토콜 이용)을 읽음
### 예시
```
S:  220 hamburger.edu
C:  HELO crepes.fr
S:  250 Hello crepes.fr, pleased to meet you
C:  MAIL FROM: <alice@crepes.fr>
S:  250 alice@crepes.fr ... Sender ok
C:  RCPT TO: <bob@hamburger.edu>
S:  250 bob@hamburger.edu ... Recipient ok
C:  DATA
S:  354 Enter mail, end with ”.” on a line by itself
C:  Do you like ketchup?
C:  How about pickles?
C:  .
S:  250 Message accepted for delivery
C:  QUIT
S:  221 hamburger.edu closing connection
```
#### 명령
- **HELO** : 연결 수립 부분이며 해당 서버가 지원하는 기능 리스트를 받을 수 있음
- **MAIL FROM** : 송신자
- **RCPT TO** : 수신자
- **DATA** : 보낼 내용
- **QUIT** : 종료 문자
#### CRLF
- 메시지의 끝을 나타냄
- carriage return & line feed => (`\r\n`)
- DATA를 보낸 후에는 CRLF.CRLF로 끝냄
# DNS
Domain Name System
## 호스트 식별 방법
1. **호스트 이름**
	> ex) www.facebook.com, www.naver.com
2. IP 주소
	> ex) 121.7.106.83
## 특징
- 호스트 이름을 IP 주소로 변환시킴
- 서버들의 계층구조로 구현된 분산 데이터베이스
- 호스트가 분산 데이터베이스로 질의하도록 허락하는 Application layer 프로토콜
- UDP 프로토콜, 53번 포트
## 순서
1. 사용자 컴퓨터는 DNS 클라이언트 측을 수행
2. 브라우저는 URL로부터 호스트 이름 www.naver.com을 추출하고 그 호스트 이름을 DNS 애플리케이션 클라이언트 측에 넘김
3. DNS 클라이언트는 DNS 서버로 호스트 이름을 포함하는 질의를 보냄
4. DNS 클라이언트는 DNS 서버로부터 호스트 이름에 대한 IP 주소를 가진 응답을 받음
5. 브라우저가 DNS로부터 IP 주소를 받으면, 해당 IP 주소와 그 주소의 80번 포트에 위치하는 HTTP 서버 프로세스로 TCP 연결
## 기능
### 1️⃣ 호스트 alias
**정식호스트** 이름 : relay1.west-coast.enterprise.com
- enterprise.com
- www.enterprise.com
=> 두개의 별칭 가능
### 2️⃣ 메일 서버 alias
bob@hotmail.com은 bob@relay1.west-coast.hotmail.com일 수 있음
MX레코드라고 함
기업 웹 서버와 메일 서버는 동일한 별칭을 가질 수 있음
### 3️⃣ 부하 분산
![](https://i.imgur.com/AQLmDmY.png)xxx.amazon.com에 대한 dns 요청을 할 때 하나의 dns에 요청하는 것이 아닌 분산된 dns를 통해 요청
#### 루트 서버
- 1000개의 루트 서버 인스턴스가 있지만 이들은 13개의 다른 루트 서버 복사체이고, 12개의 다른 기관에서 관리됨
- TLD 서버의 IP 주소 제공
#### Top Level Domain 서버 (TLD서버)
- 최상위 레벨 도메인 서버
- com, org, net, edu, gov 등의 상위 레벨 도메인과 kr, uk, fr, ca, jp 같은 국가 상위 레벨 도메인에 대한 서버
- 책임 DNS 서버에 대한 IP 주소 제공
#### Authoritative DNS 서버
- 책임 DNS 서버
- 모든 DNS 레코드를 가짐
	> **DNS 레코드** : 인터넷에서 접근하기 쉬운 호스트를 가진 모든 기관이 호스트 이름을 IP로 매핑한 것
## 질의 방법
### 1️⃣ 반복적 질의
일반적인 재귀 방법
![](https://i.imgur.com/VqaThWi.png)
1. 호스트가 자신의 로컬 DNS 서버에 변환될 호스트 이름을 담은 DNS 질의 메시지 보냄
2. 로컬 DNS 서버는 이 메시지를 루트 DNS로 보냄
3. 루트 DNS 서버는 TLD 서버의 주소를 로컬 DNS로 보냄
4. 로컬 DNS 서버는 해당 TLD 서버에게 질의 메시지를 보냄
5. TLD 서버는 책임 DNS 서버의 주소를 로컬 DNS로 보냄
6. 로컬 DNS 서버는 해당 책임 DNS 서버에 질의 메시지를 보냄
7. 해당 책임 DNS 서버는 호스트 이름을 변환한 IP 주소로 응답
### 2️⃣ 재귀적 질의
![](https://i.imgur.com/2SIOKCE.png)
1. 호스트가 자신의 로컬 DNS 서버에 변환될 호스트 이름을 담은 DNS 질의 메시지 보냄
2. 로컬 DNS 서버가 루트 DNS 서버에게 DNS 질의 메시지 보냄
3. 루트 DNS 서버가 찾은 TLD DNS 서버의 IP 주소를 통해 질의 메시지를 보냄
4. TLD DNS 서버가 찾은 책임 DNS 서버의 IP 주소를 통해 질의 메시지를 보냄
5. 책임 DNS 서버는 호스트 이름을 변환한 IP 주소를 TLD DNS에게 보냄
6. TLD DNS 서버는 받은 IP 주소를 루트 DNS 서버에게 보냄
7. 루트 DNS 서버는 받은 IP 주소를 로컬 DNS 서버에게 보냄
8. 로컬 DNS 서버는 받은 IP 주소를 호스트에게 보냄 

질의 메시지 4번, 응답 메시지 4번으로 총 8번의 DNS 메시지 송수신
TLD DNS 서버가 항상 책임 DNS 서버의 IP 주소를 아는 것이 아니고 보통 Intermediate DNS 서버의 주소를 알고 있기 때문에 일반적으로는 10번의 DNS 메시지 송수신
> 💡 **DNS 캐싱**을 이용하면 효과적
## DNS 레코드
(Name, Value, Type, TTL)의 튜플로 이루어짐
### 1️⃣ Type = A
호스트 이름에 대한 IP 주소를 매핑하는 레코드
- Name : 호스트 이름
- Value : 호스트 이름에 대한 IP 주소
- Type : A
> 질의 : (www.naver.com, , A)
> 응답 : (www.naver.com, 호스트에 대한 IP 주소, A)
### 2️⃣ Type = NS
책임 DNS 서버의 호스트 이름을 찾아내는 레코드
- Name : 도메인
- Value : 도메인 내부의 호스트에 대한 IP 주소를 얻을 수 있는 방법을 아는 책임 DNS 서버의 호스트 이름
- Type : NS
> 질의 : (foo.com, , NS)
> 응답 : (foo.com, dns.foo.com, NS)
### 3️⃣ Type = CNAME
질의 호스트에게 호스트 이름에 대한 정식 이름 제공
- Name : 별칭
- Value : 호스트 이름에 대한 정식 이름
- Type : CNAME
> 질의 : (foo.com, , CNAME)
> 응답 : (foo.com, relay1.bar.foo.com, CNAME)
### 4️⃣ Type = MX
별칭 호스트 이름을 갖는 메일 서버의 정식 이름
- Name : 별칭 메일 이름
- Value : 호스트 이름에 대한 정식 메일 이름
> 질의 : (foo.com, , MX)
> 응답 : (foo.com, mail.bar.foo.com, MX)
## 단점
- DDos
	- 공격자가 다량의 패킷을 DNS 서버에 보내면 많은 DNS 질의들이 응답을 받지 못하게 됨
	- 루트 서버로 향하는 Ping 메시지를 블록해서 해결
- 중간자 공격
	- 클라이언트의 질의를 중간에 빼돌려 다른 응답을 제공함
	- DNS 보안 확장 프로토콜이 개발되어 사용됨
# 비디오 스트리밍 & 콘텐츠 분배 네트워크
## HTTP 스트리밍 & DASH
- Dynamic Adaptive Streaming over HTTP
- 비디오가 여러 가지 버전으로 인코딩되며 각 버전은 비트율과 품질 수준, URL이 다르며 해당 URL은 메니페스트 파일에 저장
- 동적으로 비디오 조각(chunk)를 요청함
	- 가용 대역폭이 충분할 때, 높은 비트율의 비디오 버전 요청
	- 가용 대역폭이 부족할 때, 낮은 비트율의 비디오 버전 요청
	> 3G 이용자의 경우 낮은 품질의 비디오를 받고, 광섬유 연결 혹은 5G 클라이언트는 고품질의 비디오를 받는 이유
- HTTP GET 요청을 통해 매번 다른 버전의 비디오 조각 선택
- 서로 다른 품질 수준을 자유롭게 변화시킬 수 있도록 허용
## CDN
넷플릭스, 유튜브 등 스트리밍 트래픽을 전세계에 걸친 지점에 끊김 없이 안정적으로 제공해야 함
1. 거대한 데이터 센터를 구축하고 모든 비디오 자료를 해당 비디오 데이터 센터로부터 전송
	- 클라이언트가 데이터 센터로부터 지역적으로 먼 지점에 있는 경우, 다양한 ISP를 거쳐야 하므로 처리율이 낮아짐
	- 인기 있는 비디오는 같은 통신 링크를 통해 여러 번 반복적으로 전송되어 대역폭이 낭비됨
	- 데이터 센터에 장애 발생 시, 전체 서비스가 중단될 수 있음
2. 다수의 지점에 분산된 데이터 센터 이용 (CDN)
	![](https://i.imgur.com/hyPNRxh.png)
	- Content Distribution Network
	- 웹 컨텐츠 복사본을 분산 서버에 저장
	- 데이터를 제공할 수 있는 가까운 CDN 서버에서 데이터를 받아옴
	- 사용자가 지역 CDN에 존재하지 않은 비디오를 요청할 경우, 중앙 데이터센터에서 CDN이 데이터를 받아오며 해당 사본을 저장하고 반환