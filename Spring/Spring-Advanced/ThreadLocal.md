## 개념
- [[동시성 문제]] 해결 방안
- 쓰레드마다 별도의 내부 저장소 제공
- 같은 인스턴스의 쓰레드 로컬 필드에 접근해도 문제 없음

> 자바는 언어차원에서 쓰레드 로컬을 지원하기 위한 `java.lang.ThreadLocal` 클래스 제공
## 동작
![](https://i.imgur.com/QJxxI1I.png)
## 사용법
- **초기화** : `ThreadLocal<?> threadLocal = new ThreadLocal<>();`
- **값 저장** : `ThreadLocal.set(xxx);`
- **값 조회** : `ThreadLocal.get()`
- **값 제거** : `ThreadLocal.remove()`
## 주의사항
### 값 미제거
#### 예시
![](https://i.imgur.com/lYsJAOB.png)
1. 사용자A가 저장 HTTP 요청
2. WAS는 쓰레드 풀에서 쓰레드 하나 조회
3. 쓰레드 `thread-A` 할당
4. `thread-A`는 사용자 A의 데이터를 쓰레드 로컬에 저장
5. 쓰레드 로컬의 `thread-A` 전용 보관소에 사용자 A 데이터 보관
	- 쓰레드를 생성하는 비용은 비싸기 때문에 쓰레드를 제거하지 않고, 쓰레드 풀을 통해서 쓰레드를 재사용
	- `thread-A`는 쓰레드풀에 아직 살아있으며, 쓰레드 로컬의 `thread-A` 전용 보관소에 사용자A 데이터도 함께 살아있게 됨
![](https://i.imgur.com/m22GZRT.png)
6. 사용자 B가 조회를 위한 새로운 HTTP 요청
7. WAS가 쓰레드 풀에서 쓰레드 하나 조회
8. 우연히 쓰레드 `thread-A`가 할당
9. `thread-A`는 쓰레드 로컬에서 데이터를 조회
10. 쓰레드 로컬은 `thread-A` 전용 보관소에 있는 사용자 A의 데이터 반환
11. 사용자 B는 사용자 A의 정보를 조회하게 됨
#### 해결
- 쓰레드 로컬 사용 후 값을 `ThreadLocal.remove()`를 통해서 제거해야 함