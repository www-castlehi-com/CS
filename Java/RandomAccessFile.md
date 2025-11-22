> 본 문서는 동기 기반 RandomAccessFile의 내용을 포함하며, 파일 읽기 흐름만 분석한다.

## 1. 정의
파일의 임의 위치에서 읽기/쓰기/수정을 할 수 있도록 설계된 바이트 기반 I/O 클래스다.
Reader 계열이 아니기 때문에 문자 인코딩 처리도 하지 않으며 Stream 계열도 아니다.
NIO도 아니지만 필요 시 일부 사용 가능하다.

```java
private FileDescriptor fd;
```
핵심은 이 `FileDescriptor`이다.

### FileDescriptor란?
Java가 OS에게 파일 핸들을 전달받아 저장해놓은 객체이다.
Java 내부에서는 단순히 int 값만 들고 있는데, OS가 열어준 파일에 대한 핸들이다.
I/O 작업을 할 시 전부 이 fd를 OS에 넘겨서 수행한다.

예를 들어, Linux에서 프로세스의 fd는 다음과 같다.
```
0 - stdin  
1 - stdout  
2 - stderr  
3 - open file A  
4 - open socket  
5 - open file B  
...
```
Java는 이 번호를 Java 객체로 감싸 FileDescriptor라는 형태로 들고 있다.

그럼 언제 저장될까?
예를 들어 FileInputStream을 생성한다고 가정하자.
```java
new FileInputStream("a.log")
```
이 순간 OS에 파일을 열라는 syscall이 호출되고, OS가 파일 핸들(fd)을 할당하고 Java가 그 값을 `FileDescriptor.fd` 필드에 저장한다.

## 2. 메서드
### 1️⃣ read()
문자 하나를 읽는 메서드다.

```java
public int read() throws IOException {  
    return read0();  
}  
  
private native int read0() throws IOException;
```
`native` 메서드를 호출한다.
즉, C/C++로 구현된 JNI 계층을 통해 OS의 `read()` syscall을 호출한다.
결국 1byte를 읽는 것 뿐인데 반드시 OS까지 거쳐야 한다.

> 실제로 syscall은 매번 발생하지 않는다.
> OS는 디스크를 1바이트씩 읽지 않기 때문이다. 기본적으로 블록 단위로 읽어 커널 페이지 캐시에 저장한다.
> 그렇기에 커널 페이지 캐시에 hit되면 캐시에서 가져오고, 없다면 syscall을 호출한다.

### 2️⃣ read(byte[] b)
배열 크기만큼 읽는 메서드다.
```java
public int read(byte b[]) throws IOException {  
    return readBytes(b, 0, b.length);  
}

private native int readBytes(byte b[], int off, int len) throws IOException;
```
native 메서드 호출로 syscall이 발생하지만 배열 단위로 한 번에 읽기 때문에 syscall은 1번만 발생한다.

시작점은 0이고 읽을 길이는 항상 배열의 크기만큼이다.
그래서 부분 읽기가 불가능하다.
### 3️⃣ read(byte[] b, int off, int len)
배열 특정 구간에 len만큼 읽는 메서드다.

```java
public int read(byte b[], int off, int len) throws IOException {  
    return readBytes(b, off, len);  
}

private native int readBytes(byte b[], int off, int len) throws IOException;
```
가장 빠르고 실용적이다.
동작은 `read(byte[] b)`와 비슷하나 특정 위치를 선택할 수 있어 부분 읽기가 가능하다.

### 4️⃣ readLine()
한 줄을 읽는 메서드다.
EOL을 기준으로 한다.
```java
public final String readLine() throws IOException {  
    StringBuilder input = new StringBuilder();  
    int c = -1;  
    boolean eol = false;  
  
    while (!eol) {  
        switch (c = read()) {  
        case -1:  
        case '\n':  
            eol = true;  
            break;  
        //... 
        }  
    }  
  
    //...
    return input.toString();  
}
```
한 바이트씩 `read()`를 호출해 읽어온다.
앞서 알아봤듯이, `read()`는 native 메서드를 호출하므로 한 줄을 읽을 때 바이트 수만큼의 syscall이 발생한다.

그리고 StringBuilder를 만들어 반환한다.

[[BufferedReader]]는 `synchronized` 였기 때문에 StringBuilder를 사용했는데, RandomAccessFile은 왜 StringBuilder를 사용했을까?
애초에 RandomAccessFile은 thread-safe하지 않다. file pointer를 공유하고 있기 때문이다.
그래서 단일 스레드 사용이 전제되므로 StringBuilder를 사용한다.

### 5️⃣ seek(long pos)
fd가 가진 파일 포인터를 pos로 이동시키는 메서드다.
```java
public void seek(long pos) throws IOException {  
    if (pos < 0) {  
        throw new IOException("Negative seek offset");  
    } else {  
        seek0(pos);  
    }  
}

private native void seek0(long pos) throws IOException;
```
내부에서는 native 메서드인 `seek0(pos)`를 호출하기에 값비싼 메서드다.

하지만 `FileDescriptor`를 처리하지 않는 Reader류와 다르게 랜덤 접근을 제공할 수 있다는 장점이 있다.

## 3. 장점
### 1️⃣ 임의 접근 지원
`seek(pos)`로 파일의 어느 위치로든 즉시 jump가 가능하다. 그 다음 `read()`가 호출되면 그 위치에서부터 시작된다.

### 2️⃣ 읽기 쓰기 모두 지원
본 글에서 알아보지는 않았지만, RandomAccessFile은 쓰기도 지원한다.
따라서 `FileInputStream`/`FileOutputStream`으로 분리하여 관리될 필요가 없다.

### 3️⃣ FileChannel 연동
이 부분도 본 글에서 알아보지는 않았지만, NIO 기능 일부를 활용할 수 있다.

## 4. 단점
### 1️⃣ 줄 단위 읽기 성능 
`readLine()` 메서드에서 볼 수 있었듯이, 바이트마다 `read()` 가 호출된다.
StringBuilder를 사용하는 모습도 있었기에 긴 라인 + 대량 로그 파일에서 성능이 최악이다.

### 2️⃣ 잦은 syscall
RandomAccessFile의 대부분의 메서드는 native 메서드를 이용한다.
심지어 내부 버퍼도 없기에 매번 native 메서드를 호출해야 한다.
이 때문에 syscall 수가 증가하는데 syscall이 발생할 때마다 context swithcing이 발생하기에 CPU가 낭비된다.

### 3️⃣ thread-unsafe
fd의 file pointer가 공유되어 여러 스레드에서 `read`/`write`/`seek`할 경우 pointer가 꼬일 수 있다.
`synchronized` 를 이용한다고 하더라도 pointer 공유 문제는 근본적으로 해결할 수 없기에 단일 스레드 전용으로만 사용해야 한다.