# List란?
- 정수 **인덱스**를 사용해 특정 위치의 요소에 접근할 수 있는 자료구조
- 요소들을 **정렬된 순서**로 유지하는 자료구조 (사용자가 정한 순서를 그대로 유지)
- **중복**된 요소를 허용하는 자료구조

# List의 특징
- 인덱스 기반으로 접근
	- `get(int index)`
	- `set(int index, E element)`
	- `add(int index, E element`
	- `remove(int index)`
- 이터레이터 제공
	- `Iterator` : 제거 가능한 반복자. 단방향
	- `ListIterator` : 삽입, 교체, 제거 가능한 반복자. 양방향

# List의 종류
- [[ArrayList]]
- [[LinkedList]] -> Deque
- [[Vector]] -> Stack

### ArrayList vs LinkedList
#### 접근 시간
- **ArrayList** : O(1)
- **LinkedList** : O(n)
#### 삽입, 삭제 시간
- **ArrayList** : O(n) -> 요소 이동
- **LinkedList** : O(n) -> O(1)이지만 검색에 O(n)
#### 메모리 사용량
- **ArrayList** : 각 요소에 대한 메모리 오버헤드가 적지만 배열 크기를 조정하는 과정에서 일시적인 메모리 사용량 증가
- **LinekedList** : 각 요소가 추가적인 포인터를 가지므로 메모리 오버헤드 큼

#### 공간 복잡도
- **ArrayList** : O(n)
- **LinkedList** : O(n)