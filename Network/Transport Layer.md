# Transport Layer란?
- 애플리케이션 프로세스 간의 **논리적 통신**
- end  시스템에서 구현 (라우터에서 구현하는 것이 아님) -> end-to-end 통신이 직접 이루어지는 것처럼 보이게 함
- 단위 : **세그먼트**
- 프로토콜 : **TCP**, **UDP**
## vs Network Layer
우편물 배달에서 집과 집간의 배달은 Network Layer이고, 집 내부에서 누구에게 전달할지는 Transport Layer

- Network Layer : **Host** 간의 논리적 통신
- Transport Layer : **Process** 간의 논리적 통신

> 집 내부에서의 전달은 집간의 배달에 관여하지 않음
> 💡 Transport Layer는 네트워크 코어 내에서 메시지가 이동하는 방식에 대한 권한 없음
> 
> 집간의 배달은 집 내부에서의 배달에 관여하지 않음
> 💡 네트워크 코어는 Transport Layer의 정보에 대한 권한 없음
## Protocol
### 1️⃣ **TCP**
- Transmission Control Protocol
- 신뢰적이고, 순서 있는 전달
- Congestion Control : 송신 측이 속도 조절
- Flow Control : 수신 측이 속도 조절
- 3way-4way handshaking -> 연결, 종료 수립 과정 필요
- 헤더에 오류 검출 필드 -> 무결성 검사
### 2️⃣ UDP
- User Datagram Protocol
- 비 신뢰적, 순서 없는 전달
- 혼잡 고려 X
- 헤더에 오류 검출 필드 -> 무결성 검사
- best-effort IP 그 자체
	- IP : Network Layer의 프로토콜로서 비 신뢰적인 서비스 제공
# with 소켓
![](https://i.imgur.com/GB0hJGe.png)
소켓은 UDP, TCP인지에 따라 식별자가 달라짐 -> 패킷을 적절한 소켓에게 보낼 수 있어야 함
### 1️⃣ Multiplexing
- 각 데이터에 헤더 정보를 모아 캡슐화하고 네트워크 계층으로 전달하는 과정
### 2️⃣ Demultiplexing
- Transport Layer에서 세그먼트의 데이터를 올바른 소켓으로 전달하는 작업
- UDP 소켓의 구조
	1) 유일한 식별자
	2) 출발지 포트 번호 필드, 목적지 포트 번호 필드
- TCP 소켓의 구조
	1) 유일한 식별자
	2) 출발지 IP, 출발지 포트 번호 필드, 목적지 IP, 목적지 포트 번호 필드
		![](https://i.imgur.com/d3FU8Ta.png)

>- 16비트
>- 0 ~ 65535
>- **잘 알려진 포트 번호** : 0 ~ 1023
#  UDP
## 이용 예시
- 스트리밍 (real-time)
- DNS
- SNMP -> Congestion Control이 어려울 때 네트워크 관리를 하는 응용프로그램
- HTTP/3 (QUIC)
	- HTTP는 기본적으로 신뢰적이기 때문에 application Layer 자체에서 혼잡, 오류 제어 제공
## 구조
![](https://i.imgur.com/rFrNLsC.png)
### 체크섬
- 오류 검출
- 데이터 전송 완료 시, UDP 세그먼트 안의 비트에 대한 변경사항 확인
![](https://i.imgur.com/3CHwGQh.png)
- 2-bit error시 발견할 수 없음

> ⚠️ 각 계층은 서로 관여하지 않는 것이 원칙이지만 오류 검출의 경우 여러 Layer에서 중복으로 나타남
> -> 일종의 **안정 장치**, 모든 Data Link Layer가 오류 제어를 한다는 보장이 없음

> 💡 UDP는 오류 검출 비트가 존재하지만 오류가 검출되어도 아무것도 하지 않음

# TCP
## 신뢰적 데이터 전송
![](https://i.imgur.com/5JfqI2W.png)
TCP 이용 시, Transport Layer에서 신뢰적인 채널이 사용됨
![](https://i.imgur.com/DbdnxsD.png)
실제로 Network Layer의 IP가 비신뢰적인 데이터 전송을 하고, TCP를 이용하는 Transport Layer에서 신뢰성 검사를 함

> 💡 신뢰할 수 없는 계층 위에서 신뢰할 수 있도록 해야함 -> ***rdt***
### 1️⃣ rdt 1.0
- **FSM**
	- Finite-State machine
	- 유한 상태 머신
	- 다른 상태로 전환될 수 있는 송신자, 수신자 간의 모델
![](https://i.imgur.com/Dy4d0un.png)
#### Sender
- `rdt_send(data)`: 상위 계층으로부터 데이터를 받음
- `make_pkt(data)` : 받은 데이터로부터 패킷 생성
- `udt_send(packet)` : 패킷을 채널로 송신
#### Receiver
- `rdt_rcv(packet)` : 하위 계층으로부터 패킷을 받음
- `extract(packet, data)` : 패킷으로부터 데이터 추출
- `deliver_data(data)` :  상위 계층으로 추출한 데이터 전달
#### Result
- 완벽히 신뢰적인 전송 -> 피드백 필요 없음
- Receiver가 Sender의 전송 즉시 받을 수 있다는 가정 -> Congestion, Flow Control 필요 없음
### 2️⃣ rdt 2.0
- 비트 에러가 있는 채널
#### 응답 종류
1. **ACK** 
	- receiver -> sender
	- pkt이 잘 도착함 (***OK***)
2. **NAK**
	- receiver -> sender
	- pkt이 잘 도착하지 않음 (***재요청***)
![](https://i.imgur.com/IrVDAjQ.png)
![](https://i.imgur.com/zwGBuz5.png)
#### Sender
- **Wait for Above** : 상위 계층으로부터 데이터가 전달되기를 기다림
	- 이 후 과정은 rdt 1.0과 동일
- **Wait for ACK or NAK** : Receiver로부터 응답 기다림
	- `rdt_rcv(rcvpkt) && isACK(rcvpkt)` : 가장 최근에 전송된 패킷이 정확하게 수신되었음을 알게 됨
	- `rdt_rcv(rcvpkt) && isNCK(rcvpkt)`: 마지막 패킷을 재전송하고 Receiver로부터 ACK | NCK를 기다림
> 💡 **전송 후 대기** : NAK 이후 다시 응답을 기다리는 과정에서 상위 계층에서 데이터를 보낼 수 없음 (`rdt_send`)
#### Receiver
- `corrupt(rcvpkt)`: 패킷이 수신되었지만 오류 검출
- `notcorrupt(rcvpkt)`:  패킷이 수신되었고 정확하게 오류 없음
### 3️⃣ rdt 2.1
- rdt 2.0에서 ACK, NAK 자체가 손상되는 가능성은 배제함 -> rdt 2.1은 이를 처리할 수 있는 방법을 제시

1. 수신자의 응답을 다시 전달해달라고 하는 방법. 이 또한 오류가 발생할 수 있음
2. 충분한 체크섬 비트 추가
3. Sender가 왜곡된 ACK, NAK을 수신 시 Receiver가 패킷 재송신 -> 중복 패킷이 전송되며 Sender는 이것이 새로운 데이터인지 중복 패킷인지 알 수 없게됨

- 새로운 필드를 추가하고, 필드 안에 순서 번호를 삽입하는 방식
- 재전송인지 여부를 결정할 때 순서 번호를 확인
![](https://i.imgur.com/sJ3lPgd.png)
![](https://i.imgur.com/vnF8fcJ.png)
#### Sender
- 재전송시 같은 순서 번호를 사용하고, 새로운 패킷 송신 시 다른 순서 번호를 사용함
#### Receiver
- 패킷 수신 후, 패킷이 올바르다면 다른 순서 번호로 패킷을 기다리고 올바르지 않다면 같은 순서번호에 대해서 재전송 패킷을 기다림
### 4️⃣ rdt 2.2
- NAK free -> ACK 메시지를 항상 수신하며 순서 번호로 판단
![](https://i.imgur.com/Oid9fC4.png)
![](https://i.imgur.com/7rCXxgm.png)
#### Sender
- 데이터 수신 시, 수신 번호 0 혹은 1로 패킷을 보낸 후, 0번에 대한 응답을 대기함
- 응답이 ACK이며 송신한 수신 번호와 다를 시, 재전송을 하며 같을 시 다른 수신 번호를 이용하여 패킷을 보냄
#### Receiver
- 수신 번호 0 혹은 1에 대한 대기 중, 수신 받은 패킷이 같은 수신 번호를 가지고 있을 시, ACK와 해당 수신 번호를 가지는 패킷을 보냄
- 만약 다른 수신 번호를 가질 시, ACK와 다른 수신 번호를 가지는 패킷을 보냄
### 5️⃣ rdt 3.0
- 오류 + 손실 가능성
- 손실 검측 + 회복 -> **카운트다운 타이머**
	1. 첫 번째 혹은 재전송 패킷이 전송된 시점에 타이머 시작
	2. 타이머 인터럽트에 반응
	3. 타이머 멈춤

> ⚠️ 너무 빠르게 재전송을 할 시 중복 패킷
> ![](https://i.imgur.com/lHENS8x.png)

![](https://i.imgur.com/ZKRWDbQ.png)
- `timeout` :  카운트다운 타이머에 의해 인터럽트 타이머가 발생할 경우, 재전송
## 파이프라인
![](https://i.imgur.com/yzoWKSP.png)
전송 후 대기 방식으로 동작하지만, 여러 패킷을 전송하도록 허용

- 여러 패킷이 전송되기 때문에 순서 번호의 범위가 커져야 함
- 손실 패킷, 오류 패킷에 대해서 회복해야 함
### 1️⃣ GBN
- **Go-Back-N** : N부터 반복
![](https://i.imgur.com/NpUADah.png)
- base + N 패킷은 base 패킷의 응답이 확인되어 `Already ACK'd`가 되기 전에는 사용될 수 없음
![](https://i.imgur.com/0RpBZt7.png)
#### Sender
- `nextseqnum < base + N`
	- 조건 만족 시, 패킷 송신과 동시에 카운트다운 타이머를 동작, 다음 전송 후보 패킷의 인덱스값 증가
	- 조건 위배 시, 데이터 버림
- `getacknum(rcvpkt)` : 누적 ACK로 지금까지 성공한 패킷 중 마지막 전송 패킷의 순서 번호를 가지고 있음
- `timeout` : 인터럽트 타이머 발생 시 송신자는 이전에 전송되었지만 아직 응답이 확인되지 않은 모든 패킷을 보냄
![](https://i.imgur.com/5u4HECk.png)
#### Receiver
- `make_pkt(expectedseqnum, ACK, checksum)`: 성공한 패킷의 누적 수신 번호를 전달함
- 순서가 맞지 않는 패킷은 모두 버림 -> 많은 재전송을 필요로 함
	![](https://i.imgur.com/CbRFFDh.png)
### 2️⃣ SR
- **Selective Repeat** : 응답을 받지 못한 패킷만 재전송
![](https://i.imgur.com/Zpmstx8.png)
- 순서가 맞지 않아도 먼저 도착한 패킷은 가지고 있음
	![](https://i.imgur.com/HsRkK7L.png)
	- 순서에 맞는 패킷이 도착 시, keep해두었던 패킷에 대한 응답을 모두 보냄
![](https://i.imgur.com/jtrZPGc.png)
Receiver의 윈도우는 a, b모두 같지만 a의 경우 두번째 pkt0은 재전송이고, b의 경우 두번째 pkt0은 새 전송임
Receiver의 윈도우는 모두 pkt0을 받아야 할 차례이기 때문에 이를 구분하지 못함
## 세그먼트
![](https://i.imgur.com/FeQB89g.png)
- **순서 번호**, **ACK 번호** : rdt를 통해 세그먼트를 보낼 때 송신자와 수신자에 의해 사용
	![](https://i.imgur.com/LkVMTNp.png)
	- 순서 번호 : 세그먼트에 있는 첫 번째 바이트의 바이트 스트림 번호
	- 응답 번호 : 호스트 A가 호스트 B로부터 기대하는 다음 바이트의 순서 번호로 확인된 번호가 누적됨
- **수신 윈도우** : flow control을 위해 사용, 수신자가 받아들이려는 바이트의 크기
- **헤더 길이**: TCP 옵션 필드의 경우 20비트를 차지하여 가변적이기 때문
- **옵션** : 송신자와 수신자가 최대 세그먼트 크기를 할당, 윈도 확장 요소, 타임스탬프 등
- **플래그**
	- ACK : 응답 값이 유용함을 가리킴
	- RST : 연결 강제 종료
	- SYN : 핸드셰이크 연결
	- FIN : TCP 연결 종료
	- PSH : 수신자가 데이터를 상위 계층에 즉시 전달해야 함 -> 평소에는 버퍼링
	- URG : 데이터를 '긴급'으로 표시 -> 우선 처리
## RTT 예측, 타임아웃
### RTT 예측
#### **SampleRTT**
- 세그먼트가 송신된 시간으로부터 그 세그먼트에 대한 ACK이 도착한 시간까지의 시간 길이
- 재전송한 세그먼트에 대해서는 측정 X
- 라우터에서의 혼잡, 종단 시스템에서의 부하 변화로 인해 세그먼트마다 다름 -> 불규칙적
#### **EstimatedRTT**
	$$EstimatedRTT = (1 - \alpha) \times EstimatedRTT + \alpha \times sampleRTT$$
- SampleRTT값의 가중 평균
- 예전 샘플보다 최근 샘플에 더 높은 가중치를 줌 -> 현재 혼잡을 반영
#### **DevRTT**
$$DevRTT = (1 - \beta) \times DevRTT + \beta \cdot |SampleRTT - EstimatedRTT|$$
- SampleRTT가 EstimatedRTT에서 얼마나 벗어났는지 예측
- SampleRTT가 같다면 DevRTT는 작아지며, 변화가 있다면 DevRTT는 커짐
### 타임아웃
- 주기는 EstimatedRTT보다 커야함 -> 작다면 재전송이 잦음
- 주기는 EstimatedRTT와 너무 차이나서는 안됨 -> 세그먼트 손실 시, 즉각적인 재전송이 이뤄지지 않음

1. 초기 TimeoutInterval = 1초
2. 타임아웃 발생 시, TimeoutInterval *= 2
3. EstimatedRTT 수정 시, $TimeoutInterval = EstimatedRTT + 4 \times DevRTT$
## 시나리오
### 1️⃣
![](https://i.imgur.com/vMQKXV7.png)

1. 92번 세그먼트에 대한 ACK 유실
2. 송신측(Host A)에서 TimeInterval 내에 ACK을 받지 못했으므로 92번 세그먼트 재전송
3. 수신측(Host B)는 92번 세그먼트가 재전송임을 알고 있으므로 해당 세그먼트를 버리고, ACK 전달

### 2️⃣
![](https://i.imgur.com/q8KwViR.png)

1. 92번 세그먼트에 대한 ACK이 TimeInterval 내에 도착하지 못하여 송신측(HostA)는 이를 재전송
2. 100번 세그먼트에 대한 ACK이 TimeInteval 내에 도착하지 못하여 송신측(HostA)는 이를 재전송 (할 예정)
3. 재전송한 92번 세그먼트에 대해 수신측(HostB) 수신 완료
4. 수신측(HostB)는 100번 세그먼트까지 잘 받았고, 이들이 재전송임을 알고 있음. ***다음 받아야 할 순서 번호***(=ACK)는 120이므로 ACK=120 전달

### 3️⃣
![](https://i.imgur.com/dtTh8Ng.png)

1. 92번 세그먼트에 대한 ACK이 TimeInterval 내에 도착하지 못하여 송신측(HostA)는 이를 재전송
2. 100번 세그먼트에 대한 ACK이 TimeInteval 내에 도착
3. 송신측(HostA)는 92번 세그먼트에 대한 ACK을 받지 못했지만 ACK=120에 의해 119번 바이트까지 모두 수신했음을 알기에 더 이상 재전송 X
## ACK
1. 순서가 맞는 세그먼트 도착 -> 다음 세그먼트를 500ms 까지 기다리며 순서에 맞는 다음 세그먼트가 있을 시 하나의 ACK 전송
2. 순서가 바뀐 세그먼트 도착 -> 즉시 ACK을 보냄
## 빠른 재전송
세그먼트 타이머가 만료되기 전에 손실 세그먼트를 재전송
![](https://i.imgur.com/obnXoAz.png)
## Flow Control
속도를 일치시키는 서비스

- **receive window** 유지 -> 가용 버퍼 공간이 얼마나 되는지 송신자에게 알려줌
### 사례
Host A가 Host B에게 큰 파일 전송

#### Host B
**RcvBuffer** : Host B의 수신 버퍼 크기
**LastByteRead** : 버퍼로부터 읽힌 데이터 스트림의 마지막 바이트의 번호
**LastByteRcvd** : RcvBuffer에 저장된 데이터 스트림의 마지막 바이트 번호

$$LastByteRcvd - LastByteRead \le RcvBuffer$$
오버플로를 허용하지 않음
$$rwnd = RcvBuffer - [LastByteRcvd - LastByteRead]$$
**rwnd** : 버퍼의 여유 공간 -> 동적
#### Host A
**LastByteSent** : 마지막으로 보낸 바이트의 번호
**LastByteAcked** : 마지막으로 정상 응답을 받은 바이트의 번호
전송 확인 응답이 안 된 데이터 양 = $LastByteSent - LastByteAcked$

$$LastByteSent - LastByteAcked \le rwnd$$
HostB가 오버플로우 되지 않음을 보장해야 함
#### 처리 방법
1. $rwnd = 0$이 됨
2. HostB(Receiver)가 HostA(Sender)가 보낸 데이터를 차단하고, rwnd가 0임을 TCP 헤더에 포함하여 보냄
3. HostA는 rwnd값을 확인한 후, 1바이트의 데이터를 지속해서 보냄
4. HostB의 receive window가 비워지면 ACK에 0이 아닌 rwnd값이 전해짐
## 핸드셰이킹
### 1️⃣ 3-way Handshaking
연결 수립 핸드셰이킹

![](https://i.imgur.com/01B3HmQ.png)
1. Client -> Server : 특별한 TCP 세그먼트 송신
	- 애플리케이션 계층 데이터 포함 X
	- SYN 비트 1로 설정
	- seq로 `client_isn` 순서 번호 주입
2. Server가 SYNACK 세그먼트 송신
	- Client의 세그먼트에서 SYN 비트 추출
	- SYN 비트 1로 설정
	- ACK은 `client_isn + 1`로 설정
	- seq은 `server_isn` 설정 
3. 클라이언트가 버퍼와 변수 할당
	- ACK에 `server_isn + 1` 설정
	- 연결이 설정되었으므로 SYN 비트 0으로 설정

>**SYN 플러드 공격**
>무수한 SYN 수신으로 반쪽짜리 연결을 수립하여 서버의 자원을 고갈시키는 DoS 공격 중 하나
>***SYN쿠키***를 통해 반쪽 연결을 수립하지 않고, 해시 번호를 만들어두었다가 SYNACK 세그먼트와 비교를 한 후, 연결 혹은 버림
### 2️⃣ 2-way handshaking
연결 종료 핸드셰이킹

![](https://i.imgur.com/rAU4TWX.png)
1. Client -> Server : FIN 비트 1로 설정된 TCP 세그먼트 송신
2. Server -> Client : 확인 응답 세그먼트 송신
3. Server -> Client : FIN 비트 1로 설정된 TCP 세그먼트 송신
4. Client -> Server : 확인 응답 세그먼트 송신
## Congestion Control
### 혼잡 원인, 비용
#### 1️⃣ 시나리오 1 : 2개의 송신자, 무한 버퍼 라우터
![](https://i.imgur.com/xvQsraL.png)

**Host A, B**
![](https://i.imgur.com/VTxTQpF.png)
- 2개의 송신자이므로 최대 라우터의 용량 R에서 각자 R/2만큼 사용 가능

**Host C, D**
![](https://i.imgur.com/RepmG2V.png)
- 2개의 송신자가 있으므로 항상 최대 R/2만큼 이용할 수 있음
- 링크 용량에 근접함에 따라 큐잉 지연이 지수적으로 커짐
#### 2️⃣ 시나리오 2 : 2개의 송신자, 유한 버퍼 라우터
![](https://i.imgur.com/Vj9pu1G.png)
vs 시나리오 1
- 버퍼가 가득 찼을 경우 도착하는 패킷은 버려짐
- 신뢰적 -> 패킷이 라우터에서 버려질 경우 재전송

1) Host A, B가 버퍼의 가용 공간을 알 수 있음 (비현실적)
	![](https://i.imgur.com/Tm3wdSQ.png)
	패킷이 어떠한 경우에도 손실되지 않음
2) 패킷이 확실히 손실될 경우에만 재전송 -> 큰 타임아웃
	![](https://i.imgur.com/BIemjVt.png)
	재전송되는 패킷 = $R/2 - R/3$ = $R/6$
3) 큐에서 지연되고 있는 패킷을 재전송
	![](https://i.imgur.com/YjRzxBE.png)
	원래의 패킷 + 재전송 패킷 모두 수신자에게 전달
	평균적으로 두 번에 한 번씩 재전송되기 때문에 처리량은 $R/4$
#### 3️⃣ 시나리오 3 : 4개의 송신자, 유한 버퍼 라우터, 멀티 홉
![](https://i.imgur.com/R2nY8ds.png)
**A -> C 연결**
- D -> B와 R1 공유
- B -> D와 R2 공유

A -> C 전송량이 낮아지면, R1에서 D -> B의 전송량이 커질 수 있음

> 💡 멀티홉에서 처리량의 트레이드가 일어남
## 혼잡 알림 방식
### 1️⃣ 종단 간의 혼잡 제어
- 네트워크 계층의 직접적인 지원 없음
- 종단 시스템이 혼잡을 추측
- TCP 세그먼트 손실을 기준으로 윈도 크기를 조절함
### 2️⃣ 네트워크 지원 혼잡 제어
![](https://i.imgur.com/AKUhxe6.png)
- 송신자, 수신자에게 직접적인 피드백 제공
- 라우터가 자신의 출력 링크 가용량을 송신자에게 명확히 알림
- 해당 알림을 받은 수신자는 이를 송신자에게 다시 알림 -> 왕복 시간 소요
## 전통적 Congestion Control
### 1️⃣ TCP 송신자는 트래픽 전송률을 어떻게 제한하는가?
- **Congestion Window** 추적
$$ LastByteSent - LastByteAcked \le min(cwnd, rwnd)$$

**자체 클로킹**
- 확인응답을 congestion window 크기 증가를 유발하는 트리거로 사용
- ACK이 빠른 속도로 온다면 congestion window는 빠르게 증가, ACK이 느린 속도로 온다면 congestion window는 느리게 증가
### 2️⃣ TCP 송신자가 수신자와 그 사이의 혼잡을 어떻게 감지하는가?
- 타임아웃
- 3개의 중복된 ACK 수신
### 3️⃣ 혼잡이 발생하였다면, 송신율 변화를 위해 어떤 알고리즘을 써야하는가?
![](https://i.imgur.com/Jjr7xq0.png)
#### 슬로 스타트
![](https://i.imgur.com/T8eNEoe.png)
cwnd를 1MSS에서 시작하여 ACK를 받을 때마다 MSS 만큼 증가
ssthresh(slow start threshold)의 값을 cwnd/2로 설정 (혼잡이 검출된 시점의 반절)
지수적으로 증가

**종료**
- 타임아웃으로 표시되는 손실 이벤트가 발생할 경우
	- cwnd를 1로 설정하고 새롭게 시작
- cwnd가 ssthresh에 도달하였을 때
	- ***혼잡 회피 모드*** 돌입
- 3개의 중복 ACK이 발견되었을 때
	- ***빠른 회복 상태*** 돌입
#### 혼잡 회피
혼잡 회피에 돌입했을 때 cwnd = 마지막으로 혼잡이 발견되었을 때 값의 반절
RTT마다 하나의 MSS만큼 증가

**종료**
- 타임아웃이 발생했을 때
	- 슬로 스타터와 같이 동작
	-  cwnd를 1로 설정
	- $ssthresh = cwnd / 2$
- ACK이 발생했을 때 -> 타임아웃과 다르게 송신자는 계속해서 ACK을 보내기 때문에 덜 과감해야함
	- cwnd를 cwnd / 2로 설정
	- ssthresh를 업데이트 한 cwnd / 2로 설정
	- ***빠른 회복 상태*** 돌입
#### 빠른 회복
- 모든 중복된 ACK에 대해 1MSS씩 증가

**종료**
- 타임아웃이 발생했을 때
	- 슬로 스타터, 혼잡 회피와 같이 동작
	- cwnd를 1로 설정
	- $ssthresh = cwnd / 2$
	- 이후, 슬로 스타터로 전이

> 💡 TCP 혼잡 제어는 **AIMD** 혼잡 제어라고 불림
> **AIMD**
> ![[Pasted image 20240401224344.png]]
> - additive-increase, multiplicative decrease
> - RTT마다 1MSS 씩 cwnd의 선형 증가, 3개의 중복 ACK 이벤트에서 cwnd의 절반화

> **QUBIC**
> ![[Pasted image 20240401224815.png]]
> - AIMD는 혼잡 제어에서 너무나 조심스럽게 증가함
> - 세제곱함수로 증가 -> 이전 혼잡 시기와 가까울 땐 적게, 멀 땐 많이 증가
> - 정체 수준이 크게 변경된 경우 효과적

## 혼잡 추정 방식
### 1️⃣ 명시적 혼잡 알림
### 2️⃣ 지연 기반 혼잡 제어
## 공평성