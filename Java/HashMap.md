[오라클 docs : HashMap](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/HashMap.html)
# HashMap이란?
# 시간복잡도 비교
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
    V get(Object key);  
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