# JPA 5장

## **스터디 3주차**

-   김한빈, 이정준, 권영기
-   2021-10-15 ~ 2021-10-17
-   교재 : 자바 ORM 표준 JPA 프로그래밍
-   장소: 온라인 ZOOM

## 단방향 연관관계

<img width="650" alt="1" src="https://user-images.githubusercontent.com/15135565/137611407-7cb322c7-f989-4b05-b2b1-b2d7d979880b.png">

-   객체 연관관계 : 단방향
-   테이블 연관관계 : 양방향
-   외래 키 하나로 양방향 조인

    ```sql
    SELECT *
    FROM MEMBER M
    JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID

    SELECT *
    FROM TEAM T
    JOIN MEMBER M ON T.TEAM_ID = M.TEAD_id
    ```

-   객체의 양방향
    ```java
    class A {
    	B b;
    }
    class B {
    	A a;
    }
    A.b, B.a
    ```
-   객체 관계 매핑

    ![2](https://user-images.githubusercontent.com/15135565/137611167-1eab0ebe-a9d4-44b6-9c6f-8aa2a275f9a2.png)

    ```java
    @Entity
    public class Member {

    	@Id
    	@Column(name = "MEMBER_ID")
    	private String id;

    	private String username;

    	@ManyToOne
    	@JoinColumn(name = "TEAM_ID")  // 생략할 경우, team_TEAM_ID 로 외래키 매핑
    	private Team team;

    	public void setTeam(Team team) {
    		this.team = team;
    }

    @Entity
    public class Team {

    	@Id
    	@Column(name = "TEAM_ID")
    	private String id;

    	private String name;
    }
    ```

## 연관관계 사용

-   저장

    ```java
    Team team1 = new Team("team1", "팀1");
    em.persist(team1);

    Member member1 = new Member("member1", "회원1");
    member1.setTeam(team1);
    em.persist(member1);
    ```

-   조회

    ```java
    // 객체 그래프
    Member member = em.find(Member.class, "member1");
    Team team = member.getTeam();

    // JPQL
    String jpql = "select m from Member m join m.team t where t.name=:teamName";
    List<Member> resultList = em.createQuery(jpql, Member.class)
    	.setParameter("teamName", "팀1")
    	.getResultList();
    ```

-   수정

    ```java
    Team team2 = new Team("team2", "팀2");
    em.persist(team2);

    Member member = em.find(Member.class, "member1");
    member.setTeam(team2);

    // em.update()가 없다.
    ```

-   제거

    ```java
    Member member1 = em.find(Member.class, "member1");
    member1.setTeam(null);

    // 연관된 엔티티 삭제, 외래키 제약조건
    member1.setTeam(null);
    em.remove(team);
    ```

## 양방향 연관관계

![3](https://user-images.githubusercontent.com/15135565/137611173-a6b19fb9-ff6a-488b-90ae-36475eed4b15.png)

-   객체 관계 매핑

    ```java
    @Entity
    public class Team {

    	@Id
    	@Column(name = "TEAM_ID")
    	private String id;

    	private String name;

    	@OneToMany(mappedBy = "team")
    	private List<Member> members = new ArrayList<Member>();
    }
    ```

## 연관관계의 주인

-   mappedBy
-   테이블은 외래 키 하나로 두테이블의 연관관계를 관리
-   엔티티를 양방향으로 매핑하면 회원 → 팀, 팀 → 회원 두 곳에서 서로를 참조
-   객체의 참조는 둘인데, 외래 키는 하나. 여기서 차이가 발생
-   따라서 두 엔티티 객체 중 하나를 정해서 테이블의 외래키를 관리, 이를 연관관계의 주인
-   연관관계의 주인만이 외래 키를 관리할 수 있음
    ![4](https://user-images.githubusercontent.com/15135565/137611183-2e4cca70-1e7c-40a0-b5da-510b8dc40dd8.png)

-   TEAM_ID 외래키를 Member.team으로 가지고 있는 Member 엔티티가 연관관계의 주인
-   Team 엔티티는 주인이 될 수 없음

## 양방향 연관관계 저장

-   단방향 연관관계에서 저장하는 코드와 완전히 같다.

```java
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");
member1.setTeam(team1);          // 연관관계 설정 member1 -> team1
em.persist(member1);

team1.getMembers().add(member1); // 무시
```

## 양방향 연관관계 주의점

-   주인인 곳과 주인이 아닌 곳

```java
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");
// member1.setTeam(team1);       // 연관관계 설정 member1 -> team1
em.persist(member1);

team1.getMembers().add(member1); // 주인이 아닌 곳에서 연관관계 설정 => 무시
```

-   순수한 객체까지 고려

    ```java
    Team team1 = new Team("team1", "팀1");
    em.persist(team1);

    Member member1 = new Member("member1", "회원1");
    member1.setTeam(team1);           // 연관관계 설정 member1 -> team1
    team1.getMembers().add(member1);  // 연관관계 설정 team1 -> member1
    em.persist(member1);
    ```

-   연관관계 편의 메소드(리팩토링)

    ```java
    @Entity
    public class Member {

    	...

    	@ManyToOne
    	@JoinColumn(name = "TEAM_ID")  // 생략할 경우, team_TEAM_ID 로 외래키 매핑
    	private Team team;

    	public void setTeam(Team team) {
    		this.team = team;
    		team.getMembers().add(this);
    }
    ```

-   연관관계 편의 메소드 주의사항

    ![5](https://user-images.githubusercontent.com/15135565/137611190-5b6a9cd9-1f7d-465b-b147-2378b14e97ae.png)

    ```java
    @Entity
    public class Member {

    	...

    	public void setTeam(Team team) {
    		if (this.team != null){
    			this.team.getMembers.remove(this);
    		}
    		this.team = team;
    		team.getMembers().add(this);
    }
    ```

## 정리

-   단방향 매핑만으로 테이블과 객체의 연관관계 매핑은 이미 완료
-   단방향을 양방향으로 만들면 반대방향으로 객체 그래프 탐색 기능이 추가
-   양방향 연관관계를 매핑하려면 객체에서 양쪽 방향을 모두 관리

## 참고

-   연관관계의 주인을 정하는 기준
    -   단방향은 항상 외래 키가 있는 곳을 기준으로 매핑
    -   연관관계의 주인은 외래 키의 위치와 관련해서 매핑, 비즈니스 중요도로 접근하면 안됨
-   양뱡항 매핑시에 무한 루프에 빠지지 않게 주의
    -   JSON 직렬화 문제
    ```java
    member:Member {
    	team:Team {
    		members:List [
    			{
    				member:Member {
    					team:Team {
    						members:List [
    							{
    								member:Member...
    							}
    						]
    					}
    				}
    			}
    		]
    	}
    }
    ```
