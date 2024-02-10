[[Multilevel Queue Scheduling]]의 단점 극복
-> short CPU burst를 가진 프로세스의 우선순위를 높게 둠 (I/O bound, interactive processes)
-> 더 낮은 priority queue에서 오래 기다리는 프로세스를 더 높은 priority queue로 이동
-> **기아 방지**
## 원리
1️⃣ queue 0의 모든 프로세스 실행
2️⃣ queue 0에서 끝나지 않은 process들을 queue 1에 이동
3️⃣ queue 0가 비어있을 경우 queue 1 실행
4️⃣ queue 2 또한 queue 0, queue 1가 비어있을 경우 실행

![](https://i.imgur.com/z8bpbYS.png)
1. queue 0에서 1 quantum(8ms)에 끝내지 못했다면 queue 1로 이동
	- CPU burst가 8ms내인 프로세스는 종료 (우선순위 높음)
2. queue 1에서 queue 0가 비어 있을 경우, 1 quantum(16ms)을 수행하며 이를 끝내지 못했다면 queue 2로 이동
	- CPU burst가 24ms내인 프로세스는 종료
3. queue 2에서 queue 0, queue 1이 모두 비어있을 경우 FCFS 스케줄링
4. 기아를 방지하기 위해 queue 0, 1에서 너무 오래 기다리고 있는 프로세스를 각각 queue 1, 2로 이동
## 매개변수
- 큐의 개수
- 각 큐의 스케줄링 알고리즘
- 프로세스를 우선 순위가 높은 큐로 업그레이드할 시기를 결정하는데 사용할 방법
- 프로세스를 우선 순위가 낮은 큐로 다운그레이드할 시기를 결정하는데 사용할 방법
- 프로세스가 서비스를 필요로 할 때 들어갈 큐를 결정하는데 사용할 방법
## 단점
모든 매개변수에 대한 값을 선택해야 하기 때문에 가장 복잡함