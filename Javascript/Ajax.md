## 2021-09-26

# Ajax
- 비동기 자바스크립트 통신
- `XMLHttpRequest` 객체
- HTML 전체를 다시 렌더링하는 전통적인 방식에서 탈피할 수 있게 함
- 필요한 데이터만 즉각적으로 요청/응답

## JSON
- 클라이언트와 서버 간의 HTTP 통신을 위한 텍스트 데이터 포맷
- 자바스크립트에 종속되지 않는 언어 독립형 데이터 포맷
- 객체 리터럴과 유사하게 키와 값으로 구성된 순수한 텍스트
- `JSON.stringify`, 직렬화
- `JSON.parse`, 역직렬화

## XMLHttpRequest
- XMLHttpRequest 객체는 브라우저에서 제공하는 Web API이므로 브라우저 환경에서만 실행
- HTTP 요청 전송
- HTTP 응답 처리