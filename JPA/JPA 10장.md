# JPA 10장

## **스터디 5주차**

- 김한빈, 이정준, 권영기
- 2021-10-30 ~ 2021-10-31
- 교재 : 자바 ORM 표준 JPA 프로그래밍
- 장소: 온라인 ZOOM

# ✨ 객체지향 쿼리 소개

- 좀 더 복잡한 쿼리를 위해 등장
- 쿼리에 조건을 주고 싶을 때 엔티티만으로는 한계
- 객체지향 쿼리 JQPL
- JPA는 JPQL뿐만 아니라 네이티브 SQL, JDBC 직접 사용, MyBatis와 같은 SQL 매퍼 프레임워크 사용도 가능하다.

# 💎 JPQL

- 테이블이 아닌 엔티티 객체를 대상으로 하는 객체지향 쿼리
- SQL을 추상화해서 특정 데이터베이스 문법에 의존하지 않음
- Criteria 쿼리 : JPQL을 편하게 작성하도록 도와주는 API, 빌더 클래스 모음
- QueryDSL : JPQL을 편하게 작성하도록 도와주는 빌더 클래스 모음, 비표준 오픈소스 프레임워크

### 기본 문법

```java
String jpql = "select m from Member as m where m.username = 'kim'";
List<Member> resultList = em.createQuery(jpql, Member.class).getResultList();
```

### TypeQuery, Query

```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Member m", Member.class);
List<Member> resultList = query.getResultList();

Query query = em.createQuery("SELECT m.username m.age FROM Member m");
List resultList = query.getResultList();
for (Object o : resultList) { Object[] result = (Object[]) o; ... }
```

### 파라미터 바인딩

```java
TypedQuery<Member> query = em.createQuery(
	"SELECT m FROM Member m where m.username = :username", Member.class);
query.setParameter("username", "user1");
```

### 프로젝션

SELECT 절에 조회할 대상을 지정하는 것을 프로젝션

엔티티 프로젝션, 임베디드 타입 프로젝션, 스칼라 타입 프로젝션

**엔티티 프로젝션으로 조회한 엔티티는 영속성 컨텍스트에서 관리**

NEW 명령어로 바로 객체 매핑 가능

```java
TypeQuery<UserDTO> query = em.createQuery(
	"SELECT new jpabook.jpql.UserDTO(m.username, m.age) FROM Member m", UserDTO.class);
List<UserDTO> resultList = query.getResultList();
```

### 페이징 API

`setFirstResult(int startPosition)` : 조회 시작 위치

`setMaxResults(int maxResult)` : 조회할 데이터 수

### 집합과 정렬

`COUNT(), MAX(), MIN(), AVG(), SUM(), GROUP BY, HAVING, ORDER BY`

### JPQL 조인

```java
SELECT m FROM Member m INNER JOIN m.team t where t.name = :teamName // 연관필드 사용
SELECT m FROM Member m LEFT [OUTER] JOIN m.team t // 외부 조인
SELECT t, m FROM Team t LEFT JOIN t.members m // 컬렉션 조인
```

### 페치 조인

JPQL에서 성능 최적화를 위해 제공하는 기능

```java
select m from Member m join fetch m.team
```

![1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ceaa10c7-59ea-474f-a23a-a0cc4c5b24e2/1.png)

지연 로딩이 발생하지 않음

페치 조인과 일반 조인 차이

```java
select t
from Team t join t.members m
where t.name = '팀A'

SELECT
  T.*
FROM TEAM T
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID
WHERE T.NAME = '팀A*'
```

JPQL은 결과를 반환할 때 연관관계까지 고려하지 않고 단지 SELECT 절에 지정한 엔티티만 조회

페치 조인은 연관된 엔티티도 함께 조회

```java
select t
from Team t join fetch t.members
where t.name = '팀A'

SELECT
  T.*, M.*
FROM TEAM T
INNER JOIN MEMBER M ON T.ID=M.TEAM_ID
WHERE T.NAME = '팀A*'
```

### 경로 표현식

상태 필드 : `t.username, t.age`

단일 값 연관 필드 : `[m.team](http://m.team)` 묵시적 내부 조인 발생

컬렉션 값 연관 필드 : `m.orders`묵시적 내부 조인 발생

### 서브 쿼리

WHERE, HAVING 절에서만 사용 가능하고 SELECT, FROM에서는 사용 불가

`exists, all, any, some, in`

### 조건식

`AND, OR, NOT, >, <, <=, >=, =, Between, IN, Like, NULL ...`

### 다형성 쿼리

```java
select i from Item i where type(i) IN (Book, Movie)
select i from Item i where treat(i as Book).author = "kim"
```

### 엔티티 직접 사용

엔티티를 JPQL에서 직접 사용해도 SQL에서는 해당 엔티티의 기본 키값을 사용

```java
select count(m.id) from Member m
select count(m) from Member m

String qlString = "select m from Member m where m = :member";
List resultList = em.createQuery(qlString)
	.setParameter("member", member)
	.getResultList();

select m.*
from Member m
where m.id=?
```

### Named 쿼리: 정적쿼리

미리 정의한 쿼리에 이름을 부여해서 필요할 때 사용

```java
@Entity
@NamedQuery(
	name = "Member.findByUsername",
	query="select m from Member m where m.username = :username"
)
public class Member { ... }

List<Member> resultList = em.createNamedQuery("Member.findByUsername", Member.class)
	.setParameter("username", "회원1")
	.getResultList();
```

Named 쿼리를 어노테이션 대신 XML로도 작성 가능

# 🔫Criteria

- JPQL을 생성하는 빌더 클래스
- 문자가 아닌 프로그래밍 코드로 JPQL을 작성할 수 있다. 런타임 에러에서 컴파일 에러로 탐지 가능
- IDE 자동 완성 지원, 동적 쿼리

### 기본 문법

```java
//JPQL: select m from Member m

CriteriaBuilder cb = em.getCriteriaBuilder(); //Criteria 쿼리 빌더

//Criteria 생성, 반환타입 지정
CriteriaQuery<Member> cq = cb.createQuery(Member.class);

Root<Member> m = cq.from(Member.class);
cq.select(m);

TypedQuery<Member> query = em.createQuery(cq);
List<Member> members = query.getResultList();
```

### Criteria 쿼리 생성

```java
CriteriaQuery<Object> createQuery();
```

### 조회

```java
CriteriaQuery<T> select(Selection<? extends T> selection);
CriteriaQuery<T> multiselect(Selection<?>... selections);
CriteriaQuery<T> multiselect(List<Selection<?>> selectionList);
```

JPQL new 구문 ⇒ `cb.construct` 로 DTO 객체 매핑 지원

```java
cq.select(cb.construct(MemberDTO.class, m.get("username"), m.get("age"));
```

### 집합

```java
cb.groupBy(m.get("team".get("name"));
cq.multiselect(m.get("team").get("name"), maxAge, minAge)
	.groupBy(m.get("team").get("name")
	.having(cb.gt(minAge, 10));
```

### 정렬

```java
cb.desc(...), cb.asc(...)
```

### 조인

```java
Join<Member, Team> t = m.join("team", JoinType.INNER);
// 페치 조인
m.fetch("team", JoinType.LEFT);
```

### 서브 쿼리, IN 식, CASE 식, 파라미터 정의, 네이티브 함수 호출 등등 ..

### 동적 쿼리

Criteria를 쓰는 이유 중 하나는 동적 쿼리를 편리하게 생성할 수 있다는 것

### 메타 모델 API

코드 기반이므로 컴파일 시점에 오류를 발견할 수 있지만 100% 완벽한 건 아님

m.get("ageaaa")로 하면 컴파일 시점에 에러를 발견하지 못함

이런 것까지 처리하려면 메타 모델 API 사용

```java
cq.select(m)
	.where(cb.gt(m.<Integer>get("username"), 20));

cq.select(m)
	.where(cb.gt(m.get(Member_.age), 20));
```

메타 모델 API는 메타 모델 클래스가 필요

메타 모델 클래스는 코드 생성기를 사용하면 자동으로 생성해줌

코드 생성기는 메이븐, 그래들 같은 빌드 도구를 사용해서 실행

# 🍭 QueryDSL

- Criteria와 같이 JPQL 빌더 역할
- Criteria보다 훨씬 단순하고 사용하기 쉬움
- JPA 표준은 아님. 오픈소스 라이브러리
- Criteria의 메타 모델처럼 쿼리용 클래스 필요

### 기본 문법

```java
JPAQuery query = new JPAQuery(em);
QMember qMember = new QMember("m");
List<Member> members =
	query.from(qMember)
		.where(qMember.name.eq("회원1"))
		.orderBy(qMember.name.desc())
		.list(qMember);
```

### 기본 Q 생성

쿼리 클래스를 기본 인스턴스를 보관하여 이를 사용할 수 있음

같은 엔티티를 조인하거나 같은 엔티티를 서브쿼리에 사용하면 별칭을 지정해서 사용해야 함.

```java
QMember qMember = new QMember("m"); // 별칭 지정
QMember qMember = QMember.member; // 기본 인스턴스
```

### 검색 조건 쿼리

```java
List<Item> list = query.from(item)
	.where(item.name.eq("좋은상품").and(item.price.gt(20000)))
	.list(item);
```

### 결과 조회

`uniqueResult(), singleResult(), list()`

```java
query.from(item)
	.where(item.price.gt(20000))
	.orderBy(item.price.desc(), item.stockQuantity.asc())
	.offset(10).limit(20)
	.list(item);
```

### 그룹, 조인, 서브쿼리, 프로젝션 등 지원

### 빈 생성

UserDTO처럼 특정 객체로 받고 싶으면 빈 생성 기능 사용

```java
List<ItemDTO> result = query.from(item).list(
	Projections.bean(ItemDTO.class, item.name.as("username"), item.price));
```

### 수정, 삭제 배치 쿼리

JQPL 배치 쿼리와 같이 영속성 컨텍스트를 무시하고 데이터베이스를 직접 쿼리하므로 주의 필요

```java
JPAUpdateClause updateClause = new JPAUpdateClause(em. item);
long count = updateClause.where(item.name.eq("시골개발자의 JPA 책"))
	.set(item.price, item.price.add(100)
	.execute();
```

### 동적쿼리

BooleanBuilder를 사용하여 특정 조건에 따른 동적 쿼리를 편리하게 생성할 수 있음

```java
BooleanBuilder builder = new BooleanBuilder();
if(StringUtils.hastext(param.getname())) {
	builder.and(item.name.contains(param.getName()));
}
if(param.getPrice() != null) {
	builder.and(item.price.gt(param.getPrice()));
}
List<Item> result = query.from(item)
	.where(builder)
	.list(item);
```

### 메소드 위임

쿼리 타입에 검색 조건을 직접 정의할 수 있음

```java
public class ItemExpression {

    @QueryDelegate(Item.class)
    public static BooleanExpression isExpensive(Qltem item, Integer price) {
        return item.price.gt(price);
    }
}
```

# 🏝 네이티브 SQL

- SQL을 직접 사용할 수 있는 기능
- 특정 데이터베이스에 의존하는 기능을 사용해야 할 때 사용
  - SQL 쿼리 힌트
  - 인라인 뷰, UNION, INTERSECT
  - 스토어드 프로시저

### 네이티브 SQL 사용

```java
public Query createNativeQuery(String sqlString, Class resultClass); // 결과 타입 정의

public Query createNativeQuery(String sqlString); // 결과 타입을 정의할 수 없을 때

public Query createNativeQuery(String sqlString, String reusltSetMapping); // 결과 매핑
```

### 엔티티 조회

```java
String sql = "SELECT ID, AGE, NAME, TEAM_ID FROM MEMBER WHERE AGE > ?";
Query nativeQuery = em.createNativeQuery(sql, Member.class).setParameter(1, 20);
List<Member> resultList = nativeQuery.getResultList();
```

**네이티브 SQL로 SQL만 직접 사용할 뿐 나머지는 JPQL을 사용할 때와 같다. 조회한 엔티티는 영속성 컨텍스트에서 관리됨.**

### 결과 매핑

select에서 여러 컬럼을 조회하여 매핑이 복잡해질 때 사용

@SetResultSetMapping을 사용하여 결과 매핑 사용

```java
@Entity
@SqlResultSetMapping(name = "memberWIthOrderCount",
	entities = {@EntityResult(entityClass = Member.class)},
	columns = {@ColumnResult(name = "ORDER_COUNT")}
)
public class Member {...}

QUery nativeQuery = em.createNativeQuery(sql, "memberWithOrderCount");
```

### Named 네이티브 SQL

@NamedNativeQuery를 사용하여 JPQL처럼 네이티브 SQL도 Named 네이티브 SQL 사용 가능

사용할 때 `createNamedQuery` 메소드를 사용

Named 네이티브 쿼리를 XML에 정의할 수도 있음

```java
@Entity
@NamedNativeQuery(
	name = "Member.memberSQL",
	query = "SELECT ID, AGE, NAME, TEAM_ID FROM MEMBER WHERE AGE > ?",
	resultClass = Member.class
)
public class Member { ... }

TypedQuery<Member> nativeQuery = em
	.createNamedQuery("Member.memberSQL", Member.class)
	.setParameter(1, 20);
```

### Named 스토어드 프로시저

JPA는 스토어드 프로시저를 호출할 수 있음

Named 스토어드 프로시저 지원 : 스토어드 프로시저 쿼리에 이름을 부여해서 사용하는 것

# ✈ 객체지향 쿼리 심화

### 벌크 연산

여러 건을 한 번에 수정하거나 삭제할 때 사용

```java
String qlString =
	"update Product p set p.price = p.price * 1.1 where p.stockAmount < :stockAmount";

int resultCount = em
	.createQuery(qlString)
	.setParameter("stockAmount", 10)
	.executeUpdate();
```

벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리한다는 점에 주의

![3.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1e80aa23-2cc4-4144-9e6d-52812d5047f1/3.png)

![4.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3c528219-2f76-4d09-aad9-69d437d5aaff/4.png)

해결 방법은 `em.refresh()` 사용, 벌크 연산 먼저 실행, 벌크 연산 수행 후 영속성 컨텍스트 초기화

### 영속성 컨텍스트와 JPQL

JPQL의 조회 대상은 엔티티, 임베디드 타입, 값 타입 같이 다양한 종류가 있음

엔티티를 조회하면 영속성 컨텍스트에서 관리되지만 엔티티가 아니면 영속성 컨텍스트에서 관리되지 않음

```java
select m from Member m // 엔티티 조회 (관리 O)
select o.address from Order o // 임베디드 타입 조회 (관리 X)
select m.id, m.username from Member m // 단순 필드 조회 (관리 X)
```

**JPQL로 데이터베이스에서 조회한 엔티티가 영속성 컨텍스트에 이미 있으면 JPQL로 데이터베이스에서 조회한 결과를 버리고 대신에 영속성 컨텍스트에 있던 엔티티를 반환**

1. 새로운 엔티티를 영속성 컨텍스트에 하나 더 추가 (X)
2. 기존 엔티티를 새로 검색한 엔티티로 대체 (X)
3. 기존 엔티티는 그대로 두고 새로 검색한 엔티티를 버림 (O)

2번은 영속성 컨텍스트에 수정 중인 데이터가 사라질 수 있으므로 위험하다. 영속성 컨텍스트는 엔티티의 동일성을 보장하기 때문에 3번으로 동작

### find() vs JPQL

em.find() 메소드는 엔티티를 영속성 컨텍스트에서 먼저 찾고 없으면 데이터베이스에서 찾음

JPQL은 항상 데이터베이스에 SQL을 실행해서 결과를 조회

### JPQL과 플러시 모드

```java
em.setFlushMode(FlushModeType.AUTO); // 커밋 또는 쿼리 실행 시 플러시(기본값)
em.setFlushMode(FlushModeType.COMMIT); // 커밋시에만 플러시
```

JPQL은 영속성 컨텍스트를 고려하지 않는다. 따라서 JPQL을 실행하기 전에 영속성 컨텍스트의 내용을 데이터베이스에 반영해야 한다.

![6.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8baa821b-e0be-4967-91f8-80a0684cde4a/6.png)

플러시 모드가 AUTO일 경우 JPQL이 실행되면 플러시가 먼저 자동으로 호출되서 데이터베이스에 반영되지만, COMMIT일 경우 반영되지 않음

따라서 `em.flush()`를 호출해서 플러시해주거나, 플러시 모드를 설정해주어야 함

```java
em.setFlushMode(FlushModeType.COMMIT);

product.setPrice(2000);

// em.flush(); // 수동으로 플러시

Product product2 = em
	.createQuery("select p from Product p where p.price = 2000", Product.class)
	.setFlushMode(FlushModeType.AUTO) // 해당 쿼리에만 플러시가 된다.
	.getSingleResult();
```

### 플러시 모드와 최적화

```java
등록()
쿼리() // 플러시
등록()
쿼리() // 플러시
등록()
쿼리() // 플러시
커밋() // 플러시
```

플러시 모드가 COMMIT일 경우 맨 마지막에 1번만 플러시 된다.

최적화를 위해서는 플러시 모드도 고민해보아야 한다.

# 🏆 정리
