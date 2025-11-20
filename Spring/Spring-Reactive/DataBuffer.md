## 1. 배경
WebFlux는 Netty 기반이다
Netty는 ByteBuf라는 고성능 I/O 버퍼를 사용하는데, Spring은 ByteBuf에 직접 의존하지 않기 위해 DataBuffer라는 추상 레이어를 제공한다.

Netty가 ByteBuf를 사용하는 이유는 byte[]로는 부족하기 때문이다.
byte[]가 가지는 문제점은 아래와 같다.
- 힙 메모리를 사용하므로 GC 부담이 있다.
- 버퍼를 재사용(pooling)하지 않으므로 대용량 스트리밍에서 OOM 위험이 있다.
- read/write index가 없어 직접 계산해야 한다.
- 확장성이 없어 복사가 필요하다.
- zero-copy가 없어 파일 처리가 느리다.

그러면 ByteBuffer를 사용하면 되지 않을까?
실제로 ByteBuffer는 자바 NIO에서 제공하는 클래스이다.
하지만, 이도 한계가 있다.
- pooling이 없어 매번 새로 만들고 버려야 한다.
- read/write 인덱스가 분리되지 않아 두 인덱스를 하나로 쓰기 때문에 불편하다.
- zero-copy가 미흡하다. slice가 있긴 하지만 제한적이며 네트워크 I/O에서 실제 zero-copy를 효율적으로 사용하기 어렵다.
- 리사이징이 불가능하다. capacity를 넘어가면 새 ByteBuffer를 만들어서 복제해야 한다.

그래서 Netty는 ByteBuf를 사용한다.
byte[]나 ByteBuffer의 단점을 보완한 기능을 제공한다.
- pooling이 가능하다.
- heap / direct buffer 선택이 가능하다. 파일 / 네트워크 전송은 direct buffer를 이용해 훨씬 빠르게 동작시킨다.
- readIndex와 writeIndex를 분리한다.
- zero-copy가 가능하다. slice, duplicate, retained slice 등을 제공한다.
- dynamic resizing을 지원하여 자동 확장이 가능하다.
- reference counting을 지원하여 메모리 수명을 직접 관리하여 GC를 회피한다.

즉, 파일스트리밍, SSE, WebSocket, chunked HTTP 등에서 최고 성능을 내기 위한 I/O 버퍼라고 할 수 있다.

하지만, Spring은 ByteBuf를 그대로 사용하지 않는다.
Spring은 Netty외에도 Tomcat, Jetty, Undertow 등을 모두 지원한다.
ByteBuf는 Netty 전용이므로 Webflux를 Netty가 아닌 다른 HTTP 서버에서 구동하면 문제가 된다.

그래서 Spring은 서버 종류 상관없이 통용되는 추상화를 했고, 그것이 바로 DataBuffer이다.
그리고 각 서버의 buffer의 생명주기와 I/O스트림 처리를 하는 유틸성 클래스인 DataBufferUtils를 제공한다.

## 2. 종류
앞서 DataBuffer는 추상화 클래스라는 것을 알게 되었다.
구현체는 크게 두 가지이다.
![](https://velog.velcdn.com/images/www-castlehi-com/post/5d936209-dc19-4eed-99b1-9a4f1420ee1c/image.png)
### 1️⃣ NettyDataBuffer
Netty의 ByteBuf 기반이다.
대부분 WebClient/WebFlux 서버에서 NettyDataBuffer를 사용한다.
### 2️⃣ DefaultDataBuffer
힙 기반 byte[] 이다.

---
우리는 위의 다이어그램에서 Netty는 PooledDataBuffer를 구현하며, DefaultDataBuffer는 그렇지 않다는 것을 알 수 있다.
앞서 살펴보았듯이 byte[]는 힙을 기반으로 한다.

그렇다면 메모리 풀 기반과 힙 기반은 무엇이 다를까?

힙 기반이란 것은 JVM이 힙 영역에서 새로운 메모리 블록을 찾아준다는 이야기이다.
한 번 쓰고 나면 GC가 다시 이 객체를 수거한다.

매번 쓸 때마다 새로 생성해야 하고 재사용이 되지 않는다는 것이다.
그리고 많이 사용될수록 GC가 자주 돌고 GC가 느려지면 전체 서비스 성능이 저하된다.
즉, GC 부담이 크다.

반면에 메모리 풀 기반은 애플리케이션 시작 시 미리 buffer를 많이 만들어서 풀에 넣어둔다.
그리고 필요할 때 풀에서 꺼내 쓰고 다 쓰면 풀에 다시 돌려놓는다.

매번 new하지 않기 때문에 할당 비용이 0에 수렴하고 재사용하므로 메모리 단편화나 GC 압박이 없다.
특히 Netty는 off-heap memory 영역의 DirectMemory 사용을 선호하기에 GC가 이 메모리를 추적하지 않아 부담이 거의 없다.

## 3. ByteBuf 기능
### 1️⃣ index
ByteBuf는 readIndex와 writeIndex를 분리한다.
```
+-------------------------------------------------------+
|                underlying memory block                |
+-------------------------------------------------------+
^            ^                  ^
0        readerIndex       writerIndex
```
- readerIndex
  - 읽기 시작점
  - `readBytes()` 등을 호출하면 증가한다
- writeIndex
  - 쓰기 시작점
  - `writeBytes()` 등을 호출하면 증가한다
- capacity
  - 이 ByteBuf가 갖고 있는 총 메모리 크기
  
> byte[]에서는 개발자가 직접 offset과 length를 이용해 매번 계산해야 한다.
```java
int offset = 0;
int length = bytes.length;
int read = input.read(bytes, offset, length);
offset += read;
length -= read;
```
  
### 2️⃣ buffer
ByteBuf는 두 가지 종류의 buffer를 지원한다.
#### 1) HeapByteBuf
- byte[] 기반
#### 2) DirectByteBuf
- off-heap (native 메모리)

Netty가 선호하는 것은 DirectByteBuf이다.
GC가 관리하지 않지만 NIO 환경과 대용량 파일/네트워크에서 뛰어난 성능을 가지기 때문이다.

### 3️⃣ Reference Count
NettyDataBuffer는 GC가 메모리를 추적하지 않아 GC의 부담이 없다는 말은 다른 말로 하면 언제 free 해야 하는지 GC가 모른다는 의미가 된다.
결국, Netty가 메모리를 직접 해제해야 한다.
그렇지 않으면 메모리 누수가 발생한다.

Netty는 이를 reference count로 해결한다.

ByteBuf는 다음과 같은 숫자를 가지고 있다.
`refCnt = 현재 이 ByteBuf를 쓰는 주체 수`

`refCnt`가 0이라는 것은 현재 이 메모리를 사용하고 있는 주체가 없다는 것이 된다.
그럼 Netty는 `release()`를 통해 메모리를 free하고 ByteBufAllocator (pool)에 메모리를 반납한다.

---
그래서 개발자는 NettyDataBuffer를 사용할 때 사용 후 `release()`를 꼭 호출해 반납해야한다.
```java
try {
	/*NettyDataBuffer 사용*/
} catch (IOException e) {
	throw new RuntimeException();
} finally {
	DataBufferUtils.release(dataBuffer);
}
```

### 4️⃣ Zero-copy
복사 없이 같은 메모리 블록을 공유하고 인덱스(readerIndex/writeIndex)만 다르게 가진 새로운 메모리 view를 만든다.
Zero-copy를 지원하는 방법은 세 가지가 있다.

원본 ByteBuf가 아래와 같다고 가정하자.
```
Underlying Memory (1024 bytes)
+-------------------------------------------------------+
| 0  1  2  3  4  5  ...                       ... 1023  |
+-------------------------------------------------------+
```
#### 1) slice()
에를 들어 200~300인 구간만 보고 싶으면 실제 메모리는 그대로 두고, view만 만든다.
```
ByteBuf slice = original.slice(200, 100);

=>
Underlying Memory (same)
+-------------------------------------------------------+
| 0 ... 199 | <- slice region (200~299) -> | 300 ...   |
+-------------------------------------------------------+
               ^
               readerIndex=0
               writerIndex=100
```
실제 데이터 복사가 없다.
하지만, slice는 refCnt를 증가시키지 않기 때문에 original이 더 먼저 free되면 문제가 발생한다.
#### 2) duplicate()
메모리는 그대로 공유하고, 각자의 인덱스(readerIndex/writeIndex)를 가진다.
```
ByteBuf dup = original.duplicate();

=>
Underlying Memory 공유 (0~1023)
+-------------------------------------------------------+
| same memory                                           |
+-------------------------------------------------------+

original: readerIndex=0, writerIndex=1024
duplicate: readerIndex=0, writerIndex=1024 (독립된 view)
```
둘이 같은 메모리를 바라보고, 초기에는 인덱스도 동일하지만 완전히 독립된 view로 동작한다.

#### 3) retainedSlice()
`slice()`를 사용하지만 `refCnt`도 증가시킨다.
`slice()`를 안전하게 재사용하기 위한 기능이다.

### 5️⃣ pooled allocator
고성능 메모리 관리 시스템이다.
buffer를 free하지 않고 미리 큰 메모리 덩어리를 확보해서 잘라쓸 수 있도록 한다.
```
PooledByteBufAllocator
    ↓
[여러 Arena]
    ↓
[여러 Chunk]
    ↓
[여러 Page]
    ↓
[slice된 ByteBuf]
```
#### 1) Allocator
Netty 전체에서 buffer 할당 요청을 받는 매니저다.
NIO/Netty worker thread 수만큼 arena를 준비한다.
#### 2) Arena
메모리 풀의 대표 공간이다.
thread-safe 또는 thread-local 형태로 운영한다.
다중 스레드 경쟁을 줄이기 위해 여러 Areana가 존재하며, CPU 코어 이상개 존재한다.
#### 3) Chunk
Arena 안에 있는 아주 큰 메모리 덩어리다.
기본 크기는 16MB이다.
#### 4) Page
Chunk에서 일정 크기 단위로 자른 것이다.
#### 5) ByteBuf
Page의 슬라이스이다.

## 4. Backpressure
`Flux<DataBuffer>`은 Backpressure-aware이다.
하지만, 내부에 블로킹 코드를 넣거나 `request()` 시그널 제어가 불가능한 상황이 발생하도록 개발하면 Backpressure가 발생할 수 있다.
따라서 항상 Backpressure-safe한 개발을 해야 한다.
### 1️⃣ flatMapSequencital(concurrency = 1, prefetch=1)
```java
.bodyToFlux(DataBuffer.class)
.flatMapSequential(buf ->
      Mono.fromCallable(() -> writeToStream(buf))
          .subscribeOn(boundedElastic()),
    1,   // concurrency → 한 번에 하나만
    1    // prefetch → 한번에 한 청크만 요청
)
```
downStream이 처리할 때까지 `next()`를 요청하지 않는다.
처리가 끝나면 subscribe chain에서 `request(1)`씩 upstream에 전달한다.
결국 데이터 처리 과정이 느려도 upstream이 멈추기 때문에 Backpressure 문제가 해소된다.
### 2️⃣ DataBufferUtils.write()
```java
onSubscribe(subscription) {
    subscription.request(1);
}

onNext(dataBuffer) {
    write(dataBuffer);   // outputStream.write
    release(dataBuffer);
    subscription.request(1);  // 다음 chunk
}
```
간단하게 본 내부 로직 구조이다.
`DataBufferUtils.write()`는 내부적으로 자체 Subscriber를 사용한다.
따라서 `request()`를 제어하기 때문에 Backpressure 문제가 해소된다.