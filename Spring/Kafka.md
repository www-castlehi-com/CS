# 메시지란?
- 커뮤니케이션의 기본 단위
- 서비스 간에 전달되는 모든 데이터
## 예시
1. HTTP 요청
2. Apache Kafka 메시지
## 처리 방식
1. 동기 처리
2. 비동기 처리
# 메시지 큐
메시지를 교환할 때 사용하는 기술
## 구성
1. **Producer** : 정보 제공자
2. **Consumer** : 정보 소비자
3. **Message Queue** : Producer의 메시지를 임시 저장하고 Consumer에게 제공하는 곳
## 종류
### 1️⃣ 메시지 Broker
![](https://i.imgur.com/pt10pak.png)
- Producer로부터 메시지를 수신하고, 메시지를 처리하여 Consumer에게 제공하는 중간자 역할
- 소비자와 메시지 브로커의 결합력이 높아지게 됨
- 서비스 트래픽이 증가할 경우 수평적 확장 어려움

1. Producer가 메시지 생성
2. 메시지 브로커 내에서 이 메시지를 어느 큐에 발송할지 `exchange`를 통해 결정
3. 선택된 큐에 메시지 enqueue
4. Consumer에게 큐에 있는 메시지 전달하며 큐에서 삭제
### 2️⃣ 이벤트 Broker
![](https://i.imgur.com/SXkcmfh.png)
- 이벤트 또는 메시지라고 불리는 레코드를 하나만 보관하고 인덱스를 통해 개별 액세스를 관리
- 메시지 브로커, 이벤트 브로커 모두의 이벤트 수신

1. Publisher가 데이터 즉, topic을 생성하면 event streamer에 저장하고, 이벤트의 레코드 로그를 순서대로 기록
2. 해당 topic을 구독한 Consumer에게 전달
3. Consumer에게 전달한 후에도 event stream에서는 topic을 유지함

- topic을 유지하기 때문에 장애가 발생하였을 시 복구 가능
- 메시지 브로커보다 더 많은 양의 데이터 처리 가능 -> 대용량 데이터 처리에 좋음
	- 메시지 브로커의 point-to-point, request-reply는 단 하나의 수신자에 의해서 처리되지만, 이벤트 브로커는 여러 수신자가 처리 가능
	- 메시지를 보존하기 때문에 필요한 경우 같은 스트림을 재처리하거나 다른 분석에 이용
	- 파티션을 여러 서버에 분산 저장되기 때문에 수평적 확장 용이

> 💡 이벤트 브로커 -> 메시지 브로커 ⭕, 메시지 브로커 -> 이벤트 브로커 ❌
# Apache Kafka
- 대규모 데이터 스트림을 처리하기 위해 설계된 분산 스트리밍 플랫폼
- 이벤트 브로커의 한 종류
- 기본 단위 : **메시지**(record) -> topic에 저장
## 배경
소셜 네트워크 서비스 링크드인에서 개발
![](https://i.imgur.com/jcB6UGT.png)
Kafka 개발 전, App&Service가 Relational Database에 end-to-end연결됨
시스템 복잡도가 증가하며 db -app 간의 파이프라인 확장이 어려워짐

목표 : **모든 시스템으로 데이터를 전송할 수 있고, 실시간 처리도 가능하며, 확장이 용이한 시스템을 만들자**

![](https://i.imgur.com/gIyS8rR.png)
Kafka 개발 후, 문제점 완화
- 모든 이벤트/데이터의 흐름을 중앙에서 관리
- 새로운 서비스/시스템이 추가될 경우 Kafka가 제공하는 표준 포맷으로 연결하면 되므로 확장성 증가
- 개발자는 각 서비스의 연결이 아닌, 비즈니스 로직에만 집중 가능
## 기본 구성
- **Key** : 메시지 분류 시 사용, 파티셔닝 기준이 될 수 있음
- **Value** : 실제 메시지 데이터 포함
- **Timestamp** : 메시지가 생성되거나 Kafka에 의해 수신된 시간을 나타냄. 시간 불일치 문제를 다루거나 메시지 순서를 결정하는데 사용

![](https://i.imgur.com/fDrSAP8.png)
- **Topic** : 카테고리로서 하나 이상의 메시지를 포함하며 일반적으로 특정 유형의 데이터나 이벤트를 나타냄
- **Partition** : 확장성과 병렬 처리를 위함
## 구조
![](https://i.imgur.com/3kVxTQQ.png)
**Pub-Sub**
- 메시지를 보내는 역할(Pub)과 받는 역할(Sub)이 완벽하게 분리
- 느슨한 결합 -> 한쪽 시스템에서 문제가 발생하더라도 서로 의존성이 없음
- Consumer 서버가 추가되더라도 Kafka에 보내면 되기 때문에 서버 확장에 대한 부담 감소
- 하나의 topic에 여러 Producer 또는 Consumer들이 접근 가능
## 특징
- 디스크에 메시지 저장
	- TCP/IP 통신을 통해 바로 디스크에 저장
	- 서버가 다운되더라도 디스크에 적재
	- 데이터의 영속성이 보장됨을 의미
- 분산 환경 특화
	- 하나의 Kafka Cluster에 3대의 브로커부터 시작해 확장 가능
- Consumer Group
	- 하나의 Consumer는 하나 이상의 Consumer Group에 속해 있어야 함
	- 그룹 단위로 데이터를 읽으며 하나의 Group에 한 번의 데이터 전달만 하면 됨
## Zookeeper
![](https://i.imgur.com/lCUt1Yn.png)
- Kafka Cluster 상태를 관리하는 분산 코디네이터
- Kafka Cluster 내의 broker 리스트를 갖고 관리하며, Kafka는 Zookeeper 없이 실행 불가능 -> KRaft를 도입하며 의존성 줄이는 중
- 쓰기를 관장하는 **리더 서버** 1개와 읽기를 관장하는 **팔로워 서버**들로 구성