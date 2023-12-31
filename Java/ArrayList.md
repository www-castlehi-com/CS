[오라클 docs : ArrayList](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html)
# ArrayList란?
- [[List]] 인터페이스의 가변 배열
- `RandomAccess` 자료구조이기 때문에 인덱스를 알고 있다면 O(1) 시간에 접근 가능
- [[Vector]]와 유사하지만 동기화되어있지 않음
- `size`, `isEmpty`, `get`, `set`, `iterator`, `listIterator` 연산은 상수 시간 내에 실행 
	-> `add`의 경우 o(1)이지만 용량을 늘려줘야할 수 있기 때문에 분할 상환 상수 시간이라고 함 (Amortized)
- 연속된 자료구조이기 때문에 빈 공간을 허용하지 않음
- Wrapper 클래스이기 때문에 적은 양의 데이터가 있는 경우 [[Array]]보다 차지하는 메모리가 커짐
- [[LinkedList]]에 비해 약간 빠름
- 동기화되지 않기 때문에 스레드 환경에서 사용할 경우 다음과 같이 선언해야 함 -> 스레드 안전하지 않음
	```java
	List list = Collections.synchronizedList(new ArrayList(...));
	```
# 시간복잡도 비교
|                               메소드                               |  시간 복잡도   |
|:------------------------------------------------------------------:|:--------------:|
|                          `get(int index)`, `set(int index, E element)`                          |      O(1)      |
|                          `add(E element)`                          | O(1) Amortized |
|                    `add(int index, E element)`                     |      O(n)      |
|              `remove(int index)`, `remove(Object o)`               |      O(n)      |
| `indexOf(Object o)`, `lastIndexOf(Object o)`, `contains(Object o)` |      O(n)      |
|                              `size()`                              |      O(1)      |
|                            `isEmpty()`                             |      O(1)      |
|                       `clear()`                      |      O(n)      |

> **조회**는 빠르지만, **삽입/삭제**는 느림

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

## ArrayList
### 필드
```java
public class ArrayList<T> implements List<T> {  
  
    private static final int DEFAULT_CAPACITY = 10;  
    private static final Object[] EMPTY_ELEMENTDATA = {};  
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};  
    Object[] elementData;  
    private int size;

	//...
}
```
1️⃣ **DEFAULT_CAPACITY**
- 초기 기본 capacity를 나타냄

2️⃣ **EMPTY_ELEMENTDATA**
- 비어 있는 배열
- 초기에 아무런 용량도 할당하지 않음 -> 첫번째 요소가 추가될 때까지 실제 메모리 할당을 늦춤

3️⃣ **DEFAULTCAPACITY_EMPTY_ELEMENTDATA**
- 기본 capacity를 위해 사용되는 빈 배열
- 첫 원소가 추가될 때 어느 정도의 용량으로 확장해야하는지 구분하기 위해서 사용

4️⃣ **elementData**
- ArrayList의 요소가 저장되는 배열 버퍼
- `size`는 이 배열 버퍼의 길이
- `DEFAULTCAPACITY_EMPTY_ELEMENTDATA`인 빈 ArrayList는 첫 번째 요소가 추가되면 `DEFAULT_CAPACITY`로 확장됨

5️⃣ **size**
- ArrayList의 크기

### 생성자
```java
public ArrayList() {  
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;  
}  
  
public ArrayList(int initialCapacity) {  
    if (initialCapacity > 0) {  
        this.elementData = new Object[initialCapacity];  
    } else if (initialCapacity == 0) {  
        this.elementData = EMPTY_ELEMENTDATA;  
    } else {  
        throw new IllegalArgumentException("Illegal Capacity: " + initialCapacity);  
    }  
}  
  
public ArrayList(Collection<? extends T> c) {  
    Object[] a = c.toArray();  
    if ((size = a.length) != 0) {  
        if (c.getClass() == ArrayList.class) {  
            elementData = a;  
        } else {  
            elementData = Arrays.copyOf(a, size, Object[].class);  
        }  
    } else {  
        elementData = EMPTY_ELEMENTDATA;  
    }  
}
```

1️⃣ **기본 생성자**
- 초기 용량이 `10`인 빈 리스트를 생성

2️⃣ **용량 지정 생성자**
- 특정 초기 용량을 지닌 빈 리스트를 생성

3️⃣ **컬렉션 기반 생성자**
- 특정한 컬렉션을 포함하는 리스트를 생성

### 가변
```java
private Object[] grow(int minCapacity) {  
    int oldCapacity = elementData.length;  
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {  
        int newCapacity = ArraysSupport.newLength(oldCapacity, minCapacity - oldCapacity, oldCapacity >> 1);  
        return elementData = Arrays.copyOf(elementData, newCapacity);  
    } else {  
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];  
    }  
}
```

1️⃣ 비어있지 않거나
2️⃣ EMPTY_ELEMENTDATA일 경우
- **minCapacity - oldCapacity** : 최소한으로 필요한 추가 용량
- **oldCapacity >> 1** : 선호되는 증가량, 현재 크기의 절반만큼 증가

### Create
#### 맨 뒤에 원소 삽입
```java
@Override  
public boolean add(T e) {  
    if (size == elementData.length)  
        elementData = grow(size + 1);  
    elementData[size] = e;  
    size++;  
    return true;
}  
```

1️⃣ elementData가 가득 찼을 경우
- `grow()`를 이용하여 elementData의 크기를 증가시킴

2️⃣ 배열의 맨 뒤에 원소 삽입
- size 인덱스에 원소를 삽입하고 size를 1 증가시킴

#### 특정 idx에 원소 삽입
```java
@Override  
public void add(int index, T element) {  
    if (index >= size || index < 0)
        throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);  
    if (size == elementData.length)  
        elementData = grow(size + 1);  
    System.arraycopy(elementData, index, elementData, index + 1, size - index);  
    elementData[index] = element;  
    size++;  
}
```

1️⃣ 특정 index가 범위 밖일 경우
- `IndexOutOfBoundsException`을 던짐

2️⃣ elementData가 가득 찼을 경우
- `grow()`를 이용하여 elementData의 크기를 증가시킴

3️⃣ index 인덱스를 기준으로 원소를 뒤로 미룸
- `System.arraycopy`를 이용할 경우 JVM이 아닌 운영 체제의 기능을 사용하고 직접적으로 메모리에 접근하여 블록을 복사하므로 반복문보다 시간 성능이 좋음
```java
for (int i = size; i > index; i--) {
	elementData[i] = elementData[i - 1];
}
```

4️⃣ 특정 index에 원소 삽입
- index에 원소를 삽입하고 size를 1 증가시킴

### Read
#### get
```java
@Override  
public T get(int index) {  
    if (index >= size || index < 0)  
        throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);  
    return (T) elementData[index];  
}
```

1️⃣ 특정 index가 범위 밖일 경우
- `IndexOutOfBoundsException`을 던짐

2️⃣ 형변환한 index에 존재하는 원소를 반환

#### indexOf / lastIndexOf
```java
@Override  
public int indexOf(Object o) {  
    if (o == null) {  
        for (int i = 0; i < size; i++) {  
            if (elementData[i] == null) return i;  
        }  
    }  
    else {  
        for (int i = 0; i < size; i++) {  
            if (o.equals(elementData[i])) return i;  
        }  
    }  
    return -1;  
}  
  
@Override  
public int lastIndexOf(Object o) {  
    if (o == null) {  
        for (int i = size - 1; i >= 0; i--) {  
            if (elementData[i] == null) return i;  
        }  
    }  
    else {  
        for (int i = size - 1; i >= 0; i--) {  
            if (o.equals(elementData[i])) return i;  
        }  
    }  
    return -1;  
}
```

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

### Update
```java
@Override  
public T set(int index, T element) {  
    if (index >= size || index < 0)  
        throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);  
    T oldValue = (T) elementData[index];  
    elementData[index] = element;  
    return oldValue;  
}
```

1️⃣ 특정 index가 범위 밖일 경우
- `IndexOutOfBoundsException`을 던짐

2️⃣ index 원소 저장
- `oldValue`에 index에 원래 있던 원소를 저장 후 함수 종료 시 반환

3️⃣ index에 새로운 원소 저장

### Delete
#### 특정 index의 원소 삭제
```java
@Override  
public T remove(int index) {  
    if (index >= size || index < 0)  
        throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);  
    T oldValue = (T) elementData[index];  
    if (size - 1 > index)   
System.arraycopy(elementData, index + 1, elementData, index, size - 1 - index);  
    elementData[size--] = null;  
    return oldValue;  
}
```

1️⃣ 특정 index가 범위 밖일 경우
- `IndexOutOfBoundsException`을 던짐

2️⃣ index 원소 저장
- `oldValue`에 index에 원래 있던 원소를 저장 후 함수 종료 시 반환

3️⃣ 맨 끝 원소가 삭제된 것이 아닐 경우
- `System.arraycopy`를 이용하여 index + 1 ~ 끝 원소를 앞으로 하나씩 당김

4️⃣ 맨 끝 원소를 null로 변경

#### 특정 원소 삭제
```java
@Override  
public boolean remove(Object o) {  
    int i = 0;  
    found : {  
        if (o == null) {  
            for (; i < size; i++) {  
                if (elementData[i] == null) break found;  
            }  
        }  
        else {  
            for (; i < size; i++) {  
                if (o.equals(elementData[i])) break found;  
            }  
        }  
        return false;  
    }  
    if (size - 1 > i)  
        System.arraycopy(elementData, i + 1, elementData, i, size - 1 - i);  
    elementData[size--] = null;  
    return true;
}
```

1️⃣ 삭제하려는 원소 o가 `null`일 경우
- `==` 연산자를 이용하여 맨 앞에 나오는 null을 찾음

2️⃣ 삭제하려는 원소 o가 `null`이 아닐 경우
- `equals` 메소드를 이용하여 o와 같은 내용을 가진 원소 중 가장 앞의 원소를 찾음

### 기타 구현
#### size
```java
@Override  
public int size() {  
    return size;  
}
```

#### isEmpty
```java
@Override  
public boolean isEmpty() {  
    return size == 0;  
}
```

### contains
```java
@Override  
public boolean contains(Object o) {  
    return indexOf(o) >= 0;  
}
```

### clear
```java
@Override  
public void clear() {  
    for (int to = size, i = size = 0; i < to; i++) {  
        elementData[i] = null;  
    }  
}
```

- elementData는 새로운 배열로 대체되는 것이 아니고, length (할당된 공간)도 그대로 유지
- 모든 size내의 값들이 null로 설정됨

# Iterable
## Iterator 인터페이스
```java
public interface Iterator<E> {  
    boolean hasNext();  
    E next();
    default void remove() {throw new UnsupportedOperationException("remove");}  
}
```

## Iterator
### 필드
```java
private class Itr implements Iterator<T> {  
    int cursor;  
    int lastRet = -1;

	//...
}
```

1️⃣ **cursor**
- 현재 가리키고 있는 원소의 인덱스
2️⃣ **lastRet**
- 이전에 가리키고 있는 원소의 인덱스 -> `remove()`에 사용

### 생성자
```java
public Itr() {  
}
```

### hasNext
```java
@Override  
public boolean hasNext() {  
    return cursor != size;  
}
```

### next
```java
@Override  
public T next() {  
    int i = cursor;  
    if (i >= size) {  
        throw new NoSuchElementException();  
    }  
    if (i >= elementData.length) {  
        throw new NoSuchElementException();  
    }  
    cursor = i + 1;  
    return (T) elementData[lastRet = i];  
}
```

1️⃣ cursor가 size보다 크거나 같을 경우 : 원소의 개수보다 많을 경우
- `NoSuchElementException`을 던짐
2️⃣ cursor가 elementData.length보다 크거나 같을 경우 : 할당받은 메모리보다 클 경우
- `NoSuchElementException`을 던짐
3️⃣ cursor값 이동 후, 이전 값 반환
- cursor값을 한 칸 큰 값으로 이동
- 이전 값을 이전 cursor값으로 업데이트 한 후 해당 값을 반환

### remove
```java
@Override  
public void remove() {  
    if (lastRet < 0) {  
        throw new IllegalStateException();  
    }  
    try {  
        ArrayList.this.remove(lastRet);  
        cursor = lastRet;  
        lastRet = -1;  
    } catch (IndexOutOfBoundsException ex) {  
        throw new ConcurrentModificationException();  
    }  
}
```

1️⃣ lastRet이 음수일 경우 : `next()`이후 `remove()`를 한 번만 호출한 것이 아닐 경우
- `IllegalStateException()`을 반환
2️⃣ 원소 삭제 후 필드 값 업데이트
- ArrayList의 `remove(int index)`로 원소 삭제
- 원소를 한 칸씩 앞으로 당겼기 때문에 cursor를 lastRet으로 돌리고 lastRet을 -1로 초기화함으로써 `remove()`를 다시 호출하는 경우를 방지
- 외부 상황에 의해 Iterator 순회 도중 컬렉션이 변경되면 `ConcurrentModificationException()` 반환

## ListIterator 인터페이스
```java
public interface ListIterator<E> extends Iterator<E> {  
    boolean hasNext();  
    E next();  
	boolean hasPrevious();  
	E previous();  
	int nextIndex();  
	int previousIndex();  
    void remove();  
    void set(E e);  
	void add(E e);  
}
```

## ListIterator
### 생성자
```java
private class ListItr extends Itr implements ListIterator<T> {  
    ListItr(int index) {  
        super();  
        cursor = index;  
    }

	//...
}
```

1️⃣ **기본 생성자**
- Iterator의 생성자 이용
- 시작부분부터 순회 시작
2️⃣ **index 기반 생성자**
- Iterator의 기본 생성자 이용, cursor를 index로 초기화
- index부터 순회 시작

### 순회 메소드
```java
@Override  
public boolean hasPrevious() {  
    return cursor != 0;  
}  
  
@Override  
public T previous() {  
    int i = cursor - 1;  
    if (i < 0) {  
        throw new NoSuchElementException();  
    }  
    if (i >= elementData.length) {  
        throw new ConcurrentModificationException();  
    }  
  
    cursor = i;  
    return (T) elementData[lastRet = i];  
}

@Override  
public int previousIndex() {  
    return cursor - 1;  
}

@Override  
public int nextIndex() {  
    return cursor;  
}
```

Iterator의 메소드 또한 ListIterator에서 사용이 가능함

### CRUD 메소드
```java
@Override  
public void set(T t) {  
    if (lastRet < 0) {  
        throw new IllegalStateException();  
    }  
  
    try {  
        ArrayList.this.set(lastRet, t);  
    } catch (IndexOutOfBoundsException ex) {  
        throw new ConcurrentModificationException();  
    }  
}  
  
@Override  
public void add(T t) {  
    try {  
        int i = cursor;  
        ArrayList.this.add(i, t);  
        cursor = i + 1;  
        lastRet = -1;  
    } catch (IndexOutOfBoundsException ex) {  
        throw new ConcurrentModificationException();  
    }  
}
```

Iterator의 `remove()` 또한 사용 가능