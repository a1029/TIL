# JPA 6장

## **스터디 3주차**

-   김한빈, 이정준, 권영기
-   2021-10-15 ~ 2021-10-17
-   교재 : 자바 ORM 표준 JPA 프로그래밍
-   장소: 온라인 ZOOM

## 다대일

-   다대일 단방향
    ![1](https://user-images.githubusercontent.com/15135565/137611257-bf12688e-fc38-41a2-9188-ac5bebd7d941.png)

    -   회원은 Member.team으로 팀 엔티티 참조 가능
    -   팀에는 회원을 참조하는 필드가 없음

    ```java
    @Entity
    public class Member {
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;

    	private String username;

    	@ManyToOne
    	@JoinColumn(name = "TEAM_ID")
    	private Team team;
    }

    @Entity
    public class Team {
    	@Id @generatedValue
    	@Column(name = "TEAM_ID")
    	private Long id;

    	private String name;
    }
    ```

-   다대일 양방향
    ![2](https://user-images.githubusercontent.com/15135565/137611352-cae7ddbd-c6ba-48c3-8105-b6b4d01ce2c8.png)

    -   팀에도 회원을 참조하는 members 필드가 있음
    -   양방향은 외래 키가 있는 쪽이 연관관계의 주인
    -   양방향 연관관계는 항상 서로를 참조해야 함

    ```java
    @Entity
    public class Member {
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;

    	private String username;

    	@ManyToOne
    	@JoinColumn(name = "TEAM_ID")
    	private Team team;

    	public void setTeam(Team team) {
    		this.team = team;

    		// 무한루프에 빠지지 않게 체크
    		if(!team.getMembers().contains(this)){
    			team.getMembers().add(this);
    		}
    	}
    }

    @Entity
    public class Team {
    	@Id @generatedValue
    	@Column(name = "TEAM_ID")
    	private Long id;

    	private String name;

    	@OneToMany(mappedBy = "team")
    	private List<Member> members = new ArrayList<Member>();

    	// 무한루프에 빠지지 않게 체크
    	public void addMember(Member member) {
    		this.members.add(member);
    		if (member.getTeam() != this){
    			member.setTeam(this);
    		}
    	}
    }
    ```

## 일대다

-   일대다 단방향
    <img width="944" alt="3" src="https://user-images.githubusercontent.com/15135565/137611264-36cb7546-356f-410e-a217-84f4e080c39e.png">

    -   Team 엔티티가 MEMBER 테이블의 외래키를 관리한다.
    -   Member 엔티티에 외래 키를 매핑할 수 있는 참조 필드가 없기 때문
    -   Team.members로 반대편 테이블인 MEMBER의 외래키를 관리

    ```java
    @Entity
    public class Team {
    	@Id @GeneratedValue
    	@Column(name = "TEAM_ID")
    	private Long id;

    	private String name;

    	@OneToMany(mappedBy = "team")
    	@JoinColumn(name = "TEAM_ID")
    	private List<Member> members = new ArrayList<Member>();
    }

    @Entity
    public class Member {
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;

    	private String username;
    }
    ```

    -   일대다 단방향 관계 매핑은 @JoinColumn을 명시해야 함 그렇지 않으면 JPA는 조인 테이블 전략을 기본으로 사용해서 매핑
    -   단점

        -   UPDATE SQL 가 추가 실행 필요

        ```java
        Member member1 = new Member("member1");
        Member member2 = new Member("member2");

        Team team1 = new Team("team1");
        team1.getMembers().add(member1);
        team1.getMembers().add(member2);

        em.persist(member1);
        em.persist(member2);
        em.persist(team1);
        ```

        ```sql
        insert into Member (MEMBER_ID, username) values (null, ?)
        insert into Member (MEMBER_ID, username) values (null, ?)
        insert into Team (TEAM_ID, name) values (null, ?)
        update Member set TEAM_ID=? where MEMBER_ID=?
        update Member set TEAM_ID=? where MEMBER_ID=?
        ```

        -   Member 엔티티는 Team 엔티티를 모름. Mebmer 엔티티를 저장할 때 Member 테이블의 TEAM_ID 외래키에 아무값도 저장되지 않음. 대신 Team 엔티티를 저장할 때 Team.members의 참조 값을 확인해서 Member 테이블의 외래키를 업데이트 해야 함

-   일대다 양방향

    -   존재하지 않음, 다대일 양방향 매핑 사용해야 함
    -   완전히 불가능한 건 아님. 속임수로 사용 가능
    -   일대다 단방향 매핑 반대편에 다대일 단방향 매핑을 추가
        <img width="990" alt="4" src="https://user-images.githubusercontent.com/15135565/137611270-a909a3b6-c206-468a-b22a-50b734c5354d.png">


    ```java
    @Entity
    public class Member {
    	...

    	@ManyToOne
    	@JoinColumn(name="TEAM_ID", insertable=false, updatable=false)
    	private Team team;
    }
    ```

    -   다만 이렇게하면 둘다 같은 키를 관리하므로 문제가 발생할 수 있어, 읽기 속성으로 설정

## 일대일

-   일대일 관계는 그 반대도 일대일 관계
-   일대일 관계는 주 테이블이나 대상 테이블 둘 중 어느 곳이나 외래키를 가질 수 있다.
-   일대일 관계는 주 테이블이나 대상 테이블 중에 누가 외래 키를 가질지 선택해야 한다.
-   주 테이블에 외래 키
    -   주 객체가 대상 객체를 참조하는 것처럼 주 테이블에 외래 키를 두고 대상 테이블을 참조
    -   객체지향 개발자들이 선호
-   대상 테이블에 외래 키
    -   전통적인 데이터베이스 개발자들이 선호
    -   테이블 관계를 일대일에서 일대다로 변경할 때 테이블 구조 유지 가능
-   주 테이블에 외래 키

    -   주 테이블인 MEMBER가 LOCKER_ID 외래키를 가짐
    -   주 엔티티인 Member 안에서 외래키 매핑
    -   단방향
        ![5](https://user-images.githubusercontent.com/15135565/137611282-0a2760ed-5160-4b61-ab6a-92f0b20283ad.png)
    -   양방향
        <img width="946" alt="6" src="https://user-images.githubusercontent.com/15135565/137611285-077cb3f9-4839-498e-881f-f50059284fde.png">

        ```java
        @Entity
        public class Member {
        	@Id @GeneratedValue
        	@Column(name="MEMBER_ID")
        	private Long id;

        	private String username;

        	@OneToOne
        	@JoinColumn(name="LOCKER_ID")
        	private Locker locker;

        @Entity
        public class Locker {
        	@Id @GeneratedValue
        	@Column(name ="LOCKER_ID")
        	private Long id;

        	private String name;

        	@OneToOne(mappedBy="locker") // 주인 설정
        	private Member member;
        }
        ```

-   대상 테이블에 외래 키

    -   대상 테이블인 LOCKER가 MEMBER_ID 외래키를 가짐
    -   대상 엔티티인 Locker 안에서 외래키 매핑
    -   단방향
        -   일대일 관계 중 대상 테이블에 외래 키가 있는 단방향 관계는 JPA에서 지원 X
            ![7](https://user-images.githubusercontent.com/15135565/137611289-f3ae89c1-2834-4a79-a0b5-eb2431c46de5.png)

        -   단방향 관계를 Locker에서 Member 방향으로 수정하거나, 양방향 관계로 만들고 Locker를 연관관계의 주인으로 설정해야 함
    -   양방향
        <img width="888" alt="8" src="https://user-images.githubusercontent.com/15135565/137611291-68623deb-0a01-49db-9493-6a8d509bdbb4.png">


        ```java
        public class Member{
        	...
        	@OneToOne(mappedBy = "member")
        	private Locker locker;
        	...
        }

        public class Locker {
        	...
        	@OneToOne
        	@JoinColumn(name="MEMBER_ID")
        	private Member member;
        	...
        }
        ```

-   지연로딩이 불가, 프록시

## 다대다

-   데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없음
-   일대다, 다대일 관계로 풀어내는 연결 테이블 사용
    <img width="995" alt="9" src="https://user-images.githubusercontent.com/15135565/137611293-565bb285-bd8c-49c6-ae70-14cd628478f9.png">
-   객체는 연결객체가 필요없음. 객체 2개로 다대다 관계 가능, (컬렉션이 있으므로)
    ![10](https://user-images.githubusercontent.com/15135565/137611296-d29fe761-929a-460d-890d-f5fa8e9648ef.png)
-   다대다 단방향

    ```java
    @Entity
    public class Member {
    	@Id @Column(name="MEMBER_ID")
    	private String id;

    	private String username;

    	@ManyToMany
    	@JoinTable(
    		name="MEMBER_PRODUCT",
    		joinColumns = @JoinColumn(name="MEMBER_ID"),
    		inverseJoinColumns = @JoinColumn(name="PRODUCT_ID")
    	)
    	private List<Product> products = new ArrayList<Product>();
    	...
    }

    @Entity
    public class Product {
    	@Id @Column(name="PRODUCT_ID")
    	private String id;

    	private String name;
    	...
    }
    ```

    -   Member_Product 엔티티를 만들지 않았다.

-   다대다 양방향

    ```java
    @Entity
    public class Product {
    	@Id @Column(name="PRODUCT_ID")
    	private String id;

    	private String name;

    	@ManyToMany(mappedBy = "products")
    	private List<Member> members;       // Product 엔티티에도 추가
    	...
    }
    ```

-   연결 엔티티 사용 (복합키)

    -   연결 테이블에 주문 수량 컬럼, 주문한 날짜 등 컬럼이 더 필요할 수 있음
        <img width="1208" alt="11" src="https://user-images.githubusercontent.com/15135565/137611322-73000bfa-df5c-4cbc-b714-b86b62a3f83a.png">
    -   컬럼이 추가되면 더이상 @ManyToMany는 사용할 수가 없음, 주문 엔티티나 상품 엔티티에는 추가한 컬럼들을 매핑할 수가 없기 때문
    -   연결 테이블을 만든 것처럼 연결 엔티티를 만들고 이곳에 추가한 컬럼들을 매핑
    -   그리고 엔티티 간의 관계도 테이블 관계처럼 다대다에서 일대다, 다대일 관계로 풀어야 함-
        ![12](https://user-images.githubusercontent.com/15135565/137611325-e0bbd212-7577-4b97-a5c3-55f9ed50f591.jpg)
    -   상품에서 회원상품으로 객체 그래프 탐색 기능이 필요하지 않다고 판단해서 상품 엔티티는 회원 상품 엔티티와 연관관계를 맺지 않음
        ```java
        public class Member{
        	...
        	@OneToMany(mappedBy = "member")
        	private List<MemberProduct> memberProducts;
        }
        public class Product{
        	...
        }
        ```
    -   @Id와 @JoinColumn을 동시에 사용해서 기본키+외래키를 한번에 매핑
        @IdClass를 사용해서 복합 기본 키를 매핑
        JPA에서 복합키 사용시 별도의 식별자 클래스 필요, 그리고 엔티티에 @IdClass 사용 필요
        복합키를 위한 별도의 식별자 클래스는 다음과 같은 특성이 있음

        -   복합 키는 별도의 식별자 클래스로 만들어야 함
        -   Serializable을 구현해야 함
        -   equals와 hashCode 메소드를 구현해야 함
        -   기본 생성자가 있어야 함
        -   식별자 클래스는 public이어야 함
        -   @IdClass를 사용하는 방법 외에 @Embeddedd를 사용하는 방법도 있다.

        ```java
        @Entity
        @IdClass(MemberProductId.class)
        public class MemberProduct {
        	@Id
        	@ManyToOne
        	@JoinColumn(name="MEMBER_ID")
        	private Member member;

        	@Id
        	@ManyToOne
        	@JoinColumn(name="PRODUCT_ID")
        	private Product product;

        	private int orderAmount;
        	private Date orderDate;
        }

        public class MemberProductId implements Serializable {
        	private String member;
        	private String product;

        	@Override
        	public booolean equals(Object o) { ... }
        	@Override
        	public int hashCode { ... }
        ```

        ```java
        public void find(){
        	MemberProductId memberProductId = new MemberProductId();
        	memberProductId.setMember("member1");
        	memberProductId.setProduct("productA");

        	MemberProduct memberProduct = em.find(MemberProduct.class, memberProductId);

        	Member member = memberProduct.getMember();
        	Product product = memberProduct.getProduct();

        	System.out.println(...);
        }
        ```

    -   위와 같다시피 복합키를 사용 하는 방법은 매우 복잡하다. 복합키를 위한 식별자 클래스도 만들어야 하고, @IdClass 사용해야하고, 식별자 클래스에 equals, hashCode도 구현해야 하고, 이렇게 만든 식별자 클래스를 사용까지 해야 한다.

-   연결 엔티티 사용 (새로운 기본키)

    -   복합키 전략은 사용이 복잡했다.
    -   데이터베이스에서 자동 생성해주는 대리키를 연관관계 테이블의 새로운 기본키로 설정하는 방법을 권장한다.
        ![13](https://user-images.githubusercontent.com/15135565/137611330-95795dd3-d7aa-4e14-afdb-ceffabde8be1.png)
    -   매핑이 더 단순하고 이해가 쉬워졌다. 회원 엔티티와 상품 엔티티는 변경사항이 없다.

        ```java
        @Entity
        public class Order {
        	@Id @GeneratedValue
        	@Column(name="ORDER_ID")
        	private Long id;   // 대리키로 새로운 기본키 설정

        	@ManyToOne
        	@JoinColumn(name="MEMBER_ID")
        	private Member member;

        	@ManyToOne
        	@JoinColumn(name="PRODUCT_ID")
        	private Product product;

        	private int orderAmount;
        	private Date orderDate;
        }
        ```

    -   사용이 매우 단순해졌다. 식별자 클래스가 필요 없어졌다.

        ```java
        public void find(){
        	Long orderId = 1L;
        	Order order = em.find(Order.class, orderId);

        	Member member = order.getMember();
        	Product product = order.getProduct();

        	System.out.println(...);
        }
        ```

-   식별/비식별
    -   식별 관계 :받아온 식별자를 기본 키 + 외래 키로 사용함
    -   비식별 관계 : 받아온 식별자는 외래 키로만 사용하고 새로운 식별자 추가
