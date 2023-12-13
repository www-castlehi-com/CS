# 정의
하나의 통신채널을 통해서 둘 이상의 데이터를 전송하는데 사용되는 기술
다수의 프로세스를 생성하지 않고도 여러 개의 클라이언트에게 서비스를 제공할 수 있는 기술

# 종류![](https://i.imgur.com/by9i2u4.png)

# I/O multiplexing
## 1. Select
- fd를 배열에 넣고 하나하나 순차 검색하는 가장 원초적인 방식
- 시간 복잡도 O(n)
- fd 수가 최대 1024개로 제한
- 무한루프
- 매번 fd_set을 저장해야하고, 한 fd에서 데이터가 오면 기존 fd_set을 모두 변경해야하기 때문에 context switching이 빈번

## 2. Poll
- 무한 개의 fd를 검사 가능
- select는 최대 fd까지 loop를 돌지만 poll은 실제 fd개수 (nfds) 까지만 loop를 돌 수 있음
- 시간 복잡도 O(n)

## 3. epoll
- linux에서만 사용 가능
- poll처럼 fd를 무한 개로 검사 가능
- select, poll과 달리 커널에서 fd의 변화를 감지하고 직접 통지 (fd_set을 돌기 위해 루프를 돌 필요 없음)
- fd의 개수가 아닌 목록을 반환하기 때문에 대상 파일을 추가 탐색할 일 없음

## 4. Kqueue
- FreeBSD에서의 epoll
- 커널에 이벤트를 저장할 queue를 생각하면, I/O 이벤트가 queue에 쌓이고 사용자가 이를 직접 polling 하는 방식
- select, poll처럼 event가 발생한 fd를 찾기 위한 작업이 필요 없음
- kevent()는 kqueue에 특정 event를 등록하고 사용자에게 반환하는 system call
- kqueue()로 할당받은 공간을 kevent()로 관리
- 