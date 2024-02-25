[[Paging]] + [[Segmentation]]
### 특징
1. segmentation을 paging하는 것이기 때문에 물리적인 메모리에는 page 형태로 올라감 -> 동일한 크기
2. 페이지의 크기는 segmentation의 크기에 비례
3. segment 마다 page table이 존재 (process 마다 X)
### 순서
![](https://i.imgur.com/mUcbTbE.png)
1. STBR로 찾은 세그먼트 테이블의 s번째 위치에 사상된 <segment-length, page-table-base>를 찾는다.
2. offset d와 세그먼트 테이블에서 찾은 segment length와 비교하였을 때 범위가 적합하지 않다면 트랩
	> segment length는 구성되어 있는 페이지의 개수
3. 범위가 적합하다면 오프셋 d를 <page-number, page table-offset>으로 분할한다.
	>  `p = d / 페이지 크기`
	>  `d' = d % 페이지 크기`
1. segment table에서 찾은 page table base를 통해 segment에 할당된 page table을 찾고 p번째 위치에 사상된 프레임 번호를 찾는다.
2. 프레임 번호 f + 오프셋 d'를 통해 메모리에서 물리적 주소의 위치를 찾는다.