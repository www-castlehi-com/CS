[오라클 docs : HashMap](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashMap.html)

# HashMap이란?
- HashTable 기반 Map 구현
- Map의 순서가 보장되지 않음
- `get()`, `put()` 에 대해 Hash 함수가 요소들을 버킷에 적절히 분산시킨다고 가정할 때, 상수 시간 성능을 제공
- 성능에 영향을 미치는 것 : **초기 용량**, **부하 계수**
    - **초기 용량** : 해시 테이블 생성 시의 버킷 수
    - **부하 계수** : 해시 테이블이 자동으로 용량을 늘리기 전에 얼마나 차있을 수 있는지 나타내는 지표
    > `항목 수 > 현재 용량 * 부하 계수` → 재해시
- 동기화되지 않기 때문에 스레드 환경에서 사용할 경우 다음과 같이 선언해야 함 -> 스레드 안전하지 않음
    ```java
    HashMap map = Collections.synchronizedMap(new HashMap(...));
    ```
- **fail-fast** : 반복자 생성 후 맵이 구조적으로 수정되면 `ConcurrentModificationException` 발생
- 일반적인 버킷 : [[LinkedList]] / 임계값 초과 : Red-Black Tree
    - 잘 분산된 해시코드일 경우 트리 버킷 사용 빈도가 낮음

# 시간복잡도 비교
|                                               메소드                                               |                        시간 복잡도                         |
|:--------------------------------------------------------------------------------------------------:|:----------------------------------------------------------:|
|                                         `size()`, `isEmpty()`                                          |                            O(1)                            |
|                                      `containsKey(Object key)`                                       |                            O(1)                            |
|                                    `conatainsValue(Object value)`                                    |                            O(n)                            |
| `get(Object key)`, `getOrDefault(Object key, V defaultValue)`, `put(K key, V value)`, `remove(Object key)` | 평균적으로 O(1)<br>→ 해시 충돌로 인해 O(n)이 될 수 있음 |
# 구현

## Map 인터페이스

```java
package Map;

import java.util.Collection;
import java.util.Set;

public interface Map<K, V> {
    int size();
    boolean isEmpty();
    boolean containsKey(Object key);
		boolean containsValue(Object value);
    V get(Object key);
		V getOrDefault(Object key, V defaultValue);
    V put(K key, V value);
    V remove(Object key);
    Set<K> keySet();
    Collection<V> values();
    Set<Map.Entry<K, V>> entrySet();

    interface Entry<K, V> {
        K getKey();
        V getValue();
        V setValue(V value);
    }
}
```

## HashMap

### 상수 필드
```java
public class HashMap<K, V> implements Map<K, V>{  
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;  
    static final int MAXIMUM_CAPACITY = 1 << 30;  
    static final float DEFAULT_LOAD_FACTOR = 0.75f;  
    static final int TREEIFY_THRESHOLD = 8;  
    static final int UNTREEIFY_THRESHOLD = 6;  
    static final int MIN_TREEIFY_CAPAITY = 64;
}
```

1️⃣ **DEFAULT_INITIAL_CAPACITY**
- **기본 초기 용량**
- 1 << 4 = **16**
- HashMap이 처음 생성될 때 지정되지 않으면 사용되는 해시 테이블의 **기본 버킷 수**
- 2의 거듭제곱

2️⃣ **MAXIMUM_CAPACITY**
- **최대 용량**
- 1 << 30 = **2^30**
- 허용되는 최대 버킷 수
- 2의 거듭제곱

3️⃣ **DEFAULT_LOAD_FACTOR**
- **기본 부하 계수**
- 0.75f
- 해시 테이블이 얼마나 차 있을 때 크기를 확장할지 결정

4️⃣ **TREEIFY_THRESHOLD**
- **트리화 임계값**
- 8
- 버킷의 노드 수가 이 값 이상이면, 연결리스트에서 트리 구조로 변환 → 성능 향상

5️⃣ **UNTREEIFY_THRESHOLD**
- **트리 해제 임계값**
- 6
- HashTable의 크기가 조정될 대, 트리 구조의 버킷이 이 값 미만의 노드를 가지면 LinkedList로 다시 변환

6️⃣ **MIN_TREEIFY_CAPACITY**
- **트리화를 위한 최소 테이블 용량**
- 64
- HashTable의 전체 용량이 미만일 경우, 버킷은 트리 구조로 변환되지 않고 HashTable의 크기가 확장됨
### Node class 필드
```java
static class Node<K, V> implements Map.Entry<K,V> {  
	final int hash;  
	final K key;  
	V value;  
	Node<K, V> next;  
  
	public Node(int hash, K key, V value, Node<K, V> next) {  
		this.hash = hash;  
		this.key = key;  
		this.value = value;  
		this.next = next;  
	}  
  
	@Override  
	public K getKey() {  
		return key;  
	}  
  
	@Override  
	public V getValue() {  
		return value;  
	}  
  
	@Override  
	public V setValue(V value) {  
		V oldValue = this.value;  
		this.value = value;  
		return oldValue;  
	}  
  
	public final int hashCode() {  
		return Objects.hashCode(key) ^ Objects.hashCode(value);  
	}  
  
	public final boolean equals(Object o) {  
		if (o == this) return true;  
  
		return o instanceof Map.Map.Entry<?, ?> e  
				&& Objects.equals(key, e.getKey())  
				&& Objects.equals(value, e.getValue());  
	}  
}
```

1️⃣ 기본 데이터 구조, HashTable의 항목
2️⃣ 변수
- **hash** : 키의 해시코드
- **key** : 키
- **value** : 값
- **next** : 같은 버킷에 있는 다음 노드를 가리키는 포인터
3️⃣ hashcode
- `Objects.hashCode()`로 각각의 변수에 대한 hashcode 반환 -> JVM 구현에 따라 달라짐
- XOR연산을 통해 해시 코드를 결합 -> Map.Entry에 대한 고유한 해시 코드 생성, 균일한 분포
	``` java
	Integer a = 3;  
	Integer b = 3;  
	  
	System.out.println(a.hashCode() == b.hashCode()); // true  
	  
	Integer c = 3;  
	Integer d = 4;  
	  
	System.out.println(c.hashCode() == d.hashCode()); // false  
	  
	String e = "kim";  
	String f = "kim";  
	  
	System.out.println(e.hashCode() == f.hashCode()); // true  
	  
	String g = "kim";  
	String h = "kin";  
	  
	System.out.println(g.hashCode() == h.hashCode()); // false  
	  
	Person p1 = new Person("kim");  
	Person p2 = p1;  
	  
	System.out.println(p1.hashCode() == p2.hashCode()); // true  
	  
	Person p3 = new Person("kim");  
	Person p4 = new Person("kim");  
	  
	System.out.println(p3.hashCode() == p4.hashCode()); // false  
	  
	Person p5 = new Person("kim");  
	Person p6 = new Person("kin");  
	  
	System.out.println(p5.hashCode() == p6.hashCode()); // false  
	  
	Person p7 = new Person("kim", 2);  
	Person p8 = new Person("kim", 3);  
	  
	System.out.println(p7.hashCode() == p8.hashCode()); // false  
	  
	Person p9 = new Person("kim", 2);  
	Person p10 = p9;  
	  
	System.out.println(p9.hashCode() == p10.hashCode()); // true  
	  
	p10 = new Person("kin", 2);  
	  
	System.out.println(p9.hashCode() == p10.hashCode()); // false
	```

### 변수 필드
```java
Node<K, V>[] table;  
Set<Map.Entry<K, V>> entrySet;  
int size;  
int threshold;  
final flaot loadFactor;
```
1️⃣ **table**
- HashMap의 Key-Value 저장
- 연결리스트 또는 트리구조
- 2의 거듭제곱
2️⃣ **entrySet**
- HashMap의 모든 Key-Value를 Set 형태로 제공
- View 형태로 제공되어 HashMap을 순회하거나 특정 작업을 수행할 때 사용
- entrySet을 통해 변경된 내용은 HashMap에도 반영
3️⃣ **size**
- HashMap에 저장된 Key-Value 쌍의 총 개수
- Map의 크기
4️⃣ **threshold**
- HashTable의 크기를 변경해야하는 시점을 결정
- `loadFacotr * capacity` 를 구체적인 숫자로 변환
5️⃣ **loadFactor**
- HashTable의 부하 계수
- 용량 대비 Key-Value 쌍이 얼마나 차있는지를 나타냄
- HashTable의 크기가 확장되는 시점을 결정
### 생성자
```java
public HashMap() {  
    this.loadFactor = DEFAULT_LOAD_FACTOR;  
}

public HashMap(int initialCapacity) {  
    this(initialCapacity, DEFAULT_LOAD_FACTOR);  
}

public HashMap(int initialCapacity, float loadFactor) {  
    if (initialCapacity < 0) {  
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);  
    }  
  
    if (initialCapacity > MAXIMUM_CAPACITY) {  
        initialCapacity = MAXIMUM_CAPACITY;  
    }  
  
    if (loadFactor <= 0 || Float.isNaN(loadFactor)) {  
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);  
    }  
  
    this.loadFactor = loadFactor;  
    this.threshold = tableSizeFor(initialCapacity);  
}

static final int tableSizeFor(int cap) {  
    int n = -1 >>> Integer.numberOfLeadingZeros(cap - 1);  
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;  
}
```
1️⃣ **기본 생성자**
- 기본 초기 용량 16 사용
- 기본 부하 계수 0.75 사용
2️⃣ **지정된 초기 버킷 생성자**
- 초기 버킷 수 지정
- 기본 부하 계수 0.75 사용
3️⃣ **지정된 초기 버킷, 부하 계수 생성자**
- 초기 버킷 수 지정
- 초기 부하 계수 지정
- `initialCapacity < 0 || loadFactor <= 0 || loadFactor == NaN` 일 경우 `IllegalArgumentException` 발생
- **tableSizeFor**
	- cap보다 크거나 같은 2의 거듭제곱 수
