
## 21-09-05

# 배열
- 자바스크립트의 배열도 객체
    - `typeof [] => object`
- length 프로퍼티
- 밀집배열, 희소배열

## 배열 생성
- 배열 리터럴
- Array 생성자 함수
- Array.of
- Array.from

배열 요소의 참조, 배열 요소의 추가와 갱신, 배열 요소 삭제

## 배열 메서드
- isArray
- Array.prototype
  - indexOf, push, pop, unshift, concat, splice, slice, join, fill, includes, flat

## 배열 고차 함수
- Array.prototype
  - sort, forEach, map, filter, reduce, some, every, find, findIndex, flatMap

# Number
- 표준 빌트인 객체

## Number 생성자 함수
- new Number(),
  - Number()와의 차이점

## Number 프로퍼티
- EPSILON, MAX_VALUE, MIN_VALUE, MAX_SAFE_INTEGER, MIN_SAFE_INTEGER, POSITIVE_INFINITY, NEGATIVE_INFINITY, NaN

## Number 메서드
- isFinite, isInteger, isNaN, isSafeInteger
- Number.prototype
  - toExponential, toFixed, toPrecision, toString

# Math
- 표준 빌트인 객체
- 정적 프로퍼티와 정적 메서드만 제공


# Date
- 표준 빌트인 객체
- `new Date()`, `Date()`
- `new Date(milliseconds), new Date(dateString), new Date(year, month, ...)`

## Date 메서드