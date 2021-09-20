## 21-09-20

# DOM
- DOM, DOM API
- HTML 문서의 계층적 구조와 정보를 표현하며 이를 제어할 수 있는 API, 즉 프로퍼티와 메서드를 제공하는 트리 자료구조

### 노드
- HTML 요소
- 노드 객체, 종류
- 상속 구조
### 요소 노드 취득
- id, 태그이름, 클래스, CSS 선택자를 이용하여 취득
### HTMLCollection, NodeList
- DOM 컬렉션 객체
- 유사 배열 객체, 이터러블
- live, non-live 객체
### 노드 탐색
- 부모, 형제, 자식 노드 등 탐색
- 공백 텍스트 노드 탐색
### 노드 정보 취득
- nodetype, nodeName 등 노드 정보 프로퍼티 사용
### 요소 노드의 텍스트 조작
- nodeValue, textContent, innerText
### DOM 조작
- innerHTML
- insertAdjacentHTML 메서드
- 노드 생성과 추가 (createElement, createTextNode, appendChild ...)
- 복수의 노드 생성/추가시 createDocumentFragment 메서드 활용
- 노드의 삽입 (insertBefore)
- 노드 이동
- 노드 복사 (cloneNode)
- 노드 교체 (replaceChild)
- 노드 삭제 (removeChild)
### 어트리뷰트
- 어트리뷰트 노드와 attributes 프로퍼티
- HTML 어트리뷰트, DOM 프로퍼티
  - HTML 어트리뷰트는 초기값, DOM 프로퍼티는 최신값
- data 어트리뷰트와 dataset 프로퍼티
### 스타일
- 인라인 스타일 조작, style 프로퍼티
- 클래스 조작, className, classList