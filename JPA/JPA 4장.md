# JPA 4장

## 스터디 2주차
- 김한빈, 이정준, 권영기
- 2021-10-08 ~ 2021-10-10
- 교재 : 자바 ORM 표준 JPA 프로그래밍
- 장소: 온라인 ZOOM

## @Entity

- 엔티티 객체로 만들 때 사용, JPA에서 테이블과 매핑할 클래스, JPA가 관리
- 기본 생성자 필수, `final, enum, interface, inner` 클래스에는 사용 X
- 필드에 `final` 사용 X

## @Table

- 엔티티와 매핑할 테이블을 지정
- 옵션 : uniqueConstraints

## @Column

- 컬럼 설정
- 옵션 : nullable, length 등

## @Enumerated, @Temporal, @Lob

- 자바의 Enum, 자바의 Date, 자바 String(길이 제한이 없음)
- @Column과 차이점

## 데이터베이스 스키마 자동 생성

- persistence.xml에 [hibernate.hbm2ddl.auto](http://hibernate.hbm2ddl.auto) 옵션을 주면 테이블을 자동 생성하도록 설정할 수 있다.
- 테이블을 생성할 때 여러 전략을 설정할 수 있다.
    - create : 기존 테이블을 삭제하고 새로 생성 (DROP + CREATE)
    - create-drop : create에 추가로 애플리케이션을 종료할 때 생성한 테이블을 제거 (DROP+CREATE+DROP)
    - update : 테이블과 엔티티 매핑정보를 비교해서 변경 사항만 수정
    - validate : 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고만 남기고 애플리케이션을 실행하지 않음
    - none : 자동 생성 기능을 사용하지 않음 none 자체는 유효하지 않는 옵션으로, [hibernate.hbm2ddl.auto](http://hibernate.hbm2ddl.auto) 속성 자체를 삭제하거나 유효하지 않는 옵션을 주면 됨
- 개발서버와 테스트 서버, 운영 서버에서 사용할 때 적절한 전략을 주의있게 사용해야 한다. 만약 운영서버에서 create 전략을 사용한다면 매우 큰 문제가 발생할 수 있다.

## @Id

- 기본키 설정
- 데이터베이스마다 기본키 설정 방법이 다르기 때문에 JPA에서도 기본키에 대한 여러 전략을 제공함
- `Entity.setId("id1")` : 직접 할당
- IDENTITY : MySQL 의 AUTO_INCREMENT, 데이터베이스에 저장이 되고 난 후에야 식별자를 얻을 수 있기 때문에 쓰기 지연을 할 수 없음
- SEQUENCE : 오라클의 SEQUENCE, 시퀀스를 미리 만들어야 함. 만들어둔 시퀀스를 엔티티에 매핑하고, 컬럼에 적용까지 해주어야 함, IDENTITY와 반대로 식별자를 먼저 얻음
- TABLE : 오라클의 SEQUENCE와 비슷하나 시퀀스 대신 키 생성 전용 테이블을 하나 만들어 시퀀스를 흉내내는 전략
    - TABLE
        
        ```sql
        create table MY_SEQUENCES (
        	sequence_name varchar(255) not null, // => 컬럼명 기본값
        	next_val bigint, // => 컬럼명 기본값
        	primary key ( sequence_name )
        )
        ```
        
        ```java
        @Entity
        @TableGenerator(
        	name = "BOARD_SEQ_GENERATOR",
        	table = "MY_SEQUENCES",
        	pkColumnValue = "BOARD_SEQ", allocationSize = 1)
        public class Board {
        	@Id
        	@GeneratedValue(strategy = GenerationType.TABLE, generator = "BOARD_SEQ_GENERATOR")
        	private Long id;
        	...
        }
        ```
        
- AUTO : 데이터베이스에 따라 JPA가 자동으로 전략을 선택해줌. 데이터베이스를 변경해도 JPA 코드는 수정할 필요가 없다는 것이 장점.

## 자연키와 대리키

- 나는 auto_increment처럼 1씩 증가하는 숫자값을 기본키로 설정하는 것(대리키 방식)이 불편해서 했는데, 자연키 방식이 좋다고 생각했는데, 책은 정반대의 입장을 가지고 있다.
- 이메일, 주민번호, 휴대폰 번호등을 PK로 할 경우 이 값은 미래에 바뀔 수 있다. 그래서 기본키로 적당하지 않다. 따라서 대리키를 권장했다.

## 매핑의 다양한 옵션들

- p145~153

## @Access

- JPA가 엔티티 데이터에 접근하는 방식 지정
- 필드 접근
- 프로퍼티 접근