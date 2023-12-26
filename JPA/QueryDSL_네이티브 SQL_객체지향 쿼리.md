# **QueryDSL, 네이티브 SQL, 객체지향 쿼리**

## QueryDSL

- 쿼리를 문자가 아닌 코드로 작성해도, 쉽고 간결하며 그 모양도 쿼리와 비슷하게 개발할 수 있는 오픈 소스 프로젝트이다.
- JPA, JDO, JDBC, Lucnen, Hibernate Search, MongoDB, 자바 컬렉션 등을 다양하게 지원한다.
- 데이터를 조회하는 데 기능이 특화되어 있다.
- 엔티티를 기반으로 쿼리 타입이라는 쿼리용 클래스를 생성해야한다.

### QueryDSL 설정하는 법

우선 필요한 라이브러리가 있다.

- `querydsl-apt`: Querydsl 관련 코드 생성 기능 제공
- `querydsl-jpa`: Querydsl 라이브러리

`build.gradle`에 아래와 같이 환경 설정을 하면 된다.

```java
plugins {
   ...
   //querydsl 추가
   id "com.ewerk.gradle.plugins.querydsl" version "1.0.10"
   ...
}
dependencies {
   ...
   //querydsl 추가
   implementation 'com.querydsl:querydsl-jpa'
   ...
}
//querydsl 추가 시작
def querydslDir = "$buildDir/generated/querydsl"
querydsl {
   jpa = true
   querydslSourcesDir = querydslDir
}
sourceSets {
    main.java.srcDir querydslDir
}
configurations {
    querydsl.extendsFrom compileClasspath
}
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl
}
//querydsl 추가 끝
```

### QueryDSL 기초

```java
public static void queryDSL(EntityManager entityManager) {
    JPAQuery query = new JPAQuery(entityManager);
    QMember qMember = new QMember("m");     // 생성되는 JPQL의 별칭이 m
    List<Member> members = query
            .from(qMember)
            .where(qMember.username.eq("철수"))
            .orderBy(qMember.username.desc())
            .list(qMember);
    for (Member member : members) {
        System.out.println("Member : " + member.getMemberId() + ", " + member.getUsername());
    }
}
```

먼저 QueryDSL을 사용하려면 `com.mysema.query.jpa.impl.JPAQuery` 객체를 생성해야 한다.

`EntityManager`를 인자로 넘겨주고, 사용할 쿼리 타입(Q)를 생성하는데 생성자에는 별칭을 준다.

`list()` 같은 경우에는 내부에 조회할 대상을 지정하면 된다.

참고로 쿼리 타입은 QueryDSL 환경 설정을 수행하면 자동으로 지정한 소스 경로에 추가된다.

### JPQL과 비교

```java
select m from Member m
where m.name = ?1
order by m.name desc
List<Member> members = query
  .from(qMember)
  .where(qMember.username.eq("허코딩"))
  .orderBy(qMember.username.desc())
  .list(qMember);
```

### 기본 Q 생성

쿼리 타입(Q)은 사용하기 편리하도록 **기본 인스턴스를 보관**하고 있다.

**엔티티를 조인**하거나 **같은 엔티티를 서브쿼리에 사용**하면 같은 별칭이 사용되므로 **이때는 별칭을 직접 지정해서 사용**해야 한다.

- 쿼리 타입 사용

```java
QMember qMember = new QMember("m");     //직접 지정
QMember qMember = QMember.member;       //기본 인스턴스
```

- import static 활용

```java
import static 경로.QMember.member; // 기본 인스턴스
public static void basic(){
    EntityManager em=emf.createEntityManager();
    JPAQuery query=new JPAQuery(em);
    List<Member> members =
    query.from(member)
      .where(member.username.eq("허코딩"))
      .orderBy(member.username.desc())
      .list(member);
}
```

### 검색 조건 쿼리

```java
JPAQuery query = new JPAQuery(em);
QItem item = QItem.item; // QItem을 정적으로 커스텀 선언하여 가져올 수도 있음.
List<Item> list = query.from(item)
    .where(item.name.eq("관심 상품").and(item.price.gt(20000))
    .list(item);
```

QueryDSL의 `where` 절에는 `and`나 `or`을 사용할 수 있고(`,`을 사용하면 and 연산 적용), 조건절에는 `between`, `contains`와 같은 기능도 제공하고 있다.

```java
item.price.between(1000, 2000);     // 가격이 10000원~20000원 상품
item.name.contains("상품1");        // 상품1 이름을 포함하는 상품, like '%상품1%' 검색
item.name.startsWith("고급");       // like 검색, '고급%' 검색
```

### 결과 조회

쿼리 작성이 끝나고 **결과 조회 메서드를 호출하면 실제 데이터베이스를 조회**한다.

보통은 `uniqueResult()`, `list()`를 사용하고, 파라미터로 프로젝션 대상을 넘겨준다.

| 메서드 | 설명 |
| --- | --- |
| uniqueResult() | 조회 결과가 한 건일 때 사용한다.
조회 결과가 없으면 null을 반환, 결과가 하나 이상이면 예외가 발생한다. |
| singleResult() | uniqueResult()와 같지만, 결과가 하나 이상이면 처음 데이터를 반환한다. |
| list() | 결과가 하나 이상일 때 사용한다.
결과가 없으면 빈 컬렉션을 반환한다. |

### 페이징과 정렬

```java
QItem item = QItem.item;

query.from(item)
    .where(item.price.gt(20000))
    .orderBy(item.price.desc(), item.stockQuantity.asc())
    .offset(10).limit(20)
    .list(item);
```

| 기능 | 메서드 |
| --- | --- |
| 정렬 | orderBy 사용 - asc(), desc() |
| 페이징 | offset, limit |

참고로 페이징은 `restrict()` 메서드에 `com.mysema.query.QueryModifiers`를 파라미터로 사용해도 된다.

```java
QueryModifiers queryModifiers = new QueryModifiers(20L, 10L); // limit, offset
       List<Member> members = query.from(qMember)
               .restrict(queryModifiers)
               .list(qMember);
```

### 실제 페이징 처리

실제 페이징 처리를 하려면 검색된 전체 데이터 수를 알아야 하는데, 이 때 `listResults()`를 사용한다.

`listResults()`를 사용하면 전체 데이터 조회를 위한 `count` 쿼리를 한번 더 실행 후 `SearchResults`를 반환한다.(해당 객체에서 전체 데이터 수 조회가 가능하다.)

```java
SearchResult<Item> result =
    query.from(item)
        .where(item.price.gt(10000))
        .orderBy(item.price.desc(), item.stockQuantity.asc()) // 가격 내림차순, 재고수량 오름차순
        .offset(10).limit(20)
        .listResults(item);
long total = result.getTotal(); // 검색된 전체 데이터 수
long limit = result.getLimit();
long offset = result.getOffset();
List<Item> results = result.getResult(); // 조회된 데이터
```

### 그룹

```java
query.from(item)
    .groupBy(ite.price)
    .having(item.price.gt(1000))
    .list(item);
```

그룹화하려면 `groupBy`를 사용하고, 그룹화된 결과를 제한하려면 `having`을 사용하면 된다.

### 조인

- `innerJoin(join)`, `leftJoin`, `rightJoin`, `fullJoin`
- JPQL의 on
- 성능 최적화를 위한 fetch 조인: `join` 뒤에 `.fetch()`를 붙여준다.
    - fetch 조인이란 연관된 엔티티나 컬렉션을 한 번에 같이 조회하는 기능이다.
    - SQL 호출 횟수를 줄여 성능을 최적화한다.

```java
// 기본 조인
QOrder order = QOrder.order;
QMember member = QMember.member;
QOrderItem orderItem = QOrderItem.orderItem;

query.from(order)
    .join(order.member, member)
    .leftJoin(order.orderItems, orderItem)
    .list(order);

// 조인 on 사용
query.from(order)
    .leftJoin(order.orderItems, orderItem)
    .on(orderItem.cont.gt(2))
    .list(order);

// 페치 조인
query.from(order)
    .innerJoin(order.member, member).fetch()
    .leftJoin(order.orderItems, orderItem).fetch()
    .list(order);

// 세타 조인
qquery.from(order, member)
    .where(order.member.eq(member))
    .list(order);
```

### 서브 쿼리

`com.mysema.query.jpa.JPASubQuery`를 생성해서 사용한다.

서브 쿼리의 **결과가 하나**면 `unique()`, **여러 건**이면 `list()`를 사용할 수 있다.

```java
// 단건 조회
QItem item = QItem.item;
QItem itemSub = new QItem("itemSub"); // 같은 엔티티를 서브 쿼리에 사용 - 별칭 사용 필수
query.from(item)
    .where(item.price.eq(
            new JPASubQuery().from(itemSub)
                .unique(itemSub.price.max())
    ))
    .list(item);

// 여러 건 조회
QItem item = QItem.item;
QItem itemSub = new QItem("itemSub");
query.from(item)
  .where(item.in(
      new JPASubQuery().from(itemSub)
          .where(item.name.eq(itemSub.name))
          list(itemSub)
  ))
  .list(item);
```

### ****프로젝션과 결과 반환****

`select` 절에 조회 대상을 짖어하는 것을 프로젝션이라 한다.

- **프로젝션 대상이 하나**

```java
QItem item = QItem.item;
List<String> result = query.from(item).list(item.name);
```

해당 타입으로 변환한다.

- **여러 컬럼 반환과 튜플**

```java
JPAQuery query = new JPAQuery(em);
List<Tuple> result = query.from(item).list(item.name, item.price);
                                    
for(Tuple tuple: result){
    System.out.println("name = " + tuple.get(item.name));
    System.out.println("price = " + tuple.get(item.price));
  }
```

프로젝션 대상으로 여러 필드 선택 → `Tuple` 타입으로 반환(`Map`과 비슷한 내부 타입)

`tuple.get()` 메서드에 조회한 쿼리 타입을 지정하면 된다.

- **빈 생성**

쿼리 결과를 엔티티가 아닌 특정 객체로 받고 싶을 때 `빈 생성(Bean Population)` 기능을 사용한다.

```java
// DTO 에 값 채우기
public class ItemDTO{
    private String username;
    private int price;
    public ItemDTO(){}
    public ItemDTO(String username, int price){
        this.username = username;
        this.price = price;
    }
    // Getter, Setter
    ...
}
```

**프로퍼티 접근(Setter)**

```java
JPAQuery query = new JPAQuery(em);
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
  Projections.bean(ItemDTO.class, item.name.as("username"), item.price));
```

쿼리 결과와 매핑할 프로퍼티 이름이 달라 as를 통해 별칭을 준다.

**프로퍼티 직접 접근**

```java
JPAQuery query = new JPAQuery(em);
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
  Projections.fields(ItemDTO.class, item.name.as("username"), item.price));
```

필드를 `private`으로 설정해도 동작한다.

**생성자 사용**

```java
JPAQuery query = new JPAQuery(em);
QItem item = QItem.item;
List<ItemDTO> result = query.from(item).list(
    Projections.constructor(ItemDTO.class, item.name, item.price));
```

지정한 프로젝션과 파라미터 순서가 같은 생성자가 필요하다.

### 수정, 삭제 배치 쿼리

JPQL 배치 쿼리와 같이 **영속성 컨텍스를 무시**하고, **데이터베이스를 직접 쿼리**한다.

- 수정: `JPAUpdateClause`

```java
QItem item = QItem.item;
JPAUpdateClause updateClause = new JPAUpdateClause(em, item);
long count = updateClause.where(item.name.eq("만렙개발자의 JPA 책"))
    .set(item.price, item.price.add(100)); // 상품의 가격을 100원 증가
    .execute();
```

- 삭제: `JPADeleteClause`

```java
QItem item = QItem.item;
JPADeleteClause deleteClause = new JPADeleteClause(em, item);
long count = deleteClause.where(item.name.eq("만렙개발자의 JPA 책"))
    .execute();
```

### 동적 쿼리

`com.mysema.query.BooleanBuilder` 를 사용하여 특정 조건에 따른 동적 쿼리를 생성할 수 있다.

```java
// 상품 이름과 가격 유무에 따라 동적으로 쿼리 생성
SearchParam param = new SearchParam();
param.setName("백엔드개발자");
param.setPrice(10000);
QItem item = QItem.item;
BooleanBuilder builder = new BooleanBuilder();
if(StringUtils.hasText(param.getName())){
    builder.and(item.name.contains(param.getName()));
  }
if(param.getPrice() != null){
    builder.and(item.price.gt(param.getPrice()));
  }
List<Item> result = query.from(item)
    .where(builder)
    .list(item);
```

### 메서드 위임

쿼리 타입에 검색 조건을 직접 정의할 수 있다.

- **검색 조건 정의**

메서드 위임 기능을 사용하려면 정적 메서드를 만들고, `@QueryDelegate` 어노테이션에 속성으로 이 기능을 적용할 엔티티를 지정한다.

```java
public class ItemExpression {
    @QueryDelegate(Item.class)
    public static BooleanExpression isExpensive(QItem item, Integer price){ // (대상 엔티티의 쿼리타입, 필요한 파라미터...)
        return item.price.gt(price);
    }
}
```

- **쿼리 타입에 생성된 결과**

```java
public class QItem extends EntityPathBase<Item> {
    ...
    public com.mysema.query.types.expr.BooleanExpression isExpensive(Interger price) {
        return ItemExpression.isExpensive(this, price);
    }
}
```

- **메서드 위임 기능 사용**

```java
query.from(item.where(item.isExpensive(30000))).list(item);
```

`String`, `Date` 같은 자바 기본 내장 타입에도 메서드 위임 기능을 사용할 수 있다.

```java
@QQueryDelegate(String.class)
public static BooleanExpression isHelloStart(StringPath stringPath){
    return stringPath.startsWith("Hello");
}
```

정리하자면, QueryDSL을 사용해서 문자가 아닌 코드로 안전하게 코드를 작성할 수 있고, 복잡한 동적 쿼리를 쉽게 작성할 수 있다.

## 네이티브SQL

JPQL은 표준 SQL이 지원하는 대부분의 문법과 SQL 함수들을 지원하지만 특정 데이터베이스에 종속적인 기능은 지원하지 않는다.

예를 들어서,

- 특정 DB만 지원하는 함수, 문법, SQL 쿼리 힌트
- 인라인 뷰(From 절에서 사용하는 서브쿼리), UNION, INTERSECT
- 스토어드 프로시저

때로는 특정 DB에 종속적인 기능이 필요할 때가 있다. 따라서 JPA는 특정 DB에 종속적인 기능을 사용할 수 있도록 여러 방법들을 열어두었다.

- 네이티브 SQL은 **개발자가 직접 정의**하는 것이고, **영속성 컨텍스트의 기능을 그대로 사용**할 수 있다.
- 네이티브 SQL은 SQL만 직접 작성하는 것일 뿐 나머지는 JPQL을 사용할 때와 같다는 것을 명심해야 한다.

즉, JPQL은 자동 모드, 네이티브SQL은 수동 모드인 셈이다.

하지만 관리가 쉽지 않고, 자주 사용하면 특정 데이터베이스에 종속적인 쿼리가 증가해서 이식성이 떨어진다.

될 수 있으면 표준 JPQL을 사용하고 기능이 부족하면 차선책으로 JPA 구현체가 제공하는 기능을 사용하자!

### 네이티브SQL 사용법

- **엔티티 조회**

```java
// 결과 타입 정의
public Query createNativeQuery(String sqlString, Class resultClass);
// 결과 타입 정의할 수 없음
public Query createNativeQuery(String sqlString);
// 결과 매핑 사용
public Query createNativeQuery(String sqlString, String resultSetMapping);
```

실제 데이터베이스 SQL을 사용하며, 위치기반 파라미터만 지원한다. 나머지는 JPQL을 사용할 때와 같다. 조회한 엔티티도 영속성 컨텍스트에서 관리된다.

하이버네이트는 이름 기반 파라미터도 사용 가능하다.

- **값 조회**

스칼라 값만 조회하기 때문에 결과를 영속성 컨텍스트가 관리하지 않는다.

- **결과 매핑 사용**

엔티티와 스칼라 값을 함께 조회하는 것처럼 매핑이 복잡해지면 `@SqlResultSetMapping`을 정의해서 결과 매핑을 사용해야 한다.

```java
@Entity
@SqlResultSetMapping(name = "memberWithOrderCount", // 결과 매핑 정의
    entities = {@EntityResult(entityClass = Member.class)}, // ID, AGE, NAME, TEAM_ID 는 Member 엔티티와 매핑
    columns = {@ColumnResult(name = "ORDER_COUNT")} // ORDER_COUNT는 단순히 값으로 매핑
)
public class Member { ... }
```

### Named 네이티브SQL

JPQL처럼 Named 네이티브 SQL을 사용해서 정적 SQL을 작성할 수 있다.

```java
@Entity
@NamedNativeQuery(
    name = "Member.memberSQL",
    query = "SELECT ID, AGE, NAME, TEAM_ID FROM MEMBER WHERE AGE > ?",
    resultclass = Member.class
)
public Member {
    ...
}
```

위 코드를 사용하는 예제는 아래와 같다.

```java
TypedQuery<Member> nativeQuery =
    em.createNamedQuery("Member.memberSQL", Member.class)
        .setParameter(1, 20);
```

JPQL Named 쿼리와 같은 `createNamedQuery()` 메서드를 사용하므로 TypeQuery를 반환 값으로 사용할 수  있다.

- `@NamedNativeQuery` **속성**

| 속성 | 기능 |
| --- | --- |
| name | 네임드 쿼리 이름(필수) |
| query | SQL 쿼리(필수) |
| hints | 벤더 종속적인 힌트 |
| resultClass | 결과 클래스 |
| resultSetMapping | 결과 매핑 사용 |

## 객체지향 쿼리

### 벌크 연산

여러 건은 한 번에 삽입, 수정하거나 삭제하는 연산을 벌크 연산이라고 한다.

JPA 표준에서는 수정 및 삭제 벌크 연산을 `executeUpdate()` 메서드를 사용하여 구현할 수 있다.

```java
// 수정
String query = "update Product p set p.price = p.price * 1.1 " + 
    "where p.stockAmount < :stockAmount";

int resultCount = 
    em.createuery(query)
        .setParameter("stockAmount", 10)
        .executeUpdate();

// 삭제
String query = "delete from Product p where p.price < :price";

int resultCount = 
    em.createuery(query)
        .setParameter("price", 100)
        .executeUpdate();
```

참고로 하이버네이트는 삽입 연산도 `executeUpdate()` 메서드를 통해 벌크 연산이 가능하다.

### 주의점

- 벌크 연산은 영속성 컨텍스트와 2차 캐시를 무시하고 데이터베이스에 직접 쿼리한다.
- 위 특징으로 인해 아래와 같은 시나리오가 발생할 수 있다.
    - 가격이 1000원인 상품 A를 조회했다. 조회된 상품 A는 영속성 컨텍스트에서 관리된다.
    - 벌크 연산으로 모든 상품의 가격을 10% 상승시켰다. 따라서 상품 A의 각겨은 1100원이여야 한다.
    - 벌크 연산을 수행한 후에 상품 A의 가격을 출력하면 기대했던 1100원이 아니라 1000원이 출력된다.

![스크린샷 2023-10-23 오전 12.29.05.png](https://github.com/Heo-y-y/development-blog/assets/112863029/200a26c4-1fc4-4521-b1a7-a737413169fe)

![스크린샷 2023-10-23 오전 12.31.50.png](https://github.com/Heo-y-y/development-blog/assets/112863029/d994b437-e6b2-4041-b584-87bd1db73bfd)

```java
// 상품 A 조회(상품A의 가격은 1000원이다.)
Product productA = em.createQuery("select p from Product p where p.name = :name",
  Product.class)
    .setParameter("name", "productA")
    .getSingleResult();
// 출력 결과: 1000
System.out.println("productA 수정 전 = " + productA.getPrice());
// 벌크 연산 수행으로 모든 상품 가격 10% 상승
em.createQuery("update Product p set p.price=price*1.1")
    .executeUpdate();
// 출력 결과: 1000
System.out.println("productA 수정 후 = " + productA.getPrice());
```

### 벌크 연산의 문제점 해결법

- `em.refresh()` **메서드 사용**
    - 벌크 연산을 수행한 직후에 정확한 상품A 엔티티를 사용해야 한다면 `em.refresh()`를 사용하여 데이터베이스에서 상품 A를 다시 조회하면 된다.
    - `em.refresh(productA)`
- **벌크 연산 먼저 실행**
    - 가장 실용적인 해결책이다.
- **벌크 연산 수행 후 영속성 컨텍스트 초기화**
    - 영속성 컨텍스트 자체를 초기화라면 추후 데이터베이스에서 조회하여 영속성 컨텍스트에 엔티티를 저장하면 된다.

### ****영속성 컨텍스트와 JPQL****

- **쿼리 후 영속성 상태인 것과 아닌 것**
    - 조회한 엔티티만 영속성 컨텍스트가 관리한다.
    - 임베디드 타입은 조회하여 값을 변경해도 영속성 컨텍스트가 관리하지 않는다.
- JPQL로 조회한 엔티티와 영속성 컨텍스트
    - JPQL로 데이터베이스에서 조회한 엔티티가 영속성 컨텍스트에 이미 있으면 JPQL로 데이터베이스에서 조회한 결과를 버리고 대신에 영속성 컨텍스트에 있던 엔티티를 반환한다.

![스크린샷 2023-10-23 오전 12.42.39.png](https://github.com/Heo-y-y/development-blog/assets/112863029/c97141cb-9ba8-4564-b614-c21dbc650994)

1. JPQL을 사용하여 조회를 요청한다.
2. JPQL은 SQL로 변환되어 데이터베이스를 조회한다.
3. 조회한 결과와 영속성 컨텍스트를 비교한다.

![스크린샷 2023-10-23 오전 12.46.05.png](https://github.com/Heo-y-y/development-blog/assets/112863029/67433c0d-3d66-487f-9636-ac376c2f8830)

1. 식별자 값을 기준으로 member1은 이미 영속성 컨텍스트에 있기 때문에 버리고 기존에 있던 member1이 반환 대상이 된다.
2. 식별자 값을 기준으로 member2는 영속성 컨텍스트에 없기 때문에 영속성 컨텍스트에 추가된다.
3. 쿼리 결과인 member1, member2를 반환한다. 여기서 member1은 쿼리 결과가 아닌 영속성 컨텍스트에 있던 엔티티이다.
- find() vs JPQL
    - em.find() 메서드는 엔티티를 영속성 컨텍스트에서 먼저 찾고, 없으면 데이터베이스에서 찾는다.
        - 따라서 해당 엔티티가 영속성 컨텍스트에 있으면 메모리에서 바로 찾으므로 성능상 좋다.(1차 캐시)
    
    ```java
    //최초 조회, 데이터베이스에서 조회
    Member member1 = em.find(Member.class, 1L);
    //두번째 조회, 영속성 컨텍스트에 있으므로 데이터베이스를 조회하지 않음
    Member member2 = em.find(Member.class, 1L);
    // member1 == member2 는 주소 값이 같은 인스턴스
    ```
    
    - JPQL은 항상 데이터베이스에 SQL을 실행하여 결과를 조회한다.
        - 처음 JPQL 호출 시 DB에서 엔티티를 조회하고, 영속성 컨텍스트에 등록한다.
        - 두 번째 JPQL을 호출하면 DB에서 엔티티를 조회하는데 이미 영속성 컨텍스트에 조회한 같은 엔티티가 있는 경우 새로 검색한 엔티티를 버리고, 영속성 컨텍스트에 있는 기존 엔티티를 반환한다.
    
    ```java
    // 첫 번째 호출: 데이터베이스에서 조회
    Member member1 = em.createQuery("select m from Member m where m.id = :id", Member.class)
            .setParameter("id", 1L)
            .getSingResult();
    // 두 번째 호출: 데이터베이스에서 조회
      Member member2 = em.createQuery("select m from Member m where m.id = :id", Member.class)
      .setParameter("id", 1L)
      .getSingResult();
    // member1 == member2 는 주소값이 같은 인스턴스
    ```
    

### JPQL과 플러시 모드

영속성 컨텍스트의 변경 내역을 데이터베이스에 동기화한 것이다.

JPA는 플러시가 일어날 때, 영속성 컨텍스트에 등록, 수정, 삭제한 엔티티를 찾아 `INSERT`, `UPDATE`, `DELETE` SQL을 만들어 DB에 반영한다.(`em.flush()`)

플러시 모드에 따라 커밋 직전이나 쿼리 실행 기전에 자동으로 플러시 호출이 가능하다.

- `em.setFlushMode(FlushModeType.AUTO)`
    - 커밋 또는 쿼리 실행 시(실행 직전) 플러시(기본값)
- `em.setFlushMode(FlushModeType.COMMIT)`
    - 커밋 시에만 플러시
    - 성능 최적화를 위해 꼭 필요할 때만 사용하자!

JPQL은 영속성 컨텍스트에 있는 데이터를 고려하지 않고, DB에서 데이터를 조회한다.

따라서 JPQL 실행 전 영속성 컨텍스트의 내용을 DB에 반영(플러시)해야 한다.

→ **데이터 무결성**

그렇지 않으면, 영속성 컨텍스트의 엔티티 정보는 변경되었지만, DB에는 변경사항이 적용되지 않는 문제가 발생할 수 있다.

```java
// 플러시 모드 설정
em.setFlushMode(FlushModeType.COMMIT);
product.setPrice(2000);
em.flush(); //1. 직접 호출
Product product2 =
    em.createQuery("select p from Product P where p.price = 2000", Product.class)
        .setFlushMode(FlushModeType.AUTO) // 2. setFlushMode() 설정
        .getSingleResult();
```

플러시 모드를 `COMMIT`으로 설정해 놓아서 자동으로 플러시를 호출하지 않는다.

1번처럼 수동으로 플러시를 하거나, 2번처럼 해당 쿼리에만 `AUTO` 모드로 플러시 모드를 적용하면 된다.

`COMMIT` 모드를 사용하는 이유는 뭘까?

플러시가 자주 일어나는 상황에 해당 모드를 사용하면 쿼리할 때 발생하는 플러시 횟수를 줄여 성능을 최적화할 수 있다.

예를 들어서, `등록() - 쿼리() - 등록() - 쿼리() - 등록() - 쿼리()` 같이 여러 번 반복되는 경우이다.

- 만약 JPA를 거치지 않고, JDBC로 쿼리를 실행한다면 JPA는 JDBC가 실행한 쿼리를 인식할 수 없다. 따라서 `AUTO` 모드를 해도 플러시가 일어나지 않는다.
- 이 때는 JDBC로 쿼리를 실행하기 전에 `em.flush()`를 호출하여 영속성 컨텍스트의 내용을 데이터베이스에 동기화 시켜주는 것이 안전하다.

**참고 자료**

- <https://steady-coding.tistory.com/589>
- <https://www.nowwatersblog.com/jpa/ch10/10-5>
- [https://velog.io/@offsujin/JPA-10장-객체지향-쿼리-언어2](https://velog.io/@offsujin/JPA-10%EC%9E%A5-%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5-%EC%BF%BC%EB%A6%AC-%EC%96%B8%EC%96%B42)
