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
### 2.1 명령어 옵션
```
# SET [key] [value] NX
SET hobby golf NX
```
지정한 키가 없을 때에만 데이터를 저장한다.
## 3️⃣ Sorted Set
### 3.1 개념
![](https://i.imgur.com/TcE7bxg.png)
score(점수)를 기준으로 자동 정렬되는 중복 없는 자료구조이다.
랭킹 시스템, 실시간 인기 데이터 관리 등 순서와 가중치가 중요한 경우에 주로 활용된다.
### 3.2 명령어
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
## 4️⃣ Geospatial
### 4.1 개념
위치 데이터(경도, 위도)를 저장하고 검색하는데 최적화된 자료구조이다.
특정 위치 기준으로 주어진 거리 내의 데이터를 찾아야 할 때 자주 사용한다.
### 4.2 명령어
#### 저장
```shell
# GEOADD [key] [위도] [경도] [member]
GEOADD cafe 127.0276 37.4979 a_cafe
```
내부적으로 Sorted Set처럼 저장된다.
#### 조회
```shell
ZRANGE cafe 0 -1
```
Sorted Set처럼 저장되니 모든 member를 조회할 때에는 `ZRANGE`를 사용할 수 있다.
```shell
#GEOPOS [key] [member...]
GEOPOS cafe a_cafe b_cafe c_cafe
```
#### 거리 조회
```shell
#GEODIST [key] [member1] [member2] [단위]
GEODIST cafe a_cafe b_cafe m
```
두 데이터 사이의 직선 거리를 조회한다.
```shell
# GEOSEARCH [key] FROMLONLAT [위도] [경도] BYRADIUS [반경] [단위] [ASC/DESC] WITHDIST
GEOSEARCH cafe FROMLONLAT 127.0280 37.4975 BYRADIUS 1000 m ASC WITHDIST
```
특정 위치를 기준으로 원하는 반경 내에 있는 데이터를 조회한다.
## 5️⃣ Set
### 5.1 개념
![](https://i.imgur.com/PWNsKPP.png)
정렬되지 않은 중복 없는 자료구조이다.
순서는 중요하지 않지만, 중복되면 안되는 데이터를 다룰 때 적합하다.
### 5.2 명령어
#### 삽입
```shell
# SADD [key] [member ...]
SADD member seongha
```
Set에 데이터를 저장한다.
한 번에 여러 개의 값을 넣을 수도 있으며 중복되는 값이 요청될 경우 삽입되지 않는다.
#### 조회
```shell
# SMEMBERS [key]
SMEMBERS member
```
모든 데이터를 조회한다.
```shell
# SCARD [key]
SCARD member
```
데이터의 개수를 조회한다.
## 6️⃣ Bitmap
### 6.1 개념
![](https://i.imgur.com/IKu8LtS.png)
아주 적은 메모리로 대량의 boolean 상태를 저장할 때 사용하는 자료구조이다.
Set처럼 중복을 허용하지 않는다.
사용자별 일일 출석 여부를 확인하거나 대규모 이벤트에서 사용자별 참석 여부를 확인하거나 하루 동안 서비스에 방문한 사용자 수인 DAU를 구할 때 사용한다.

key, offset, value 구성 요소를 가진다.
offset에는 0 이상의 정수만 넣을 수 있어 인덱스 역할을 한다.
value에는 0 또는 1 숫자(boolean) 만을 입력할 수 있다.
### 6.2 명령어
#### 삽입
```shell
# SETBIT [key] [offset] [value]
SETBIT user:attend 4 1
```
#### 조회
```shell
# GETBIT [key] [offset]
GETBIT user:attend 2
GETBIT user:attend 7
```
특정 offset의 value를 확인한다.
저장되어 있지 않은 offset 요청 시 0을 출력한다.
```shell
# BITCOUNT [key]
BITCOUNT user:attend
```
value가 1로 저장된 모든 데이터의 개수를 카운팅한다.
## 7️⃣  Hash
### 7.1 개념
![](https://i.imgur.com/v7d9fav.png)
하나의 key 안에 여러 개의 field-value 쌍을 저장할 수 있는 자료구조다.
여러 속성들을 묶어서 저장할 때 적합하다.
### 7.2 명령어
#### 삽입
```shell
# HSET [key] [field] [value]
HSET user name seongha
HSET user age 27
HSET user hobby game
```
#### 조회
```shell
# HGETALL [key]
HGETALL user
```
Hash에 들어있는 모든 field, value를 조회한다.
```shell
# HGET [key] [field]
HGET user name
HGET user age
HGET user hobby
```
Hash에 들어있는 특정 field의 value를 조회한다.