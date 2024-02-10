queue가 여러 개인 스케줄링

1️⃣ **Priority + RR** scheduling
![](https://i.imgur.com/e8bwytN.png)
여러 개의 priority queue가 있고, 전체적인 queue는 RR 스케줄링 사용

2️⃣ **Process Type**
![](https://i.imgur.com/cwbaZCN.png)
foreground에서 동작하는 process가 높은 우선순위를 지니고,
background에서 동작하는 process가 낮은 우선순위를 지님

>💡 foreground process가 background process보다 더 빠른 응답 시간을 요구함
>큐들 사이에서의 스케줄링은 고정 우선순위 선점형 스케줄링으로 구현됨 -> 실시간 프로세스가 더 우세

### queue 내부 스케줄링
1️⃣ **foreground**
- [[RR]]
- [[Priority Scheduling]]
> 💡 **선점형 스케줄링** : 빠른 응답시간이 중요하기 때문

2️⃣ **background**
- [[FCFS]]
- [[SJF]]
> 💡 **비선점형 스케줄링** : 처리 속도보다는 자원 활용도가 더 중요하고, 긴 작업을 처리하는 데 적합하기 때문

### queue간 스케줄링
1️⃣ **우선순위**
	interactive, system process queue가 비어있지 않으면 batch process는 실행되지 않음
	batch process 실행 중 우선순위가 높은 process가 들어오면 해당 프로세스가 CPU 선점
2️⃣ **time slice**
	각 큐가 CPU 시간의 일정 비율을 할당
	ex) foreground 큐에 80%, background 큐에 20% 할당
	
## 단점
process가 queue에 할당될 때 foreground, background에 영구적으로 할당
-> 유연하지 않음
-> [[Multilevel Feedback Queue Scheduling]]으로 해결