## 1. 정의
`Reader` 계열 스트림에 버퍼링 기능을 추가한 래퍼이다.

Java는 기본 입력 스트림과 기본 문자 입력 스트림을 가진다.
문자와 바이트를 구분하기 때문이다.
기본 입력 스트림은 바이트 단위 입출력을 가지고, 기본 문자 입력 스트림은 char단위 입출력을 가진다.

기본 입력 스트림은 `InputStream`을 이용한다.
`InputStream`은 인코딩 없이 byte단위로 입출력을 하므로 이진 데이터를 처리한다.

기본 문자 입력 스트림은 `Reader`를 이용한다.
char단위로 읽기 때문에 텍스트 파일 처리 전용이며 인코딩을 고려한다.

그래서 BufferedReader는 아래의 체인처럼 동작한다.
```
파일(바이트) → FileInputStream(InputStream) → InputStreamReader(Reader) → BufferedReader(Reader)
```

Java I/O는 데코레이터 패턴으로 설계되었다.
그래서 필요한 기능을 래퍼로 쌓아 조합하는 방식이다.

1. byte로 된 파일을 읽기 위해 `FileInputStream`을 사용해 byte를  읽는다.
2. `InputStreamReader`를 이용해 byte를 char로 변환한다.
3. 버퍼링 기능으로 성능을 높이기 위해 `BufferedReader`를 이용한다.

### 버퍼링이란?
실제 I/O 호출 횟수를 줄이기 위한 대용량 임시 저장 메모리를 의미한다.
파일 네트워크에서는 파일을 읽을 때마다 매번 커널 모드 전환이 발생한다.

```java
private char[] cb;  
private static int defaultCharBufferSize = 8192;  
```
BufferedReader는 `char[]` 버퍼를 가지며 기본적으로 8192bytes (=8KB)를 가진다.

즉, 내부 버퍼만큼 한 번에 읽어서 메모리에 저장하기 때문에 `read()`, `readLine()`호출 시 대부분의 데이터는 파일 I/O를 거치지 않고 JVM 메모리에서 가져올 수 있다.

## 2. 메서드
### 1️⃣ fill()
버퍼를 채우는 메서드다.

```java
private void fill() throws IOException {  
    //...
  
    int n;  
    do {  
        n = in.read(cb, dst, cb.length - dst);  
    } while (n == 0);  
    if (n > 0) {  
        nChars = dst + n;  
        nextChar = dst;  
    }  
}
```
실제 I/O를 해서 새로운 데이터를 읽어와 버퍼에 채운다.

### 2️⃣ read()
문자 하나를 읽는 메서드다.

```java
public int read() throws IOException {  
    synchronized (lock) {  
        ensureOpen();  
        for (;;) {  
            if (nextChar >= nChars) {  
                fill();  
                //...  
            }  
            //...
            return cb[nextChar++];  
        }  
    }  
}
```
`synchronized (lock)`이 걸려 있다.
BufferedReader는 스레드 안전한 reader라는 의미이다.

`ensureOpen()`으로 스트림이 닫히지 않았는지 확인한다.

그 후, 버퍼로부터 문자 하나를 반환한다.
만약, 버퍼가 비어있을 경우 `fill()`로 버퍼를 채운다.

### 3️⃣ read(char[] buf, int off, int len)
여러 문자를 읽는 메서드다.

```java
public int read(char[] cbuf, int off, int len) throws IOException {  
    synchronized (lock) {  
        ensureOpen();  
        //...  
  
        int n = read1(cbuf, off, len);  
        //...  
        while ((n < len) && in.ready()) {  
            int n1 = read1(cbuf, off + n, len - n);  
            //... 
        }  
        return n;  
    }  
}
```
`synchronized (lock)`이 걸려 있다.
`read()`와 마찬가지로 thread-safe 하다.

먼저 `ensureOpen()`으로 스트림이 닫히지 않았는지 확인한다.

그 후, 원하는 만큼 읽을 때까지 `read1()`을 호출해 여러 문자를 읽는다.
사용자가 원하는 크기가 버퍼보다 클 수도 있어 요구량을 채울 때까지 버퍼를 채우고 읽는 과정을 반복하는 것이다.

그럼 실제로 여러 문자를 읽는 메서드인  `read1()`을 확인해보자.

```java
private int read1(char[] cbuf, int off, int len) throws IOException {  
    if (nextChar >= nChars) {  
        //...   
	    if (len >= cb.length && markedChar <= UNMARKED && !skipLF) {  
            return in.read(cbuf, off, len);  
        }  
        fill();  
    }  
    //...
    int n = Math.min(len, nChars - nextChar);  
    System.arraycopy(cb, nextChar, cbuf, off, n);  
    nextChar += n;  
    return n;  
}
```
우선 버퍼가 비었는지 확인하고 `fill()` 하는 작업이 있다.
그런데, 조건이 하나 있다.
사용자가 읽으려는 것이 버퍼보다 클 경우에는 `Reader`로부터 직접 읽어온다.
어차피 버퍼를 거쳐도 모두 반환될 것이고, 복사 비용도 추가로 들기 때문이다.

만약, 버퍼가 비어있지 않을 경우에는 사용자 요청만큼 혹은 현재 버퍼에 남은 수만큼 읽는다.

예로 들어, 사용자 요청의 길이가 1000인데 내부 버퍼에 남은 데이터는 600이라면 600만 복사하고 종료한다.
내부 버퍼에 남은 데이터가 2000이라면 사용자 요청 길이인 1000만 복사하고 종료한다.

`read1()` 한 번 호출로 사용자가 원하는 만큼 못 읽을 수 있기 때문에 `read1()`을 여러 번 호출하는 것이다.

### 4️⃣ readLine()
한 줄을 읽는 메서드다.
EOL을 기준으로 한다.

```java
public String readLine() throws IOException {  
    return readLine(false, null);  
}

String readLine(boolean ignoreLF, boolean[] term) throws IOException {  
    StringBuilder s = null;  
    int startChar;  
  
    synchronized (lock) {  
        ensureOpen();  
        //...
  
    bufferLoop:  
        for (;;) {  
  
            if (nextChar >= nChars)  
                fill();  
            //...  
  
        charLoop:  
            for (i = nextChar; i < nChars; i++) {  
                c = cb[i];  
                if ((c == '\n') || (c == '\r')) {  
                    if (term != null) term[0] = true;  
                    eol = true;  
                    break charLoop;  
                }  
            }  
  
            //... 
  
            if (eol) {    
                //...  
                return str;  
            }  
  
            if (s == null)  
                s = new StringBuilder(defaultExpectedLineLength);  
            s.append(cb, startChar, i - startChar);  
        }  
    }  
}
```
마찬가지로 `synchronized (lock)`을 걸고, `ensureOpen()`으로 스트림이 닫히지 않았는지 확인한다.
그리고 다른 버퍼가 비어 있으면 `fill()` 하는 작업이 반복된다.

내부적으로 `StringBuilder`를 이용한다.
>`StringBuilder`는 `synchronized`가 없는데 `BufferedReader` 내부적으로 `synchronized`이기 때문에 단일 스레드에서 호출되어 동기화가 필요 없다.
그래서 `synchronized lock`이 없어 빠른 `StringBuilder`를 이용한다.

charLoop에서는 버퍼를 순회하며 EOL을 찾는다.
만약 EOL을 발견하면 str을 만들어 반환한다.
(이 때 StringBuilder를 쓸 수도 있고 쓰지 않을 수 있다.)

만약 EOL을 발견하지 않았다면 StringBuilder를 만들어 누적시킨다.
그럼 다시 bufferLoop 상단에서 `fill()`을 호출해 버퍼를 채우고 위 과정을 반복한다.

## 3. 장점
### 1️⃣ System call 감소
앞서 말했듯이, I/O는 비싸다.
그래서 문자를 하나 읽을 때마다 OS에 syscall이 들어가 커널 모드로 전환하게 되면 속도가 매우 느려진다.

BufferedReader의 `fill()`에서 볼 수 있었듯이, 내부 buf를 채운 후 메모리 접근을 하기 때문에 System call이 감소한다.

그리고 디스크/네트워크는 원래 블록 단위로 읽는 것이 더 효율적이다.
### 2️⃣ 공간 지역성
BufferedReader는 char[] 을 사용하는데, 배열은 연속된 메모리에 저장된다.
그래서 CPU cache hit가 잘 발생하여 메모리 접근 비용이 크게 줄어들 수 있다.

## 4. 단점
### 1️⃣ 특정 offset 상수 접근 불가
BufferedReader는 순차 읽기 전용이다.
특정 위치로 점프하기 위해서는 `readLine()` 혹은 `read()`를 반복해서 사용해야 한다.

### 2️⃣ 대용량 파일 처리 한계
`readLine()`에서 볼 수 있었듯이,  긴 줄을 만나면 버퍼 `fill()`과 스캔이 여러 번 반복된다. 그래서 CPU에 부담이 생긴다.
그리고 `StringBuilder.append()`가 반복된다.
`StringBuilder.append()`는 내부적으로 더 큰 char[]를 생성하고 기존 내용을 복사하면서 기존 배열은 버리는 작업을 한다.  이 과정에서 줄이 길면 길수록 대형 객체가 생성되게 된다. 결국 GC에 부담이 생기고 CPU 부담으로 이어진다.