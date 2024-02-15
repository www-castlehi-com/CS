**Non-Uniform Memory Access** : 불균일 기억 장치 접근
# 등장 배경
초기 CPU는 일반적으로 메모리보다 천천히 동작
1960년대에 슈퍼 컴퓨터, 고속 컴퓨터가 개발되며 이 속도가 역전
이 이후부터 CPU는 메모리에서 데이터를 다 가지고 올 때까지 기다려야 했음
메모리에 접근하는 경우의 수를 줄이기 위해 캐시 메모리를 지속적으로 증가시키는 것과 캐시 미스를 줄이기 위한 알고리즘을 발전시킴
하지만, [[운영체제]]의 크기의 비약적인 증가와 애플리케이션의 크기 증가는 캐시를 통한 성능 향상을 압도함
특히, 다중 프로세서일 경우에도 하나의 프로세서만이 메모리에 접근할 수 있기 때문에 대기 상태는 더 심각
따라서 각각의 프로세서에 독립적인 별도의 메모리를 제공하는 NUMA system 등장
# 원리
![](https://i.imgur.com/nhYHO6f.png)
- 램 모듈(Memory Bank)이 CPU 노드마다 별도로 존재
- 여러 개의 CPU 코어를 가진 NUMA 노드가 해당 노드에 직접 연결된 RAM을 가지며, 이를 로컬메모리로 사용
- 다른 노드가 타 로컬메모리에 접근하기 위해서 원격 접근이 필요하며, 로컬 메모리 접근보다 느림