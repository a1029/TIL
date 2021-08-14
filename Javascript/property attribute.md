
## 21-08-14

# 프로퍼티 어트리뷰트

내부 슬롯, 내부 메서드

프로퍼티 어트리뷰트, 프로퍼티 디스크립터 객체

데이터 프로퍼티 어트리뷰트
|프로퍼티 어트리뷰트|프로퍼티 디스크립터 객체의 프로퍼티|설명|
|------|---|---|
|[[Value]]|value|프로퍼티 값|
|[[Writable]]|writable|프로퍼티 변경 가능 여부|
|[[Enumerable]]|enumerable|프로퍼티 열거 가능 여부|
|[[Configurabe]]|configurable|프로퍼티 재정의 가능 여부|

접근자 프로퍼티 어트리뷰트
|프로퍼티 어트리뷰트|프로퍼티 디스크립터 객체의 프로퍼티|설명|
|------|---|---|
|[[Get]]|get|프로퍼티 접근시 getter 호출|
|[[Set]]|set|프로퍼티 저장시 setter 호출|
|[[Enumerable]]|enumerable|프로퍼티 열거 가능 여부|
|[[Configurabe]]|configurable|프로퍼티 재정의 가능 여부|

확인방법 : `Object.getOwnPropertyDescriptor()`

프로퍼티 정의 : `Object.defineProperty()` `Object.defineProperites()`


## 객체 변경 방지

객체 확장 금지 : `Object.preventExtensions()`

객체 밀봉 : `Object.seal()`

객체 동결 : `Object.freeze()`


# 생성자 함수에 의한 객체 생성

템플릿 역할을 할 수 있어 여러개를 생성하는 경우 객체 리터럴 방식보다 유용  

`new Object(), new String(), ...`

`function Circle(){ ... } => const circle1 = new Circle()`

new 연산자 사용, 인스턴스 생성과 this 바인딩

내부 메서드 [[Call]]과 [[Construct]]

constructor, non-constructor

new.target
