# ArrayList란?
- [[List]] 인터페이스의 가변 배열
- [[Vector]]와 유사하지만 동기화되어있지 않음
- `size`, `isEmpty`, `get`, `set`, `iterator`, `listIterator` 연산은 상수 시간 내에 실행 
	-> `add`의 경우 o(1)이지만 용량을 늘려줘야할 수 있기 때문에 분할 상환 상수 시간이라고 함 (Amortized)
- 연속된 자료구조이기 때문에 빈 공간을 허용하지 않음
- Wrapper 클래스이기 때문에 적은 양의 데이터가 있는 경우 [[Array]]보다 차지하는 메모리가 커짐
- [[LinkedList]]에 비해 약간 빠름
- 동기화되지 않기 때문에 스레드 환경에서 사용할 경우 다음과 같이 선언해야 함
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
|                       `clear()`, `toArray()`                       |      O(n)      |

> **조회**는 빠르지만, **삽입/삭제**는 느림'

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
