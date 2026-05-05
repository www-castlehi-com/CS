## 1️⃣ List
### 1.1 개념
![](https://i.imgur.com/Uveqs2J.png)
순서대로 데이터를 저장하고 양쪽으로 데이터를 삽입/삭제할 수 있는 자료구조이다.
실제 서비스에서는 스택(stack)이나 큐(queue)로 자주 사용된다.
### 1.2 명령어
#### 삽입
```shell
# RPUSH | LPUSH [key] [value]
RPUSH my_list 1
LPUSH my_list 2
```
리스트의 양쪽(RPUSH : 오른쪽 / LPUSH : 왼쪽)에 데이터를 추가한다.
동시에 key를 생성한다.
#### 조회
```shell
# LRANGE [key] [start index] [end index]
LRANGE my_list 0 -1
```
인덱스 범위를 활용해 리스트의 모든 데이터를 조회한다.
end index가 -1인 경우 리스트의 모든 데이터를 조회하게 된다.
```shell
# LLEN [key]
LLEN my_list
```
리스트에서 데이터의 총 개수를 조회한다.
### 삭제
```shell
# LPOP | RPOP [key]
RPOP my_list
LPOP my_list
```
리스트의 양쪽(RPOP : 오른쪽 / LPOP : 왼쪽)에서 데이터를 꺼내온다.
## 2️⃣ String
### 1.1 명령어 옵션
```
# SET [key] [value] NX
SET hobby golf NX
```
지정한 키가 없을 때에만 데이터를 저장한다.
## 3️⃣ Sorted Set
### 1.1 개념
![](https://i.imgur.com/TcE7bxg.png)
score(점수)를 기준으로 자동 정렬되는 중복 없는 자료구조이다.
랭킹 시스템, 실시간 인기 데이터 관리 등 순서와 가중치가 중요한 경우에 주로 활용된다.
### 1.2 명령어
#### 삽입
```shell
# ZADD [key] [score] [member]
ZADD ranking:2025 5 jihoon
ZADD ranking:2025 9 jaeseong
ZADD ranking:2025 2 yeonwoo
```
#### 조회
```shell
# ZRANGE [key] [start index] [end index] WITHSCORES
ZRANGE ranking:2025 0 1 WITHSCORES 
ZRANGE ranking:2025 0 -1 WITHSCORES
ZRANGE ranking:2025 0 -1 WITHSCORES REV
```
`WITHSCORES` 옵션은 score와 함께 Member를 반환한다.
score는 오름차순 정렬이 기본이며, `REV` 옵션의 경우 내림차순으로 조회할 수 있다.
```shell
# ZRANGE [key] [start index] [end index]
ZRANGE ranking:2025 0 1 
ZRANGE ranking:2025 0 -1
ZRANGE ranking:2025 0 -1 REV
```
`WITHSCORES` 옵션을 제거할 경우 score 없이 Member만 반환한다.
#### 수정
```shell
# ZINCRBY [key] [increment] [member]
$ ZINCRBY ranking:2026 3 jaeseong
$ ZINCRBY ranking:2026 2 jaeseong
```
만약 key가 존재하지 않을 경우 생성과 동시에 Member, Score를 저장한다.
key가 존재할 경우 해당하는 Member의 Score를 increment만큼 증가시킨다.
#### 데이터 개수 조회
```shell
# ZCARD [key]
ZCARD ranking
```
#### score 기준 데이터 삭제
```shell
# ZREMRANGEBYSCORE [key] [min score] [max score]
ZREMRANGEBYSCORE ranking 0 3
```