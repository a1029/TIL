
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
- Nodejs/mysql.md 참조