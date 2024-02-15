[[임계구역 문제]]를 해결하기 위한 정교한 소프트웨어 도구
여러 스레드가 공유 자원에 동시에 접근할 수 있음
## 종류
1️⃣ **이진 세마포어**
- 0 또는 1의 값만 가질 수 있는 세마포어
- [[Mutex]]와 유사
2️⃣ **카운팅 세마포어**
- 0 이상의 값을 가질 수 있는 세마포어
- 한 번에 여러 스레드가 자원을 사용할 수 있도록 허용
## 원리
1️⃣ critical section에 들어가기 전에 허가 받음 -> `wait()`
```c
wait(S) {
	while (S <= 0)
		; // busy wait
	S--;
}
```
2️⃣ critical section에서 나올 때 세마포어 해제 -> `signal()`
```c
signal(S) {
	S++;
}
```
## 단점
- [[데드락]]의 발원지
- busy wait
## busy wait 해결 방안
**일시 중단**
- `waiting` 상태로 전환하고 제어권이 CPU 스케줄러에게 넘어가서 스케줄러가 다음 실행할 프로세스를 선택
	```c
	typedef struct {
		// 양수일 경우 공유하는 스레드 개수, 음수일 경우 대기 중인 스레드 개수
		int value;
		struct process *list; //waiting 상태인 프로세스 목록
	} semaphore;
	
	wait(Semaphore *S) {
		S->value--;
		if (S->value < 0) {
			add this process to S->list;
			sleep();
		}
	}
	```
- 다른 프로세스가 `signal()` 동작을 실행할 때 다시 실행 -> `wakeup()` 동작을 통해 `ready`상태로 변경
	```c
	signal(Semaphore *S) {
		S->value++;
		if (S->value <= 0) {
			remove a process P from S->list;
			wakeup(P);
		}
	}
	```
	> 💡 `sleep()`, `wakeup()`은 OS가 기본 [[시스템 콜]]로 제공
- 프로세스가 ready queue에 배치
- CPU 스케줄링 알고리즘에 따라 바로 `running` 상태가 될 수도 안될 수도 있음