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
