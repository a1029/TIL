
## 21-08-21

# MySQL
- MySQL AB가 제작 => MySQL AB가 썬 마이크로시스템즈에 인수 => 썬 마이크로시스템즈가 오라클에 인수 => 오라클이 관리
- RDBMS
- MySQL Workbench (GUI)
- cmd로도 가능 (CLI)
- 데이터베이스를 MySQL에선 schema(스키마)라고 부름
- 명령의 끝에는 반드시 `;` 를 붙여야함

## command
- `create schema `nodejs` default character set utf8;`
- `create table nodejs.users (...);`
- INT, VARCHAR, TINYINT, TEXT, DATETIME, ...
- NOT NULL, PRIMARY KEY(id), FOREIGN KEY (), UNIQUE INDEX. ...
- ...

## Engine
- MyISAM
- InnoDB


# Nodejs에서 mysql 사용하기
- 시퀄라이즈(Sequelize)

## Sequlize
- ORM
- 자바스크립트 객체와 데이터베이스의 릴레이션을 매핑
- MariaDB, PostgreSQL 등 다른 데이터베이스도 가능
- 자바스크립트 구문을 알아서 SQL로 변환

## Sequlize-모델 정의하기
- MySQL에서 정의한 테이블을 시퀄라이즈에서도 정의
- 시퀄라이즈는 모델과 MySQL의 테이블을 연결해주는 역할