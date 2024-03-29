## 등장 배경
[[Mutex]], [[Semaphore]]를 잘못 사용할 경우 감지하기 어려운 타이밍 오류가 발생할 수 있음

1. `wait()`, `signal()`의 순서가 바뀌었을 때
	```c
	signal(mutex);
		...
		critical section
		...
	wait(mutex);
	```
	: 여러 프로세스가 동시에 실행되어 mutual-exclusion 위반
2. `wait()`이 반복 호출될 경우
	```c
	wait(mutex);
		...
		critical section
		...
	wait(mutex);
	```
	: 영구적으로 대기
3. `wait()`, `signal()` 생략 혹은 둘 다 생략할 경우
	: mutual-exclusion이 위반되거나 프로세스가 영구적으로 차단
	
위와 같은 동기화 도구를 통합하는 요소
## 구성
![](https://i.imgur.com/XyosSvS.png)

```c
monitor monitor_name
{
	/* shared variable declarations */
	function P1 (. . .) {
		. . .
	}

	function P2 (. . .) {
		. . .
	}
	.
	.
	.
	function Pn (. . .) {
		. . .
	}

	initialization_code (. . .) {
		. . .
	}
}
```
### 조건 변수
- 공유 자원에 접근하려는 프로세스간의 실행 순서를 제공
#### 선언
```c
condition x, y;
```
#### 사용
1️⃣ 조건이 만족될 때까지 대기 상태로 전환
```c
x.wait();
```
1. **대기 상태 전환** : 조건 변수 `x`와 관련된 조건이 만족될 때까지 대기
2. **모니터, 뮤텍스 해제** : 스레드가 잡고 있던 모니터 또는 뮤텍스, 세마포어를 자동으로 해제
   -> 다른 스레드가 임계 구역에 진입하여 작업 수행 가능

2️⃣ 조건 변수를 기다리는 `waiting` 중인 스레드 하나를 깨움
```c
x.signal();
```
1. **스레드 깨우기** : 하나의 스레드를 임의로 선택하여 깨움
2. **모니터, 뮤텍스 획득** : 임계 구역에 진입하기 위해 다시 모니터 또는 뮤텍스 획득

> 💡해당 조건 변수를 보호하는 모니터 또는 뮤텍스 내에서 호출해야함

## 실행 순서
조건에서 여러 프로세스가 중단되고 `x.signal()`에 의해 재개될 때 프로세스를 결정하는 방법
1️⃣ **FCFS**
2️⃣ **조건부 대기**
```c
x.wait(c);
```
- **c** : priority number
```c
x.signal();
```
- c가 가장 낮은 프로세스부터 시작
