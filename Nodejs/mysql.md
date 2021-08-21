
## 21-08-21

# NodeJS에서 MySQL 사용하기

## Sequlize(시퀄라이즈)
- ORM
- 자바스크립트 객체와 데이터베이스의 릴레이션을 매핑
- MariaDB, PostgreSQL 등 다른 데이터베이스도 가능
- 자바스크립트 구문을 알아서 SQL로 변환

## 시퀄라이즈-모델 정의
- MySQL에서 정의한 테이블을 시퀄라이즈에서도 정의
- 자바스크립트 객체
- 시퀄라이즈는 모델과 MySQL의 테이블을 연결해주는 역할

## 관계 정의
- 1:N
  - `hasMany`, `belongsTo`
- 1:1
  - `hasOne`, `belongsTo`
- N:M
  - `belongsToMany`, `belongsToMany`

## 시퀄라이즈 쿼리
- 호출 : `모델명.함수명` => `User.findAll()`
- INSERT INTO
  - `create`
- SELECT
  - `findAll`, `findOne`
  - 특정 컬럼만 : `attributes` 속성
  - 조건 where : `where` 속성
  - 프로미스 리턴
- UPDATE SET
  - `update`
  - 조건 where : `where` 속성
- DELETE FROM
  - `destory`
  - 조건 where : `where` 속성
- 관계 쿼리
  - JOIN 기능
  - `include` 속성

## 직접 SQL 쿼리
  - 시퀄라이즈 쿼리를 사용하지 않을 시 직접 SQL문 작성도 가능
  - `sequelize.query('SELECT * FROM USERS);`
