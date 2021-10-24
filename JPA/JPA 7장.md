# JPA 7장

## **스터디 4주차**

-   김한빈, 이정준, 권영기
-   2021-10-22 ~ 2021-10-24
-   교재 : 자바 ORM 표준 JPA 프로그래밍
-   장소: 온라인 ZOOM

## 1. 상속 관계 매핑

### 1) 조인 전략

![1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ef7c9e8a-4f75-48c1-a199-c5745a94e660/1.png)

-   엔티티 각각을 모두 테이블로 만듬
-   자식 테이블이 부모 테이블의 기본 키를 받아서 기본 키 + 외래 키로 사용
-   타입을 구분하는 컬럼 필요
-   코드

    ```java
    @Entity
    @Inheritance(strategy = InheritanceType.JOINED) // 매핑 전략 => 조인 전략
    @DiscriminatorColumn(name = "DTYPE") // 부모 클래스에서 구분 컬럼
    public abstract class Item {
    	@Id @GeneratedValue
    	@Column(name = "ITEM_ID")
    	private Long id;

    	private String name;
    	private int price;
    	...
    }

    @Entity
    @DiscriminatorValue("A") // 구분 컬럼에 입력할 값
    @PrimaryKeyJoinColumn(name = "BOOK_ID") // ID 재정의
    public class Album extends Item {
    	private String artist;
    	...
    }

    @Entity
    @DiscriminatorValue("M")
    public class Moive extends Item {
    	private String director;
    	private String actor;
    	...
    }

    ```

-   장점
    -   테이블의 정규화
    -   외래 키 참조 무결성 제약조건 활용 가능
    -   저장공간의 효율적 사용
-   단점
    -   조회할 때 조인을 사용하므로 성능 저하 가능성
    -   조회 쿼리가 복잡
    -   데이터 등록 시 INSERT SQL 두 번 실행

### 2) 단일 테이블 전략

![2.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/148112c0-f2c9-4826-b217-e7c90727ce25/2.png)

-   테이블 하나만 사용
-   구분 컬럼을 꼭 사용
-   코드

    ```java
    @Entity
    @Inheritance(strategy = InheritanceType.SINGLE_TABLE) // 매핑 전략 => 단일 테이블 전략
    @DiscriminatorColumn(name = "DTYPE")
    public abstract class Item {
    	@Id @GeneratedValue
    	@Column(name = "ITEM_ID")
    	private Long id;

    	private String name;
    	private int price;
    	...
    }
    ...
    ```

-   장점
    -   조인이 필요 없다 ⇒ 조회 성능이 빠름
    -   조회 쿼리도 단순
-   단점
    -   자식 엔티티가 매핑한 컬럼은 null이 허용되야 함
    -   단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있음 ⇒ 상황에 따라 조회 성능 문제

### 3) 구현 클래스마다 테이블 전략

![3.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a702b3d6-5cae-437f-b32a-5762602a7216/3.png)

-   조인 전략과 비슷하나 부모 테이블이 없고 부모 테이블의 컬럼을 자식들이 모두 가지고 있음
-   데이터베이스 설계자와 ORM 전문가 둘 다 추천하지 않음
-   구분 컬럼을 사용하지 않음
-   코드

    ```java
    @Entity
    @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS) //=> 구현 클래스마다 테이블 전략
    public abstract class Item {
    	@Id @GeneratedValue
    	@Column(name = "ITEM_ID")
    	private Long id;

    	private String name;
    	private int price;
    	...
    }
    ...
    ```

-   장점
    -   서브 타입을 구분해서 처리할 때 효과적
    -   not null 제약조건 사용 가능
-   단점
    -   여러 자식 테이블을 함께 조회할 때 성능이 느림
    -   자식 테이블의 통합 관리가 어려움

## 2. @MappedSuperclass

![4.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f0965564-b18a-45e1-9b11-34d9114b349f/4.png)

![5.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/59eec22a-1712-46dd-8a3e-b9e7cb497216/5.png)

-   부모 클래스는 테이블과 매핑하지 않고 부모 클래스를 상속받는 자식 클래스에게 매핑 정보만 제공하고 싶을 때 사용
-   엔티티는 있지만 테이블은 생성되지 않음
-   코드

    ```java
    @MappedSuperclass
    public abstract class BaseEntity {
    	@Id @GeneratedValue
    	private Long id;
    	private String name;
    	...
    }

    @Entity
    public class Member extends BaseEntity {
    	private String email;
    	...
    }
    ...
    ```

-   @MappedSuperclass로 지정한 클래스는 엔티티가 아니므로 `em.find()`나 JPQL에서 사용할 수 없음
-   부모로부터 상속받은 매핑 정보 재정의 ⇒ `@AttributeOverride, @AttributeOverrides`
-   연관관계 재정의 ⇒ `@AssociationOverrides, @AssociationOverride`
-   특히 등록날짜, 수정날짜 등에 자주 쓰임

## 3. 복합 키와 식별 관계 매핑

### 1) 식별관계

![6.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5b8cbda1-4d68-4425-a664-49de81636ae6/6.png)

-   부모 테이블의 기본 키를 자식 테이블의 기본 키 + 외래 키로 사용하는 관계

### 2) 비식별 관계

![7.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/60032369-ae87-4778-b7b8-6fb2117d634b/7.png)

-   부모 테이블의 기본 키를 자식 테이블의 외래키로만 사용하는 관계
-   필수적 비식별 관계 : 외래 키에 NULL을 허용하지 않음, 연관관계 맺음 필수
-   선택적 비식별 관계 : 외래 키에 NULL을 허용, 연관관계를 맺지 않아도 됨

### (1) 복합 키 : 비식별 관계 매핑

![8.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4f560559-95fe-4ec8-9ace-49a03e094ee1/8.png)

-   식별자를 둘 이상 사용하려면 별도 식별자 클래스 필요
-   식별자 클래스의 속성명과 엔티티 사용하는 식별자의 속성명이 같아야 함
-   `@IdClass`

    -   `Serializable` 인터페이스, `equals`, `hashCode`기본 생성자, `public`
    -   엔티티를 등록하기 직전에 내부에서 Parent.id1, Parent.id2 값을 사용해서 식별자 클래스인 parentId를 생성하고 영속성 컨텍스트의 키로 사용
    -   자식 클래스에서의 매핑
        -   자식 테이블의 외래 키도 복합 키, `@JoinColumns`, `@JoinColumn` 으로 매핑
    -   코드

        ```java
        // 부모 클래스
        @Entity
        @IdClass(ParentId.Class)
        public class Parent{
        	@Id
        	@Column(name = "PARENT_ID1")
        	private String id1; // ParentId.id1 과 연결

        	@Id
        	@Column(name = "PARENT_ID2")
        	private String id2; // ParentId.id2 과 연결

        	...
        }

        // 식별자 클래스
        public class ParentId implements Serializable{
        	private String id1; // Parent.id1 과 연결
        	private String id2; // Parent.id2 과 연결

        	public ParentId(){}
        	public ParentId(String id1,String id2){this.id1=id1;this.id2=id2;}

        	@Override
        	public boolean equals(Object o){...}

        	@Override
        	public int hashCode(){...}
        }

        Parent parent = new Parent();
        parent.setId("myid1");
        parent.setId("myid2");
        em.persist(parent);

        ParentId parentId = new ParentId("myid1","myid2");
        Parent parent = em.find(Parent.class,parent.Id);

        // 자식 클래스
        @Entity
        public class Child{
        	@Id
        	private String id;

        	@ManyToOne
        	@JoinColumns({
        		@JoinColumn(name = "PARENT_ID1", referencedColumnName = "PARENT_ID1"),
        		@JoinColumn(name = "PARENT_ID2", referencedColumnName = "PARENT_ID2")
        	})
        	private Parent parent;
        }
        ```

-   `@EmbeddedId`

    -   조금 더 객체지향적인 방법
    -   Parent 엔티티에서 식별자 클래스를 직접 사용하고 `@EmbeddedId` 어노테이션을 붙임
    -   식별자 클래스에는 `@Embeddable`을 붙이고 식별자 클래스에서 기본 키를 매핑
    -   `Serializable` 인터페이스, `equals`, `hashCode`기본 생성자, `public`
    -   사용할 때 식별자 클래스를 생성해서 사용
    -   코드

        ```java
        // 부모 클래스
        @Entity
        public class Parent{
        	@EmbeddedId
        	private ParentId id;

        	...
        }

        // 식별자 클래스
        @Embeddable
        public class ParentId implements Serializable{
        	@Column(name = "PARENT_ID1")
        	private String id1;
        	@Column(name = "PARENT_ID2")
        	private String id2;

        	// equals and hashCode
        	...
        }

        // Parent 저장
        Parent parent = new Parent();
        ParentId parentId = new ParentId("myid1","myid2");
        parent.setId(parentId);
        em.persist(parent);
        // Parent 조회
        ParentId parentId = new ParentId("myId1","myId2");
        Parent parent = em.find(Parent.class,parentId);
        ```

-   `@IdClass` , `@EmbeddedId` 는 취향에 맞게 사용, EmbeddedId는 JPQL에서 더 길어질 수 있음
    -   코드
        ```java
        em.creaeteQuery("select p.id.id1,p.id.id2 from Parent p"); // @EmbeddedId
        em.creaeteQuery("select p.id1,p.id2 from Parent p"); // @IdClass
        ```

### (2) 복합 키 : 식별 관계 매핑

![9.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c6a9fad3-5ac1-4297-bf59-b548be72b9dc/9.png)

-   부모부터 손자까지 계속해서 기본 키를 전달
-   `@IdClass`

    -   코드

        ```java
        // 부모 클래스
        @Entity
        public class Parent{
        	@Id @Column(name = "PARENT_ID")
        	private String id;
        	...
        }

        // 자식 클래스
        @Entity
        @IdClass(ChildId.class)
        public class Child{
        	@Id
        	@ManyToOne
        	@JoinColumn(name = "PARENT_ID")
        	private Parent parent;

        	@Id @Column(name = "CHILD_ID")
        	private String childId;
        	...
        }

        // 자식 ID (식별자 클래스)
        public class ChildId implements Serializable{
        	private String parent; // Child.parent 매핑
        	private String childId; // Child.childId 매핑

        	// equals, hashCode
        	...
        }

        // 손자 클래스
        @Entity
        @IdClass(GrandChildId.class)
        public class GrandChild{
        	@Id
        	@ManyToOne
        	@JoinColumns({
        		@JoinColumn(name = "PARENT_ID"),
        		@JoinColumn(name = "CHILD_ID")
        	})
        	private Child child;

        	@Id @Column(name = "GRANDCHILD_ID")
        	private String id;

        	...
        }

        // 손자 ID
        public class GrandChildId implements Serializable{
        	private ChildId child; // GrandChild.child 매핑
        	private String id; // GrandChild.id 매핑

        	// equals, hashCode
        	...
        }
        ```

-   `@EmbeddedId`

    -   코드

        ```java
        // 부모
        @Entity
        public class Parent{
        	@Id @Column(name = "PARENT_ID")
        	private String id;
        	...
        }

        // 자식
        @Entity
        public class Child{
        	@EmbeddedId
        	private ChildId id;

        	@MapsId("parentId") // ChildId.parentId 매핑
        	@ManyToOne
        	@JoinColumn(name = "PARENT_ID")
        	public Parent parent;

        	...
        }

        // 자식 ID
        @Embeddable
        public class ChildId implements Serializable{
        	private String parentId; // @MapsId("parentId")로 매핑

        	@Column(name = "CHILD_ID")
        	private String id;

        	// equals, hashCode
        	...
        }

        // 손자
        @Entity
        public class GrandChild{
        	@EmbeddedId
        	private GrandChildId id;

        	@MapId("childId") // GrandCHildId.childId 매핑
        	@ManyToOne
        	@JoinColumns({
        		@JoinColumn(name = "PARENT_ID"),
        		@JoinColumn(name = "CHILD_ID")
        	})
        	private Child child;

        	...
        }

        // 손자 ID
        @Embeddable
        public class GrandChildId implements Serializable{
        	private ChildId childId; // @MapsId("childId") 매핑

        	@Column(name = GRANDCHILD_ID)
        	private String id;

        	...
        }
        ```

    -   `@MapsId` : 외래키와 매핑한 연관관계를 기본 키에도 매핑, 속성 값은 식별자 클래스의 기본 키 필드를 지정

### (3) 식별 관계를 비식별 관계로 변경

![10.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1a3bd3f3-7bbf-4407-89dd-0c703a866f2e/10.png)

-   코드

    ```java
    // 부모
    @Entity
    public class Parent{
    	@Id @GeneratedValue
    	@Column(name = "PARENT_ID")
    	private Long id;
    	private String name;
    	...
    }

    // 자식
    @Entity
    public class Child{
    	@Id @GeneratedValue
    	@Column(name "CHILD_ID")
    	private Long id;
    	private String name;

    	@ManyToOne
    	@JoinColumn(name = "PARENT_ID")
    	private Parent parent;
    	...
    }

    // 손자
    @Entity
    public class GrandChild{
    	@Id @GeneratedValue
    	@Column(name = "GRANDCHILD_ID")
    	private Long id;
    	private String name;

    	@ManyToOne
    	@JoinColumn(name = "CHILD_ID")
    	private Child child;
    	...
    }
    ```

### (4) 일대일 식별 관계

![11.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a3b8ff38-4358-43c7-a08b-3c8a9be24174/11.png)

-   자식 테이블의 기본 키 값으로 부모 테이블의 기본 키 값만 사용
-   부모 테이블의 기본 키가 복합키가 아니면 자식 테이블도 복합 키로 구성하지 않아도 됨
-   코드

    ```java
    // 부모
    @Entity
    public class Board{
    	@Id @GeneratedValue
    	@Column(name = "BOARD_ID")
    	private Long id;

    	private String title;

    	@OneToOne(mappedBy = "board")
    	private BoardDetail boardDetail;
    	...
    }

    // 자식
    @Entity
    public class BoardDetail{
    	@Id
    	private Long boardId;

    	@MapsId // BoardDetail.boardId 매핑
    	@OneToOne
    	@JoinColumn(name = "BOARD_ID")
    	private Board board;

    	private String content;
    	...
    }

    public void save(){
    	Board board = new Board();
    	board.setTitle("title");
    	em.persist(board);

    	BoardDetail boardDetail = new BoardDetail();
    	boardDetail.setContent("content");
    	boardDetail.setBoard(board); // 부모
    	em.persist(boardDetail);
    }
    ```

### (5) 식별, 비식별 관계의 장단점

-   비식별 관계를 더 선호
-   식별 관계는 자식 테이블로 기본 키를 전파하므로 점점 기본 키 컬럼이 늘어남, 조인할 때 SQL이 복잡해지고 기본 키 인덱스가 불필요
-   식별 관계는 기본 키로 비즈니스적 의미가 있는 자연 키 사용, 비즈니스 의미가 없는 대리키 사용을 권장
-   식별 관계는 복합키 클래스 사용 필요
-   대리키는 JPA에서 자동으로 생성할 수 있는 편리한 기능 제공
-   식별 관계도 장점은 있음
    -   부모 아이디가 A인 모든 자식 조회
    ```java
    // 부모 아이디가 A인 모든 자식
    SELECT * FROM CHILD
    WHERE PARENT_ID='A'
    ```
-   Integer 대신 Long을 쓰는 이유도 알게됨

## 4. 조인 테이블

### 1) 연관관계를 설계하는 방법은 크게 2가지

-   조인 컬럼 사용(외래 키), `@JoinColumn`
-   조인 테이블 사용(테이블), `@JoinTable`

![12.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/215491ec-d1ad-4dcf-83d7-15f21b09bb51/12.png)

### ===============================================

![13.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/09a692a8-171a-4f96-bfe5-d24f85d4f73b/13.png)

### (1) 일대일 조인 테이블

![14.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8ce093b8-5625-4cfc-b8c0-b6748f586d4a/14.png)

-   코드

    ```java
    // 부모
    @Entity
    public class Parent{
    	@Id @GeneratedValue
    	@Column(name = "PARENT_ID")
    	private Long id;
    	private String name;

    	@OneToOne
    	@JoinTable(name = "PARENT_CHILD", // 매핑할 조인 테이블 이름
    		joinColumns = @JoinColumn(name = "PARENT_ID"), // 현재 엔티티를 참조하는 외래 키
    		inverseJoinColumns = @JoinColumn(name = "CHILD_ID")) //  반대방향 엔티티를 참조하는 외래 키
    	private Child child;
    	...
    }

    // 자식
    @Entity
    public class Child{
    	@Id @GeneratedValue
    	@Column(name = "CHILD_ID")
    	private Long id;
    	private String name;
    	...
    }

    // 자식
    ...
    @OneToOne(mappedBy="child")
    priavte Parent parent;
    ```

-   조인 테이블의 외래 키 컬럼 각각에 총 2개의 유니크 제약조건 걸어야 함

### (2)일대다 조인 테이블

![15.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1a4b5f61-7cfb-45b0-92a3-deafb1bc4762/15.png)

-   코드

    ```java
    // 부모
    @Entity
    public class Parent{
    	@Id @GeneratedValue
    	@Column(name = "PARENT_ID")
    	private Long id;
    	private String name;

    	@OneToMany
    	@JoinTable(name = "PARENT_CHILD",
    		joinColumns = @JoinColumn(name = "PARENT_ID"),
    		inverseJoinColumns = @JoinColumn(name = "CHILD_ID"))
    	private List<Child> child = new ArrayList<Child>();
    }

    // 자식
    @Entity
    public class Child{
    	@Id @GeneratedValue
    	@Column(name = "CHILD_ID")
    	private Long id;
    	private String name;
    	...
    }
    ```

-   조인 테이블의 컬럼 중 다(N)와 관련된 컬럼인 곳에 유니크 제약조건 걸어야 함

### (3) 다대일 조인 테이블

-   코드

    ```java
    / 부모
    @Entity
    public class Parent{
    	@Id @GeneratedValue
    	@Column(name = "PARENT_ID")
    	private Long id;
    	private String name;

    	@OneToMany(mappedBy = "parent")
    	private List<Child> child = new ArrayList<Child>();
    	...
    }

    // 자식
    @Entity
    public class Child{
    	@Id @GeneratedValue
    	@Column(name = "CHILD_ID")
    	private Long id;
    	private String name;

    	@ManyToOne(optional = false)
    	@JoinTable(name = "PARENT_CHILD",
    		joinColumns = @JoinColumn(name = "CHILD_ID"),
    		inverseJoinColumns = @JoinColumn(name = "PARENT_ID"))
    	private Parent parent;
    	...
    }
    ```

### (4) 다대다 조인 테이블

![16.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49a015ff-fc0d-48d0-b8fc-d37f561de7fe/16.png)

-   코드

    ```java
    // 부모
    @Entity
    public class Parent{
    	@Id @GeneratedValue
    	@Column(name = "PARENT_ID")
    	private Long id;
    	private String name;

    	@ManyToMany
    	@JoinTable(name = "PARENT_CHILD",
    		joinColumns = @JoinColumn(name = "PARENT_ID"),
    		inverseJoinColumns = @JoinColumn(name = "CHILD_ID"))
    	private List<Child> child = new ArrayList<Child>();
    	...
    }

    // 자식
    @Entity
    public class Child{
    	@Id @GeneratedValue
    	@Column(name = "CHILD_ID")
    	private Long id;
    	private String name;
    	...
    }
    ```

-   조인 테이블의 두 컬럼을 합해서 하나의 복합 유니크 제약조건을 걸어야 함

## 5. 엔티티 하나에 여러 테이블 매핑

![17.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0094b131-a414-45ee-bfdf-ad6a18734dd3/17.png)

-   코드

    ```java
    @Entity
    @Table(name = "BOARD") // BOARD 테이블과 매핑
    @SecondaryTable(name = "BOARD_DETAIL", // 매핑할 다른 테이블의 이름
    	pkJoinColumns = @PrimaryKeyJoinColumn(name = "BOARD_DETAIL_ID")) // 다른 테이블의 기본 키명
    public class Board{
    	@Id @GeneratedValue
    	@Column(name = "BOARD_ID")
    	private Long id;

    	private String title;

    	@Column(table = "BOARD_DETAIL")
    	private String content;
    }
    ```

-   더 많은 테이블을 매핑하려면 `@SecondaryTables` 사용
-   항상 두 테이블 이상을 조회하므로 최적화하기 어려움
