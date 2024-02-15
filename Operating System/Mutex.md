[[임계구역 문제]]를 해결하기 위한 가장 간단한 소프트웨어 도구
**MUTual EXclusion**의 줄임말
하나의 스레드만을 제어
## 역할
1. 임계구역 보호
2. race condition 방지
## 원리
![w2anhYF.png](https://i.imgur.com/w2anhYF.png)
`boolean available` : lock이 가능한지 불가능한지 여부
- 이미 lock이 걸려있을 경우 release할 때 가능해짐

1️⃣ critical section에 들어가기 전에 lock -> `acquire()`
```c
acquire() {
	while (!available)
		; /* busy wait */
	available = false;
}
```
2️⃣ critical section을 나올 때 unlock -> `release()`
```c
release() {
	available = true;
}
```
## 단점
- busy wait 필요
	: 한 프로세스가 자신의 임계 구역에 있으면, 자신의 임계 구역에 진입하려는 다른 프로세스가 진입 코드를 계속 반복하는 것
- [[데드락]]의 발원지