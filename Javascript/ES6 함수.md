
## 21-08-18

함수의 구분, 메서드 정의의 모호성

일반함수

메서드

화살표 함수

# ES6 함수
- ES6 사양에서 메서드는 메서드 축약 표현으로 정의된 함수만을 의미
- non-constructor
- 내부 슬롯 `[[HomeObject]]`을 가짐. super키워드 사용 가능 

<br>

### 콜백 함수 내부의 this 문제
- 콜백 함수의 this와 외부 함수의 this가 서로 다른 값을 가리킨다는 문제
- this회피, 인수로 this 전달, bind 사용 등
- 화살표 함수로도 해결 가능

# 화살표 함수

- non-constructor
- this, arguments, super, new.target 바인딩을 갖지 않음
- 화살표 함수 내부에서 this를 참조하면 상위 스코프의 this를 참조. (lexical this)
- call, apply, bind로도 화살표 함수 내부의 this를 교체할 수 없음
- super, arguments도 마찬가지로 상위 스코프에서 참조

## Rest 파라미터
- 가변 인자
- 매개변수 이름 앞 `...` 을 붙여서 정의
- arguments는 유사 배열 객체, 배열 사용시 arguments 대신 Res t파라미터를 쓰면 더 간편

## 매개변수 기본 값