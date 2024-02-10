**First-Come First-Serve** : 선입선출
비선점형
FIFO 큐를 사용하여 관리
![](https://i.imgur.com/7qnm1cD.png)
### 원리
1️⃣ 프로세스가 Ready Queue에 들어가면 프로세스의 PCB가 대기열의 꼬리 부분에 연결
2️⃣ CPU가 여유가 생기면 대기열의 맨 앞에 있는 프로세스에 CPU 할당
3️⃣ 실행 중인 프로세스는 대기열에서 삭제

![](https://i.imgur.com/o2zPKGp.png)

처리 순서가 P2 -> P3 -> P1일 경우
![](https://i.imgur.com/YnNOg23.png)P1 대기시간 : 6
P2 대기시간 : 0	
P3 대기시간 : 3
> 💡총 대기시간 : (6 + 0 + 3) / 3 = 3
### 장점
이해하기 쉽고 간단
### 단점
비선점형 -> 대화형 시스템에서 문제
평균 대기 시간이 긴 경우가 많음
CPU burst 시간에 의해 크게 달라짐
 
> 위 예시에서 처리순서가 P1 -> P2 -> P3일 경우
>	![](https://i.imgur.com/w9boZZB.png)
>	P1 대기시간 : 0
>	P2 대기시간 : 24
>	P3 대기시간 : 27
>	💡 총 대기시간 : (0 + 24 + 27) / 3 = 17