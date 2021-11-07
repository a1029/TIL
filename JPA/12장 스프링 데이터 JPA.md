# 12장 스프링 데이터 JPA

## **스터디 6주차**

- 김한빈, 이정준, 권영기
- 2021-11-06 ~ 2021-11-08
- 교재 : 자바 ORM 표준 JPA 프로그래밍
- 장소: 온라인 ZOOM

## 1. 스프링 데이터 JPA 소개

- 스프링 프레임워크에서 JPA를 편리하게 사용할 수 있도록 지원하는 프로젝트
- 공통 인터페이스 제공, 스프링 데이터 JPA가 인터페이스 구현 객체를 동적으로 생성해서 주입

```java
public interface ItemRepository extends JpaRepository<Item, Long> {
}

public interface MemberRepository extends JpaRepository<Member, Long> {
	Member findByUsername(String username);
}
```

### 스프링 데이터 프로젝트

- 스프링 데이터 JPA는 스프링 데이터 프로젝트의 하위 프로젝트 중 하나.
- 스프링 데이터 프로젝트는 JPA, 몽고DB, REDIS, HADOOP, NEO4J 등 다양한 데이터 저장소에 대한 접근을 추상화

## 2. 스프링 데이터 JPA 설정

- XML, 어노테이션 두 가지 설정 가능
- base package를 설정하여야 함. 설정하면 스프링 데이터 JPA는 base package에 있는 리포지토리 인터페이스들을 찾아서 해당 인터페이스를 구현한 클래스를 동적으로 생성한 다음 스프링 빈으로 등록

## 3. 공통 인터페이스 기능

![1.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/651e95cc-2787-4113-a44f-f85f0753ecfb/1.png)

- save(S) : 새로운 엔티티는 저장하고 이미 있는 엔티티는 수정한다.
- delete(T) : 엔티티 하나를 삭제한다. 내부에서 EntityManager.remove() 호출
- findOne(ID) : 엔티티 하나를 조회한다. 내부에서 EntityManager.find() 호출
- getOne(ID) : 엔티티를 프록시로 조회한다. 내부에서 EntityManager.getReference() 호출
- findAll(...) : 모든 엔티티를 조회한다. 정렬이나 페이징 조건을 파라미터로 제공 가능

## 4. 쿼리 메소드 기능

메소드 이름만으로 쿼리를 생성하는 기능으로써, 적절한 JPQL 쿼리를 생성해서 실행

- 메소드 이름으로 쿼리를 생성하는 기능
- 메소드 이름으로 JPA NamedQuery 호출
- @Query 어노테이션을 사용해서 리포지토리 인터페이스에 직접 쿼리 정의

### 메소드 이름으로 쿼리 생성

```java
public interface MemberRepository extends Repository<Member, Long> {
	// select m from Member m where m.email = ?1 and m.name = ?2
	List<Member> findByEmailAndName(String email, String name);
}
```

- 정해진 규칙이 있으므로 막 지어서는 안됨

### JPA NamedQuery

메소드 이름으로 Named 쿼리를 호출할 수 있음

```java
@Entity
@NamedQuery(
	name="Member.findByUsername",
	query="select m from Member m where m.username = :username"
)
public class Member { ... }

public interface MemberRepository extends JpaRepository<Member, Long> {
	List<Member> findByUsername(@Param("username") String username);
}
```

- 선언한 "도메인 클래스 + . + 메소드 이름" 으로 Named 쿼리를 찾아서 실행
- 실행한 Named 쿼리가 없으면 메소드 이름으로 쿼리 생성 전략 사용

### @Query

직접 쿼리를 리포지토리 메소드에 지정하는 방법

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
	@Query("select m from Member m where m.username = ?1")
	Member findByUsername(String name);

	@Query(value = "SELECT * FROm MEMBER WHERE USERNAME = ?0", nativeQuery = true)
	Member findByUsername(Stirng username);
}
```

### 벌크성 수정 쿼리

@Modifying 어노테이션 사용하면 됨

```java
@Modifying
@Query("update Product p set p.price = p.price * 1.1 where p.stockAmount < :stockAmount")
int bulkPriceUp(@Param("stockAmount") String stockAmount);
```

### 페이징과 정렬

```java
Page<Member> findByName(String name, Pageable pageable); // count 쿼리 사용

List<Member> findByName(String name, Pageable pageable); // count 쿼리 사용 안함

List<Member> findByName(String name, Sort sort);

public interface MemberRepository extends Repository<Member, Long> {
	Page<Member> findByNameStrtingWith(String name, Pageable Pageable);
}

PageRequest pageRequest = new PageRequest(0, 10, new Sort(Direction.DESC, "name"));

Page<Member> result = memberRepository.findByNameStartingWith("김", pageRequest);

List<Member> members = result.getContent(); // 조회된 데이터
int totalPages = result.getTotalPages(); // 전체 페이지 수
boolean hasNextPage = result.hasNextPage(); // 다음 페이지 존재 여부
```

- Pageable을 구현한 PageRequest를 사용해야 함

### 힌트

- JPA 쿼리 힌트를 사용하려면 @QueryHints 어노테이션을 사용하면 됨

```java
@QueryHints(value = { @QueryHint(name = "org.hibernate.readOnly",
	value = "true")}, forCounting = true)
Page<Member> findByName(String name, Pageable pageable);
```

### Lock

- 쿼리시 락을 걸려면 @Lock 어노테이션을 사용하면 됨

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
List<Member> findByName(String name);
```

## 5. 명세

- 도메인 주도 설계
- 명세를 이해하기 위한 핵심 단어는 술어. 데이터를 검색하기 위한 제약 조건 하나하나를 술어라 할 수 있음
- 이 술어를 스프링 데이터 JPA는 Specification 클래스로 정의

```java
public interface OrderRepository extends JpaRepository<Order, Long>,
JpaSpecificationExecutor<Order> {
}
```

- 명세 기능을 사용하려면 JpaSpecificationExecutor 인터페이스를 상속받으면 됨

```java
import static org.springframework.data.jpa.domain.Specifications.*; // where()
import static jpabook.jpashop.domain.spec.OrderSpec.*;

public List<Order> findOrders(String name) {
	List<Order> result = orderRepository.findAll(
		where(memberName(name)).and(isOrderStatus())
	);

	return result;
}
```

- 명세를 사용하는 코드

```java
public class OrderSpec {
	public static Specification<Order> memberName(final String memberName) {
		return new Specification<Order>() {
			public Predicate toPredicate(Root<Order> root,
				CriteriaQuery<?> query, CriteriaBuilder builder) {

				if(StringUtils.isEmpty(memberName)) return null;

				Join<Order, Member> m = root.join("member", JoinType.INNER);
				return builder.equal(m.get("name"), memberName);
			}
		};
	}

	public static Specification<Order> isOrderStatus() {
		return new Specification<Order>() {
			public Predicate toPredicate(Root<Order> root,
				CriteriaQuery<?> query, CriteriaBuilder builder) {

				return builder.equal(root.get("status"), OrderStatus.ORDER);
			}
		};
	}
}
```

- 명세를 정의하는 코드
- 명세를 정의하려면 Specification 인터페이스를 구현하고, toPredicate(...) 메소드만 구현하면 됨
- JPA Criteria의 Root, CriteriaQuery, CriteriaBuilder 클래스가 모두 파라미터로 주어지므로 이를 활용해서 적절한 검색 조건을 반환하면 됨

## 6. 사용자 정의 리포지토리 구현

- 메소드를 추가로 직접 구현해야 할 경우
- 리포지토리 인터페이스를 직접 구현하면 공통 인터페이스가 제공하는 기능까지 모두 구현해야 함, 스프링 데이터 JPA는 다른 방법을 제공

```java
public interface MemberRepositoryCustom {
	public List<Member> findMemberCustom();
}

public class MemberRepositoryImpl implements MemberRepositoryCustom {
	@Override
	public List<Member> findMemberCustom() {
		... // 사용자 정의 구현
	}
}

public interface MemberRepository extends JpaRepository<Member, Long>,
	MemberRepositoryCustom {
}
```

- 사용자 정의 인터페이스를 작성해야 함
- 사용자 정의 인터페이스를 구현한 클래스를 작성해야 함
- 구현 클래스 이름을 지을 때 리포지토리 인터페이스 이름 + Impl로 지어야 함. 이렇게 하면 스프링 데이터 JPA가 사용자 정의 구현 클래스로 인식
- 리포지토리 인터페이스에서 사용자 정의 인터페이스를 상속받으면 됨

## 7. Web 확장

스프링 MVC에서 사용할 수 있는 편리한 기능을 제공

SpringDataWebConfiguraion을 XML에서 빈으로 등록하거나 @EnableSpringDataWebSupport을 사용

### 도메인 클래스 컨버터 기능

- HTTP 파라미터로 넘어온 엔티티의 아이디로 엔티티 객체를 찾아서 바인딩함

```java
@Controller
public class MemberController {
	@RequestMapping("member/memberUpdateForm")
	public String memberUpdateForm(@RequestParam("id" Member member, Model model) {
		model.addAttribute("member", member);
		return "member/memberSaveForm";
	}
}
```

### 페이징과 정렬 기능

```java
@RequestMapping(valu = "/members", method = RequestMethod.GET)
public String list(Pageable pageable, Model model) {
	Page<Member> page = memberService.findMembers(pageable);
	model.addAttribute("members", page.getContent());
	return "members/memberList";
}
```

- 사용해야 할 페이징 정보가 둘 이상이면 접두사를 사용해서 구분, @Qualifier 어노테이션 사용
- Pageable 기본값을 수정하려면 @PageableDefault 어노테이션 사용

## 8. 스프링 데이터 JPA가 사용하는 구현체

스프링 데이터 JPA가 제공하는 공통 인터페이스는 SimpleJpaRepository 클래스가 구현

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID extends Serializable>
	implements JpaRepository<T, ID>, JpaSpecificationExecutor<T> {

	@Transactional
	public <S extends T> S save(S entity) {
		...
	}
	...
}

```

- @Repository :적용 JPA 예외를 스프링이 추상화환 예외로 변환
- @Transactional 적용 : JPA의 모든 변경은 트랜잭션 안에서 이루어져야 함. SimpleJpaRepository에 Transaction 처리가 되어있으므로 개발자는 서비스 계층에서 트랜잭션 처리를 하지 않아도 됨. 물론 서비스 계층에서 트랜잭션을 시작했으면 리포지토리도 해당 트랜잭션을 전파받아서 그대로 사용
- 데이터를 변경하지 않는 트랜잭션에 사용하는 readOnly 옵션은 플러시를 생략해서 약간의 성능 향상 가능
- save() 메소드 : 저장할 엔티티가 새로운 엔티티면 저장하고 이미 있는 엔티티면 병합함. 새로운 엔티티를 판단하는 기본 전략은 엔티티의 식별자로 판단. 식별자가 객체이고 null 이거나, 자바 기본 타입이고 숫자 0이면 새로운 엔티티로 판단

## 9. JPA 샵에 적용

### 명세 적용

1. 리포지토리 인터페이스는 JpaSpecificationExecutor 상속, 상속을 받으면 Specification을 매개변수로 받는 여러 메소드가 상속됨
2. 명세 구현 (Specification, Predicate)
3. 검색조건을 가지고 있는 OrderSearch 객체에 자신이 가진 검색조건으로 Specification을 생성하도록 코드 추가
4. 서비스에서 리포지토리에 검색 조건을 넘길 때 Specification을 넘기도록 수정

## 10. 스프링 데이터 JPA와 QueryDSL 통합

스프링 데이터 JPA는 2가지 방법으로 QueryDSL을 지원

### QueryDslPredicateExecutor 사용

```java
public interface ItemRepository extends JpaRepository<Item, Long>,
	QueryDslPredicateExecutor<Item> {
}

QItem item = QItem.item;
Iterable<Item> result = itemRepository.findAll (
	item.name.contains("장난감").and(item.price.between(10000, 20000))
);
```

- QueryDslPredicateExecutor를 상속받으면 됨
- QueryDSL을 검색조건(매개변수)로 사용하는 여러 메소드를 상속받음
- 이 기능은 한계가 있음. join, fetch는 사용할 수 없음
- QueryDSL가 제공하는 다양한 기능을 사용하려면 JPAQuery를 직접 사용하거나 QueryDslRepositorySupport를 사용해야 함

### QueryDslRepositorySupport

- QueryDslRepositorySupport를 상속받아야 함
- from, delete, update, ... 등 메소드를 상속받음
- 사용자 정의 리포지토리 구현 클래스의 생성자에서 QueryDslRepositorySupport에 엔티티 클래스 정보를 넘겨주어야 함

## 정리

- 스프링 데이터 JPA덕분에 데이터 접근 계층의 코드가 많이 줄었음
- 스프링 프레임워크와 JPA를 함께 사용한다면 스프링 데이터 JPA는 필수
