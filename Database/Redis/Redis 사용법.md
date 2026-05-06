## 1. 기본 명령어
### 1.1 데이터 저장
```shell
# set [key 이름] [value]
$ set seongha:name "seongha park" #띄어쓰기 해서 저장하려면 쌍따옴표로 묶는다
$ set seongha:hobby game
```
기존 키의 값을 갱신할 수도 있다.
### 1.2 데이터 조회
```shell
# get [key 이름]
$ get seongha:name
$ get seongha:hobby
```
없는 데이터를 조회할 경우 (nil)이 출력된다.
### 1.3 저장된 모든 key 조회
```shell
$ keys *
```
### 1.4 데이터 삭제
```shell
# del [key 이름]
$ del seongha:hobby
```
### 1.5 TTL
레디스는 RDBMS와 다르게 데이터 저장 시 만료 시간을 설정할 수 있다.
메모리 공간이 한정 되어 있기 때문에 모든 데이터를 레디스에 저장할 수 없다. 따라서 만료 시간을 활용해 자주 사용하는 데이터만 레디스에 저장해놓고 사용한다.
```shell
# set [key 이름] [value] ex [만료 시간(초)]
$ set seongha:pet none ex 30
```
### 1.6 TTL 확인
```shell
# ttl [key 이름]
$ ttl seongha:pet
```
만료 시간이 반환된다.
키가 없는 경우 -2가 반환된다.
키는 존재하지만 만료 시간이 설정되어 있지 않은 경우 -1을 반환한다.
### 1.7 EXPIRE
```shell
# EXPIRE [key] [seconds]
$ EXPIRE ranking 10 
```
이미 존재하는 key에 만료 시간을 설정한다.
### 1.8 모든 데이터 삭제하기
```shell
$ flushall
```
### 1.9 메모리 확인
```shell
# MEMORY USAGE [key]
$ MEMORY USAGE dau:2026-05-06
```
key의 데이터를 저장하는데 사용된 메모리를 확인한다.
## 2. key 네이밍 컨벤션
콜론(:)을  활용해 계층적으로 의미를 구분해서 사용한다.
> ex.
> - `users:100:profile` : 사용자들(users) 중에서 PK가 100인 사용자(user)의 프로필(profile)
> - `products:123:details` : 상품들(products) 중에서 PK가 123인 상품(product)의 세부사항(details)

계층적으로 의미를 구분했을 때 장점은 아래와 같다.
- **가독성** : 데이터의 의미와 용도를 쉽게 파악할 수 있다.
- **일관성** : 컨벤션을 따름으로써 코드의 일관성이 높아지고 유지보수가 쉬워진다.
- **검색 및 필터링 용이성** : 패턴 매칭을 사용해 특정 유형의 key를 쉽게 찾을 수 있다.
- **확장성** : 서로 다른 key와 이름이 겹쳐 충돌할 일이 적어진다.