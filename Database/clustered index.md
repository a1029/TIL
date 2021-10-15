# clusterd index
![f090211](https://user-images.githubusercontent.com/15135565/137471809-582572b7-8231-48e0-bc26-ef356570c768.gif)
-   클러스터드 인덱스는 인덱스가 아님
-   b+tree와 비슷한 구조
-   페이지 단위로 노드 구성
-   노드의 값은 키순서대로 저장됨
-   노드의 row는 key-pointer 구조로 이루어짐

### 논리프노드

-   논리프노드는 pointer에 다음 페이지 번호를 저장함

### 리프노드

-   리프노드는 실제 데이터를 가짐

# secondary index
