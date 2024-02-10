# CPU burst, IO burst
**burst** : 짧은 시간동안 집중적으로 발생하는 활동이나 사건
**CPU burst** : CPU를 사용하여 계산 작업을 수행하는 기간
**IO burst** : 입출력 작업을 수행하는 기간

![](https://i.imgur.com/JW8LAxH.png)

## CPU burst 길이
![](https://i.imgur.com/mFbbjcx.png)
주로 짧은 시간을 가짐 -> I/O burst외에도 다양한 이유가 있음

1️⃣ **시분할, [[Context Switch]]**
- 프로세스가 독점하는 것을 방지하기 때문에 CPU burst가 짧아짐
2️⃣ **[[시스템 콜]], [[인터럽트]]**
- 운영체제 서비스를 요청하거나 외부 이벤트에 반응할 때 CPU를 다른 작업에 할당

# CPU Scheduling
## CPU 스케줄링 발생 조건
1️⃣ [[프로세스]]의 상태가 `running` -> `waiting`으로 변경
	- IO request의 결과가 필요할 때
	- 자식 프로세스의 중단을 위해 `wait()`을 호출할 때
2️⃣ 프로세스의 상태가 `running` -> `ready`로 변경
	- 비 IO 하드웨어 인터럽트 (타이머 인터럽트)
3️⃣ 프로세스의 상태가 `waiting` -> `ready`로 변경
	- IO 완료
4️⃣ 프로세스의 상태가 `terminated`로 변경
	- 프로세스 종료

### 비선점형 스케줄링
1️⃣ : 프로세스가 CPU를 자발적으로 양도
4️⃣ : 자발적으로 CPU를 마치고 프로세스가 시스템에서 완전히 제거

> 💡 프로세스가 CPU를 선점하지 않음

### 선점형 스케줄링
2️⃣ : 타이머 인터럽트 등을 사용하여 운영체제가 프로세스를 선점하여 상태 변경
3️⃣ : I/O 작업 완료 인터럽트가 발생할 때 운영체제가 프로세스를 `ready` 상태로 변경

> 💡 운영체제가 선점을 통해 프로세스를 관리

⚠ Data Race 발생 가능성 있음 ⚠️
다른 곳에서 읽을 가능성이 있는 어떤 메모리 위치에 쓰기 작업을 하는 것
![](https://i.imgur.com/h4rdYQl.png)
-> 600ms에 죽어야 할 철학자의 죽음을 monitoring하지 못해서 다른 프로세스는 철학자가 살아있다고 생각해서 발생
![](https://i.imgur.com/PvgyiUB.png)

시스템 콜로 컨텍스트 스위칭 도중 데이터가 변경되면 위와 같이 data race가 감지될 수 있음
-> I/O 요청을 기다리는 조건으로 mutex, semaphore 등이 필요

## Dispatcher
### 역할
1. 한 프로세스에서 다른 프로세스로 Context Switch
2. user mode로 스위치
3. 프로그램을 다시 시작하기 위해 유저 프로그램의 적절한 위치로 점프 (PCB 정보를 저장하고 불러옴)

## 스케줄링 기준
1️⃣ **CPU 사용률**
: 최대한 많이 사용할 수 있도록 함
40~90% 사용
2️⃣ **Throughput**
: 시간 단위 당 완료되는 프로세스의 수
3️⃣ **Turnaround time**
: 해당 프로세스를 실행하는데 걸리는 시간
ready queue 대기 시간 + CPU 실행 시간 + I/O 시간
4️⃣ **Waiting time**
: 프로세스가 ready queue에서 대기하는데 걸리는 시간
5️⃣ **Response time**
: 요청 후 첫 번째 응답이 생성될 때까지의 시간
응답을 시작하는데 걸리는 시간

CPU 사용률, Throughput 🔺
Turnaround time, Waiting time, Response time 🔻

## 스케줄링 알고리즘
### 1️⃣ [[FCFS]]
### 2️⃣ [[SJF]]
### 3️⃣ [[RR]]
### 4️⃣ [[Priority Scheduling]]
### 5️⃣ [[Multilevel Queue Scheduling]]
### 6️⃣ [[Multilevel Feedback Queue Scheduling]]
# Multi-Processor Scheduling
## Multi-Processor란?
1️⃣ Multicore CPU
2️⃣ Multithreaded core
3️⃣ [[NUMA system]]
4️⃣ [[HMP]]
## 방법
### 1️⃣ Asymmetric multiprocessing
**원리**
- 단일 프로세서 (마스터 서버)가 모든 스케줄링, 입출력 등을 처리
- 다른 프로세서는 사용자 코드만 실행
**장점**
- 하나의 코어만 데이터 구조에 엑세스하기 때문에 간단하고 데이터 공유 감소
**단점**
- 마스터 서버가 전체 시스템 성능을 저하시킬 수 있음
### 2️⃣ Symmetric multiprocessing
**원리**
- 각 프로세서가 자체 스케줄링 수행
**방법**
1. 모든 스레드가 동일한 ready queue에 존재
	![](https://i.imgur.com/q64X7XY.png)
	데이터 레이스가 발생할 수 있으므로 lock이 필요하지만 원격 접근 시 병목 발생
2. 각각의  프로세서가 자신만의 스레드 queue를 가짐
	![](https://i.imgur.com/yXMc4dr.png)
	일반적인 SMP
	공유 메모리에 대한 문제 해결
