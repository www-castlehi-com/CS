# 프로세스 간 통신 (Interprocess communication)
- 프로세스 간 데이터를 교환하는 기법
## 종류
### 1. 공유 메모리
![](https://i.imgur.com/qi7t40u.png)
- 메모리 충돌을 고려해야하기 때문에 구현이 어려움
- 공유 메모리를 만들 때에만 시스템 콜 호출
### 2. 메시지 전달
![](https://i.imgur.com/VppSFN8.png)
- 시스템 콜을 사용하여 구현되므로 느림