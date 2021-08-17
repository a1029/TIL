
## 21-08-17

# strict mode
암묵적 전역

# 빌트인 객체

표준 빌트인 객체, 호스트 객체, 사용자 정의 객체

원시값과 래퍼 객체

### 빌트인 전역 프로퍼티
- `Infinity`, `NaN`, `undefined`

### 빌트인 전역 함수
- `eval`, `isInfinity`, `isNaN`, `parseFloat`, `parseInt`, 
- `encodeURI`, `decodeURI`, `encodeURIComponent`, `decodeURIComponent`  

# this

자바스크립트의 this는 자바나 C++ 같은 클래스 기반 언어에서의 this와는 조금 다르다.

자바스크립트의 this는 함수가 호출되는 방식에 따라 this에 바인딩될 값, 즉 this 바인딩이 동적으로 결정된다.

### 호출 방식
1. 일반 함수 호출
2. 메서드 호출
3. 생성자 함수 호출
4. `Function.prototype.apply/call/bind`