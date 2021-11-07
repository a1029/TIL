# 2장 JPA 시작

## 스터디 1주차

- 김한빈, 이정준, 권영기
- 2021-10-01 ~ 2021-10-03
- 교재 : 자바 ORM 표준 JPA 프로그래밍
- 장소: 온라인 ZOOM

## 실습 환경

- IDE : Intellij Community
- DB : H2
- Project Management : Maven
- Framework : Hibernate

## 객체 매핑

- `@Entity`
- `@Table`
- `@Id`
- `@Column`

## persistence.xml

- JPA, Hibernate를 사용하기 위한 기본 설정
- JDBC 연결 위한 DB 아이디, 비밀번호, 벤더 설정 등..

## 엔티티 매니저

- persistence.xml => Persistence => EnetityManagerFactory => EntityManager
- 엔티티 매니저가 JPA 사용의 핵심
- 트랜잭션 관리
- 비즈니스 로직
  - C : `persist`
  - R : `find`
  - U : 없음, 엔티티 세터
  - D : `remove`
- 트랜잭션 커밋
- 트랜잭션 롤백

## JPQL

- 검색의 경우 엔티티 객체로 검색해야 하는 문제
- 객체지향 쿼리 언어
- JPQL은 엔티티 객체를 대상으로 쿼리, 클래스와 필드를 대상으로 쿼리
- SQL은 데이터베이스 테이블을 대상으로 쿼리
- JPQL은 데이터베이스 테이블을 전혀 알지 못함

## 기타

- 라이브러리와 프레임워크의 차이점
  - 라이브러리는 라이브러리가 사용자에게 종속되어 있다. 사용자는 라이브러리를 자유자재로 사용할 수 있다.
  - 프레임워크는 사용자가 프레임워크에 종속되어 있다. 사용자는 프레임워크가 정해둔 대로 사용을 해야 한다.
