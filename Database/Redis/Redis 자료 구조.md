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
