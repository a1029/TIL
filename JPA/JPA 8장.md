# JPA 8장

## **스터디 4주차**

-   김한빈, 이정준, 권영기
-   2021-10-22 ~ 2021-10-24
-   교재 : 자바 ORM 표준 JPA 프로그래밍
-   장소: 온라인 ZOOM

## 1. 프록시

### 1) 프록시 기초

-   회원 엔티티와 팀 엔티티는 연관관계
-   회원의 정보만 사용하고 팀의 정보는 필요하지 않을 때

![1](https://user-images.githubusercontent.com/15135565/138593074-ea7826fb-956e-441e-acff-8956f5b0e874.png)

-   데이터베이스를 조회하지 않고 실제 엔티티 객체도 생성하지 않음
-   데이터베이스 접근을 위임한 프록시 객체를 반환

![2](https://user-images.githubusercontent.com/15135565/138593077-d53e8c42-86af-41fc-9178-c6a5ed896114.png)

![3](https://user-images.githubusercontent.com/15135565/138593081-c6690329-d5a0-4101-95bb-0843647f392b.png)

-   프록시 클래스는 실제 클래스를 상속, 겉모양이 같음
-   실제 객체에 대한 참조(target)를 보관
-   프록시 객체의 초기화 : `member.getName()` 처럼 실제 사용될 때 데이터베이스 조회 후 실제 엔티티 객체 생성
-   프록시 객체
    ```java
    // MemberProxy 반환
    Member member = em.getReference(Member.class,"id1");
    member.getName(); // 1. getName

    // 프록시 클래스 예상
    class MemberProxy extends Member{
    	Member target = null; // 실제 엔티티 참조

    	public String getName(){
    		if(target==null){
    			// 2. 초기화 요청
    			// 3. DB 조회
    			// 4. 실제 엔티티 생성 및 참조 보관
    			this.target = ...;
    		}
    		// 5. target.getName();
    		return target.getName();
    	}
    }
    ```
-   프록시 특징
    -   프록시 객체는 처음 사용할 때 한 번만 초기화
    -   프록시 객체를 초기화한다고 프록시 객체가 실제 엔티티로 바뀌는 것은 아님
    -   프록시 객체는 원본 엔티티를 상속받은 객체이므로 타입 체크 주의 필요
    -   영속성 컨텍스트에 찾는 엔티티가 이미 있으면 데이터베이스를 조회할 필요가 없음, `em.getReference()` 를 호출해도 프록시가 아닌 실제 엔티티 반환
    -   초기화는 영속성 컨텍스트의 도움을 받아야 가능하다. 따라서 준영속 상태의 프록시를 초기화하면 문제 발생

### 2) 프록시와 식별자

-   엔티티 접근 방식을 프로퍼티로 설정하면 프록시 객체 초기화 X
-   필드로 설정하면 프록시 객체 초기화

### 3) 프록시 확인

-   초기화되지 않은 프록시 인스턴스는 false를 반환, 이미 초기화되었거나 프록시 인스턴스가 아닌 경우 true 반환
-   프록시 초기화 확인
    ```java
    boolean isLoad = em.getEntityManagerFactory()
    	.getPersistenceUnitUtil()
    	.isLoaded(entity);
    ```

## 2. 즉시 로딩과 지연 로딩

-   JPA는 엔티티 조회 시점을 선택할 수 있도록 즉시 로딩과 지연 로딩 제공
-   프록시 객체는 연관된 엔티티를 지연 로딩할 때 사용

### 1) 즉시 로딩

![4](https://user-images.githubusercontent.com/15135565/138593083-e3dab8ba-058c-4315-af43-f23c6fb2a162.png)

-   엔티티를 조회할 때 연관된 엔티티도 함께 조회
-   즉시 로딩
    ```java
    @Entity
    public class Member{
    	...
    	@ManyToOne(fetch = FetchType.EAGER)
    	@JoinColumn(name = "TEAM_ID")
    	private Team team;
    	...
    }

    // 즉시 로딩
    Member member = em.find(Member.class,"member1");
    Team team = member.getTeam(); // 객체 그래프 탐색
    ```
-   즉시 로딩 데이터베이스 쿼리문
    ```sql
    SELECT
    	M.MEMBER_ID AS MEMBER_ID,
    	M.TEAM_ID AS TEAM_ID,
    	M.USERNAME AS USERNAME,
    	T.TEAM_ID AS TEAM_ID,
    	T.NAME AS NAME
    FROM
    	MEMBER M LEFT OUTER JOIN TEAM T
    		ON M.TEAM_ID=T.TEAM_ID
    WHERE
    	M.MEBER_ID='member1'
    ```
-   JPA 구현체는 즉시 로딩을 최적화하기 위해 가능하면 조인쿼리 사용

### 2) 지연 로딩

![5](https://user-images.githubusercontent.com/15135565/138593088-4d1b36ef-0a2d-41c6-9a62-eee6a94a38b4.png)

-   연관된 엔티티를 실제 사용할 때 조회
-   지연 로딩
    ```java
    @Entity
    public class Member{
    	...
    	@ManyToOne(fetch = FetchType.LAZY)
    	@JoinColumn(name = "TEAM_ID")
    	private Team team;
    	...
    }

    // 지연 로딩
    Member member = em.find(Member.class,"member1");
    Team team = member.getTeam(); // 객체 그래프 탐색
    team.getName(); // 팀 객체 실제 사용
    ```
-   지연 로딩 데이터베이스 쿼리문
    ```sql
    SELECT * FROM MEMBER WHERE MEMBER_ID = 'member1';

    SELECT * FROM TEAM WHERE TEAM_ID = 'team1';
    ```

### 3) 즉시 로딩, 지연 로딩 정리

-   처음부터 연관된 엔티티를 모두 영속성 컨텍스트에 올려두는 것은 현실적이지 않음
-   그렇다고 필요할 때마다 SQL을 실행해서 연관된 엔티티를 지연로딩하는 것도 최적화 관점에서는 꼭 좋은 것은 아님
-   상황에 따라 다름

## 3. 지연 로딩 활용

![6](https://user-images.githubusercontent.com/15135565/138593092-8d76efc2-8bbb-4ea0-934c-4daebecba02d.png)

-   지연 로딩 활용
    ```java
    // Member
    @Entity
    public class Member{
    	@Id
    	private String id;
    	private String username;
    	private Integer age;

    	@ManyToOne(fetch = FetchType.EAGER)
    	private Team team;

    	@OneToMany(mappedBy = "member",fetch = FetchType.LAZY)
    	private List<Order> orders;
    	...
    }
    ```

![8](https://user-images.githubusercontent.com/15135565/138593097-1d51d3f3-573b-4b34-8a16-4e8df0ef7eea.png)

### 1) 프록시와 컬렉션 레퍼

-   컬렉션 래퍼
-   하이버네이트는 엔티티를 영속 상태로 만들 때 엔티티에 컬렉션이 있으면 컬렉션을 추적하고 관리할 목적으로 원본 컬렉션을 하이버네이트가 제공하는 내장 컬렉션으로 변경
-   엔티티를 지연 로딩하면 프록시 객체가 사용되지만 컬렉션은 컬렉션 래퍼가 지연 로딩 처리, 프록시 역할을 하는 것은 같음
-   컬렉션 레퍼
    ```java
    Member member = em.find(Member.class,"member1");
    List<Order> orders = member.getOrders();
    System.out.println("orders = "+orders.getClass().getName());
    // orders = org.hibernate.collection.internal.PersistentBag
    ```

### 2) JPA 기본 페치 전략 및 주의점

-   @ManyToOne, @OneToOne : 즉시 로딩
-   @ManyToMany, @OneToMany : 지연 로딩
-   연관된 컬렉션의 길이가 수만 건이상일 때의 문제
-   컬렉션 하나 이상을 즉시 로딩은 권장하지 않음, N+1 문제
-   컬렉션 즉시 로딩은 항상 외부 조인 사용

## 4. 영속성 전이: CASCADE

-   특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 동시에 영속 상태로 만들고 싶을 때
-   영속성 전이 기능 사용, JPA는 CASCADE 옵션으로 제공

![9](https://user-images.githubusercontent.com/15135565/138593099-6f72d96a-8017-4dfc-95f9-f4d9e54588d2.png)

### 1) 영속성 전이 : 저장

-   영속성 전이 사용 안했을 때
    ```java
    // 부모 엔티티
    @Entity
    public class Parent{
    	...
    	@OneToMany(mappedBy = "parent")
    	private List<Child> children = new ArrayList<Child>();
    	...
    }

    // 자식 엔티티
    @Entity
    public class Child {
    	@Id @GeneratedValue
    	private Long id;

    	@ManyToOne
    	private Parent parent;
    	...
    }

    Parent parent = new Parent();
    em.persist(parent);

    Child child1 = new Child();
    child1.setParent(parent);
    parent.getChildren().add(child1);
    em.persist(child1);

    Child child2 = new Child();
    child2.setParent(parent);
    parent.getChildren().add(child2);
    em.persist(child2);
    ```
-   영속성 전이 사용했을 때
    ```java
    // 부모 엔티티
    @Entity
    public class Parent{
    	...
    	@OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
    	private List<Child> children = new ArrayList<Child>();
    	...
    }

    Child child1 = new Child();
    Child child2 = new Child();

    Parent parent = new Parent();
    child1.setParent(parent);
    child2.setParent(parent);
    parent.getChildren().add(child1);
    parent.getChildren().add(child2);

    em.persist(parent);
    ```

### 2) 영속성 전이 : 삭제

-   영속성 전이 삭제
    ```java
    Parent parent = em.find(Parent.class, 1L);
    Child child1 = em.find(Child.class, 1L);
    Child child2 = em.find(Child.class, 2L);

    em.remove(child1);
    em.remove(child2);
    em.remove(parent);

    // 영속성 전이 삭제 정의했을 경우
    em.remove(parent);
    ```

### 3) 영속성 전이 종류

-   CASCADE 종류
    ```java
    public enum CascadeType{
    	ALL, // 모두 적용
    	PERSIST, // 영속
    	MERGE, // 병합
    	REMOVE, // 삭제
    	REFRESH, // REFRESH
    	DETACH // DETACH
    }

    cascade = {CascadeType.PERSIST, CascadeType.REMOVE} // 여러 속성 같이 사용 가능
    ```

## 5. 고아 객체

-   부모 엔티티와 연관관계가 끊어진 자식 엔티티
-   JPA는 고아 객체를 자동으로 삭제하는 기능 제공 = 고아 객체(ORPHAN) 제거
-   부모 엔티티의 컬렉션에서 자식 엔티티의 참조만 제거하면 자식 엔티티가 자동으로 삭제
-   고아 객체 설정
    ```java
    @Entity
    public class Parent{
    	...
    	@OneToMany(mappedBy = "parent", orphanRemoval = true)
    	private List<Child> children = new ArrayList<Child>();
    	...
    }

    Parent parent = em.find(Parent.class,id);
    parent.getChildren().remove(0); // 자식 엔티티를 컬렉션에서 제거

    // SQL 결과
    DELETE FROM CHILD WHERE ID=?

    // 모든 자식 엔티티 제거
    parent1.getChildren().clear();
    ```
