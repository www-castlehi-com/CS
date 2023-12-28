[오라클 docs : LinkedList](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedList.html)
# LinkedList란?
- `list`와 `deque`를 구현하는 이중연결리스트
- 순차적으로 조회할 때 좋음
- 인덱스로 접근하는 연산의 경우 시작점 또는 끝점에서 가까운 점부터 순회
- 복사가 필요 없어 메모리 관리에 효율적
- `iterator`와 `listIterator`는 반복 중 구조적으로 변경될 경우 `ConcurrentModificationException`을 발생시킴
- 동기화되지 않기 때문에 스레드 환경에서 사용할 경우 다음과 같이 선언해야 함 -> 스레드 안전하지 않음
	```java
	List list = Collections.synchronizedList(new ArrayList(...));
	```

# 시간복잡도 비교
|                               메소드                               |  시간 복잡도   |
|:------------------------------------------------------------------:|:--------------:|
|                          `get(int index)`, `set(int index, E element)`                          |      O(1)      |
|                          `add(E element)`, `addFirst(E element)`, `addLast(E element)`                          | O(1) |
|                    `add(int index, E element)`                     |      O(n)      |
|              `remove(int index)`, `remove(Object o)`               |      O(n)      |
| `indexOf(Object o)`, `lastIndexOf(Object o)`, `contains(Object o)` |      O(n)      |
|                              `size()`                              |      O(1)      |
|                            `isEmpty()`                             |      O(1)      |
|                       `clear()`                      |      O(n)      |

> 전체적으로 [[ArrayList]]보다 느리지만, **빈번한 요소 추가/삭제** 시 효율적 -> 덱이나 큐 구현

# 구현
## List 인터페이스
```java
import java.util.Iterator;  
import java.util.ListIterator;  
  
public interface List<T> {  
    int size();  
    boolean isEmpty();  
  
    boolean contains(Object o);  
    int indexOf(Object o);  
    int lastIndexOf(Object o);  
  
    Iterator<T> iterator();  
    ListIterator<T> listIterator();  
  
    boolean add(T e);  
    void add(int index, T element);  
    T get(int index);  
    T set(int index, T element);  
    boolean remove(Object o);  
    T remove(int index);  
  
    void clear();  
}
```

## LinkedList
### 필드
```java
public class LinkedList<T> implements List<T> {  
    private static class Node<T> {  
        T item;  
        Node<T> next;  
        Node<T> prev;  
  
        public Node(T item, Node<T> next, Node<T> prev) {  
            this.item = item;  
            this.next = next;  
            this.prev = prev;  
        }  
    }  
  
    int size = 0;  
    Node<T> first;  
    Node<T> last;

	//...
}
```

1️⃣ **Node**
- LinkedList에 담을 요소
- 이전 노드와 다음 노드에 대한 정보를 가지고 있음

2️⃣ **size**
- LinkedList의 크기

3️⃣ **first, last**
- 처음, 끝 노드에 대한 정보

### 생성자
```java
public LinkedList() {}  
  
public LinkedList(Collection<? extends T> c) {  
    this();  
  
    for (T element : c) {  
        Node<T> newNode = new Node<>(element, last, null);  
  
        if (last == null) {  
            first = newNode;  
        }  
        else {  
            last.next = newNode;  
        }  
        last = newNode;  
    }  
  
    size = c.size();  
}
```

1️⃣ **기본 생성자**

2️⃣ **컬렉션 기반 생성자**
- 컬렉션에 있는 element를 담은 새로운 리스트 생성

### Create
#### List의 add 메소드
```java
@Override  
public boolean add(T e) {  
    Node<T> newNode = new Node<>(e, last, null);  
    if (last == null)  
        first = newNode;  
    else        
	    last.next = newNode;  
    last = newNode;  
    size++;  
    return true;
}  
  
@Override  
public void add(int index, T element) {  
    if (index < 0 || index > size)  
        throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);  
  
    if (index == size)  
        add(element);  
    else {
	    // 들어갈 자리 찾기  
        Node<T> x;  
        if (index < (size >> 1)) {  
            x = first;  
            for (int i = 0; i < index; i++) x = x.next;  
        }  
        else {  
            x = last;  
            for (int i = size - 1; i > index; i--) x = x.prev;  
        } 
        
		// 들어갈 자리에 새 노드 삽입
        Node<T> pred = x.prev;  
        Node<T> newNode = new Node<>(element, pred, x);  
        if (pred == null) first = newNode;  
        else pred.next = newNode;  
        x.prev = newNode;  
        size++;  
    }  
}
```

1️⃣ `public boolean add(T e)` : 끝 부분에 노드 삽입
- last가 null (빈 리스트)일 경우 first와 last 모두 newNode로 초기화
- 빈 리스트가 아닐 경우 기존 last의 next를 newNode로 초기화하고 last를 newNode로 업데이트

2️⃣ `public void add(int index, T element)` : index 부분에 노드 삽입
- index가 끝 부분을 가리킬 경우 끝 부분에 노드 삽입 (`add(T e)`)
- index가 size의 절반보다 작을 경우 first 부터 탐색하고, 절반보다 클 경우 last부터 탐색
- index에 있던 노드가 first일 경우 first를 newNode로 업데이트하고, 원래 있던 노드의 prev는 newNode를 가리킴
- index에 있던 노드가 중간 노드일 경우 원래 index에 있던 노드의 이전 노드의 next가 newNode를 가리킴

#### 추가 add 메소드
```java
public void addFirst(T e) {  
    Node<T> newNode = new Node<>(e, null, first);  
    if (first == null)  
        last = newNode;  
    else        first.prev = newNode;  
    first = newNode;  
    size++;  
}  
  
public void addLast(T e) {  
    add(e);  
}
```

1️⃣ `public void addFirst(T e)` : 첫 부분에 노드 삽입
- 빈 리스트일 경우 first, last를 모두 newNode로 초기화
- 빈 리스트가 아닐 경우 first의 prev가 newNode를 가리키게 하고 first를 newNode로 초기화

2️⃣ `public void addLast(T e)` : 끝 부분에 노드 삽입
- `public boolean add(T e)`와 동일

#### Deque 메소드
```java
public boolean offer(T e) {  
    return add(e);  
}

public void push(T e) {  
    addFirst(e);  
}
```

1️⃣ `boolean offer(T e)` = `offerLast()`: FIFO
add` 메소드들의 경우 공간이 부족하면 `IllegalStateException`을 던진다.
하지만 Queue가 제공하는 메소드 중 하나인 offer를 이용할 경우 true / false를 던진다.
[Oracle docs](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Queue.html)에는 위처럼 명시되어 있지만 실제 구현은 `add` 메소드를 이용해서 구현한다.
> LinkedList의 동적 확장성 때문에 공간이 부족해 예외가 발생할 일이 적기 때문에 `add` 메소드를 그대로 이용

2️⃣ `void push(T e)` : LIFO

### Read
### get
```java
@Override  
public T get(int index) {  
    if (index < 0 || index >= size)  
        throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);  
  
    Node<T> x;  
    if (index < (size >> 1)) {  
        x = first;  
        for (int i = 0; i < index; i++) x = x.next;  
    }  
    else {  
        x = last;  
        for (int i = size - 1; i > index; i--) x = x.prev;  
    }  
  
    return x.item;  
}
```
- index 부분에 있는 노드를 찾아 요소를 반환 -> 찾는 방법은 `public void add(int index, T element)` 와 동일

#### indexOf / lastIndexOf
```java
@Override  
public int indexOf(Object o) {  
    int index = 0;  
    if (o == null) {  
        for (Node<T> x = first; x != null; x = x.next) {  
            if (x.item == null)  
                return index;  
            index++;  
        }  
    }  
    else {  
        for (Node<T> x = first; x != null; x = x.next) {  
            if (o.equals(x.item))  
                return index;  
            index++;  
        }  
    }  
    return -1;  
}  
  
@Override  
public int lastIndexOf(Object o) {  
    int index = size;  
    if (o == null) {  
        for (Node<T> x = last; x != null; x = x.prev) {  
            index--;  
            if (x.item == null)  
                return index;  
        }  
    }  
    else {  
        for (Node<T> x = last; x != null; x = x.prev) {  
            index--;  
            if (o.equals(x.item))  
                return index;  
        }  
    }  
    return -1;  
}
```

1️⃣ 접근하려는 원소 o가 `null`일 경우
- `==` 연산자를 이용하여 맨 앞에 나오는 null을 찾음

2️⃣ 접근하려는 원소 o가 `null`이 아닐 경우
- `equals` 메소드를 이용하여 o와 같은 내용을 가진 원소 중 가장 먼저 나오는 원소를 찾음

>**`null` 비교와 `Non-null` 비교**
> - `==` 연산자
> 	- null 비교 시 사용
> 	- 메모리의 주소를 비교
> 	- 객체 간의 비교 시 추천되지 않는 방식
> - `equals` 메소드
> 	- null 비교 시 `NullPointerException` 
> 	- 메모리의 주소를 비교
> 	- `@Override` 시 내용을 비교할 수 있음 (보통 `hashCode`와 함께 재정의)
> 	- 객체 간의 비교 시 추천되는 방식

#### Deque 메소드
```java
public T element() {  
    if (first == null) {  
        throw new NoSuchElementException();  
    }  
    return first.item;  
}

public T peek() {  
    return (first == null) ? null : first.item;  
}
```

1️⃣ `T element()`
- 빈 리스트일 경우 `NoSuchElementException` 반환

2️⃣ `T peek()` = `peekFirst()`
- 빈 리스트일 경우 null 반환

### Update
```java
@Override  
public T set(int index, T element) {  
    if (index < 0 || index >= size)  
        throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);  
  
    Node<T> x;  
    if (index < (size >> 1)) {  
        x = first;  
        for (int i = 0; i < index; i++) x = x.next;  
    }  
    else {  
        x = last;  
        for (int i = size - 1; i > index; i--) x = x.prev;  
    }  
    T oldVal = x.item;  
    x.item = element;  
    return oldVal;  
}
```

- index에 위치한 노드의 원래 값을 저장해 반환
- index에 위치한 노드를 새로운 element 값으로 업데이트

### Delete
#### remove
```java
@Override  
public boolean remove(Object o) {  
    if (o == null) {  
        for (Node<T> x = first; x != null; x = x.next) {  
            if (x.item == null) {  
                unlink(x);  
                return true;            }  
        }  
    }  
    else {  
        for (Node<T> x = first; x != null; x = x.next) {  
            if (o.equals(x.item)) {  
                unlink(x);  
                return true;            }  
        }  
    }  
    return false;  
}  
  
@Override  
public T remove(int index) {  
    if (index < 0 || index >= size)  
        throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);  
  
    Node<T> x;  
    if (index < (size >> 1)) {  
        x = first;  
        for (int i = 0; i < index; i++) x = x.next;  
    }  
    else {  
        x = last;  
        for (int i = size - 1; i > index; i--) x = x.prev;  
    }  
    return unlink(x);  
}
```

파라미터인 원소 혹은 index에 해당하는 노드를 찾아 `unlink()`를 해주고 원래 값을 반환

```java
private T unlink(Node<T> x) {  
    final T element = x.item;  
    final Node<T> next = x.next;  
    final Node<T> prev = x.prev;  
  
    if (prev == null) {  
        first = next;  
    } else {  
        prev.next = next;  
        x.prev = null;  
    }  
  
    if (next == null) {  
        last = prev;  
    } else {  
        next.prev = prev;  
        x.next = null;  
    }  
  
    x.item = null;  
    size--;  
    return element;  
}
```

- 해당 노드가 first (`prev == null`)일 경우, first를 해당 노드의 다음 노드(`next`)로 변경
- 해당 노드가 first가 아닐 경우, 해당 노드의 이전 노드의 next(`prev.next`)를 다음 노드(`next`)로 변경하고 해당 노드의 prev는 null로 참조를 제거하여 메모리 누수 방지

- 해당 노드가 last (`next == null`)일 경우, last를 해당 노드의 이전 노드(`prev`)로 변경
- 해당 노드가 last가 아닐 경우, 해당 노드의 다음 노드의 prev(`next.prev`)를 이전 노드(`prev`)로 변경하고 해당 노드의 next는 null로 참조를 제거하여 메모리 누수 방지

#### Deque 메소드
```java
public T remove() {  
    return removeFirst();  
}

public T removeFirst() {  
    if (first == null) {  
        throw new NoSuchElementException();  
    }  
    return unlink(first);  
}  
  
public T removeLast() {  
    if (last == null) {  
        throw new NoSuchElementException();  
    }  
    return unlink(last);  
}

public T poll() {  
    return (first == null) ? null : unlink(first);  
}  
  
public T pop() {  
    return removeFirst(); 
}
```

1️⃣ `remove(), removeFirst(), removeLast()`
- 빈 리스트일 시 `NoSuchElementException()` 반환
	> `remove()`의 경우 LinkedList에서만 제공하는 `remove(Object o)`, `remove(int index)`와 혼동되지 않도록 주의!
	> 💡 `remove()`는 LinkedList의 것과 다르게 요소가 없을 경우 예외를 반환함

2️⃣ `poll()` = `pollFirst()`
- 빈 리스트일 시 null 반환

3️⃣ `pop()`
- 구현은 `removeFirst()`와 같음
- 스택의 semantic

### 기타 구현
#### size
```java
@Override  
public int size() {  
    return size;  
}
```

### isEmpty
```java
@Override  
public boolean isEmpty() {  
    return size == 0;  
}
```

실제 구현에서는 isEmpty를 오버라이드하지 않고 Collection에 있는 함수를 가져다 씀
> 이유는 알 수 없다

#### contains
```java
@Override  
public boolean contains(Object o) {  
    return indexOf(o) >= 0;  
}
```

#### clear()
```java
@Override  
public void clear() {  
    for (Node<T> x = first; x != null; ) {  
        Node<T> next = x.next;  
        x.item = null;  
        x.next = null;  
        x.prev = null;  
        x = next;  
    }  
    first = last = null;  
    size = 0;  
}
```

- 모든 노드를 순회하며 item, next, prev필드를 null로 메모리 초기화
- first와 last 변수도 null로 초기화
- size 0으로 초기화