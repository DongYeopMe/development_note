JPA는 복잡한 검색 조건을 사용해 엔티티 객체를 조회할 수 있는 다양한 쿼리 기술을 지원한다.

JPA에서 사용할 수 있는 쿼리 기술

- JPQL
- Criteria : JPQL을 편하게 작성하도록 도와주는 API, 빌더 클래스 모음
- QureyDSL : Criteria 쿼리처럼 JPQL을 편하게 작성하도록 도와주는 빌더 클래스모음, 비표준 오픈소스 프레임워크
- 네이티브 SQL : JPA에서 JPQL대신 직접 SQL을 사용할수 있다.

Critera나 QureyDSL은 결국 JPQL을 편리하게 사용할수 있게 해주는 기술이기에 JPQL은 중요하다.

## JPQL이란?

특징은

- JPA가 제공하는 SQL을 추상화한 객체 지향 쿼리 언어
- SQL과 문법 유사하고 ANSI 표준 지원
- SQL을 추상화 해서 특정 데이터베이스 SQL에 의존하지 않는다.
- JQL은 결국 SQL로 변환한다.
- JPQL은 엔티티 객체를 대상으로 쿼리하고 SQL은 데이터베이스 테이블을 대상으로 쿼리한다.

예제

```java
@Entity(name = "Member")
public class Member{

    @Column(name = "name")
    private String username;

}

//JPQL 사용
String jpql = "select m from Member as m where m.username = 'kim'";
List<Member> resultList = em.createQuery(jpql, Member.class).getResultList();
```

이렇게 username이 kim인 엔티티를 조회한다.

em.createQuery() 메소드에 실행할 JPQL과 반환할 엔티티 클래스 타입인 Member.class를 넘겨주고 getResultList() 메소드를 실행하면 JPA는 JQL을 SQL로 변환해 데이터베이스를 조회한다.

그다음 조회한 결과로 Member 엔티티를 생성해 반환한다.

## Criteria 쿼리

- 문자가 아니라 자바코드로 JPQL을 작성할 수 있다.
- JPQL을 생성하는 빌더 클래스이며 JPA공식 기능

장점

- 컴파일 시점에 오류를 발견할 수 있다.
- IDE를 사용하면 코드 자동완성이 된다.
- 동적 쿼리를 작성하기 편하다.

단점 

- 너무 복잡해서 사용하기 불편하고 가독성이 안좋음
- 그래서 실무에서 잘 안쓰임.

```java
// Criteria 사용 준비
CriteriaBuilder cb = em.getCriteriaBuilder();
// 쿼리 생성
CriteriaQuery<Member> query = cb.createQuery(Member.class);
// from (조회를 시작할 클래스)
Root<Member> m = query.from(Member.class);
// select + from + where
CriteriaQuery<Member> cq = query.select(m).where(cb.equal(m.get("username"), "kim"));
 
List<Member> result = em.createQuery(cq).getResultList();
```

## JPQL

기본 SQL 문법과 동일합니다

select 문:: =

update 문:: =

delete 문:: =

- select m from Member as m where m.age > 20
    
    → 엔티티와 속성은 대소문자 구분.
    
- JPQL 키워드는 대소문자 구분 하지 않는다(SELECT,FROM,WHERE)
- 엔티티 이름 사용한다. 테이블명 X
- 별칭은 필수. (AS는 생략 가능)
- GROUP BY, HAVING, 그룹함수(COUNT, SUM, AVG, MAX, MIN), ORDER BY 사용 가능하다.

→

```java
select
	COUNT(m),   //회원 수
	SUM(m.age), //나이 합
	AVG(m.age), //평균 나이
	MAX(m.age), //최대 나이
	MIN(m.age)  //최소 나이
from Member m
```

### TypeQuery,Query

```java
TypedQuery<Member> query = em.createQuery("SELECT m FROM Meber m", Member.class);
Query query = em.createQuery("SELECT m.username, m.age FROM Meber m");
```

- TypeQuery : 반환 타입이 명활할때 사용
- Query : 반환 타입이 명확하지 않을때 사용.

### 결과조회 API

```java
Member member = new Member();
member.setUsername("member1");
member.setAge(15);
em.persist(member);
 
TypedQuery<Member> query1 = em.createQuery("select m from Member m", Member.class);
List<Member> result = query1.getResultList();
 
for(Member m : result) {
    System.out.println("member = " + m);
}
```

- `getResultList()` : 결과가 하나 이상일 때 사용하며 리스트를 반환한다.
    - 결과가 없으면 빈 리스트 반환

```java
// JpaMain.java
Member member = new Member();
member.setUsername("member1");
member.setAge(10);
em.persist(member);
 
TypedQuery<Member> query1 = em.createQuery("select m from Member m where m.age = 10", Member.class);
Member result = query1.getSingleResult();
 
System.out.println("member = " + result);
```

- `getSingleResult()` : 결과가 정확히 하나일 때 사용하며 단일 객체를 반환한다.
    - 결과가 없으면 `javax.persistence.NoResultException` 예외
    - 둘 이상이면 `javax.persistence.NonUniqueResultException` 예외

### 파라미터 바인딩

위치 기반 바인딩은 잘 사용하지 않는다. 왜냐하면 중간에 파라미터 추가 시 뒤에 순서가 다 밀리고 이로 인해 장애가 발생할 수 있다.

```java
/*Usercase - 1 이름 기반 */
SELECT m FROM Member m where m.username = :username
query.setParameter("username" usernameParam);

/*Usecase - 2 위치 기반*/
SELECT m FROM Member m where m.username = ?1
query.setParameter(1, usernameParam);
```

## 프로젝션(SELECT)

SELECT 절에 조회할 대상을 지정하는것을 프로젝션이라고 한다.

대상엔 엔티티,임베디드 타입,스칼라 타입이 있습니다.

DISTINCT로 중복 제거 가능하다.

### 엔티티 프로젝션

```java
// 엔티티 프로젝션은 결과가 다 영속성 컨텍스트에서 관리된다.

List<Member> result = em.createQuery("select m from Member m", Member.class).getResultList();

// 영속성 컨텍스트로 관리되기 때문에 나이값 변경 update 쿼리 실행
Member findMember = result.get(0);
findMember.setAge(20);
```

조인

```java
//묵시적 조인
List<Team> result = em.createQuery("select m.team from Member m", Team.calss)
// SQL: SELECT t.id, t.name FROM Member m inner join TEAM t on m.team_id = t.id

//명시적 조인
List<Team> result = em.createQuery("select t from Member m join m.team t", Team.calss)
//SQL: SELECT t.id, t.name FROM Member m inner join TEAM t on m.team_id = t.id
```

묵시적 조인 : JPQL에는 JOIN문법이 없지만 자연스럽게 JOIN을 해서 Team Entity를 조회해 온다.

명시적 조인 : 실행 되는 SQL은 동일하나 명시적으로 JPQL에 적어줘서 가독성이 높아지고 JOIN 쿼리가 날아 가겟구나 라고 예측 가능하다.

그래서 명시적 조인을 쓰는게 좋다.

### 임베디드 타입 프로젝션

```java
em.createQuery("select o.address from Order o", Address.calss).getRresultList();
//SQL: SELECT o.city, o.street, o.zipcode FROM ORDERS o
```

임베디드 타입은 따로 조인을 해서 가져오지는 않는다.
하지만 from절에 Order가 아닌 Address를 적으면 에러가난다. 엔티티로부터 시작되야 한다.

### 스칼라 타입 프로젝션/여러 값 조회

<aside>
💡 단 하나 값만 저장할 수 있는 데이터 타입을 스칼라 데이터 타입이라 하고,

두개 이상 값을 저장할 수 있는 데이터 타입을 컴포지트 데이터 타입이라고 한다.

</aside>

```java
//스칼라 타입
em.createQuery("select distinct m.address, m.age from Member m").getRresultList();
```

여러 값조회

여러가지가 있는데

- Query 타입으로 조회
    
    ```java
    
    List resultList = em.createQuery("select m.username, m.age from Member m").getResultList();
    // 결과를 Object[] 로 캐스팅해서 사용
    Object obj = resultList.get(0);
    Object[] objects = (Object[]) obj;
    System.out.println("query.username = " + objects[0]);
    System.out.println("query.age = " + objects[1]);
    ```
    
- Object[] 타입으로 조회
- 

```java
List<Object[]> resultList = em.createQuery("select m.username, m.age from Member m").getResultList();
 
Object[] obj = resultList.get(0);
System.out.println("object.username = " + obj[0]);
System.out.println("object.age = " + obj[1]);
```

- new 명령어로 조회
    - 단순값을 DTO로 바로 조회
    - 패키지명을 포함한 전체 클래스명 입력
    - 순서와 타입이 일치하는 생성자가 필요

그전에 이런 DTO를 만들어야한다.

```java
@Getter
@Setter
public class MemberDTO {
    
    private String username;
    private int age;
 
    public MemberDTO(){}
    public MemberDTO(String username, int age) {
        this.username = username;
        this.age = age;
    }
 
    
}
```

```java
// 패키지명을 포함한 전체 클래스명을 입력해야 하고, 순서가 일치하는 생성자가 있어야 한다.
List<MemberDTO> resultList = em.createQuery("select new jpatest.MemberDTO(m.username, m.age) from Member m", MemberDTO.class).getResultList();
 
MemberDTO memberDTO = resultList.get(0);
System.out.println("new.username = " + memberDTO.getUsername());
System.out.println("new.age = " + memberDTO.getAge());
```

이렇게 무조건 패키지명을 포함한 클래스 명을 입력하고, 생성자 매개변수 순서가 일치해야한다.

### 페이징 API

- JPA는 페이징을 2개의 API로 추상화
    - `setFirstResult(int startPosition)` : 조회 시작 위치 (0부터 시작)
    - `setMaxResults(int maxResult)` : 조회할 데이터 수

→ 기존엔  DB 뭘 써냐에 따라 쿼리를 하나하나 구현해야햇는데 이젠 이 두개의 함수로 다 사용할수 있다.

```java
List<Member> result = em.createQuery("select m from Member m order by m.age desc", Member.class)
                        .setFirstResult(0)
                        .setMaxResults(10)
                        .getResultList();
 
System.out.println("result.size : " + result.size());
// toString() 이용 출력
for(Member m : result) {
    System.out.println("member : " + m);
}
```

### **GROUP BY, HAVING, ORDER BY**

```java
// GROUP BY
"select ~~ from Member m left join m.team tgroup by t.name"

//HAVING
"select ~~ from Member m left join m.team t group by t.name having avg(m.age) >= 10"
//ORDER BY
"select m from Member m order by m.age DESC, m.username ASC"
```

Group By : 통계 데이터를 구할때 특정 그룹끼리 묶어준다.

→ 예제는 모든 회원을 팀 이름 기준으로 그룹별로 묶어서 데이터 구함.

HAVING : Group By랑 같이 사용하는데 그룹화한 통계 데이터를 필터링한다.

→ 예제는 10살 이상으로 필터링함

ORDER By : 결과를 정렬할 때 사용

→ 나이 기준으로 내림차순으로 정렬하고 같으면 이름을 기준으로 오름차순으로 정렬

- ACS : 오른차순(기본값)
- DESC : 내림차순

## 조인

```java
//내부 조인 Member와 Team을 조인하는 SQL
SELECT m FROM Member m [INNER] JOIN m.team t

//외부 조인
SELECT m FROM Member m LEFT [OUTER] JOIN m.team t

//join On 절
select m, t from Member m left join m.team t on t.name = 'A'
//팀이름이 A인 팀만 조인

//연관관계 없는 엔티티 외부 조인
select m, t from Member m left join Team t on m.username = t.name
```

JPQL 조인의 가장 큰 특징은 연관 필드를 사용한다는것이다.

내부 조인 : INNER JPIN 을 사용하고 INNER는 생략 할수 있다.

- **JPQL 조인은 SQL 조인처럼 사용하면 문법 오류가 발생한다.**
    - ex) select m from Member m join Team t

외부 조인 : LEFT OUTER JOIN을 사용하고 OUTER는 생략 가능

- 위 예제 코드 처럼 Team을 조회하면  Team이 Null일 경우 조인된 Member 수만큼 Team 객체가 null 값으로 채워진다.

Join ON 절 : 조인 대상을 필터링 하고 조인할수 있다. 내부 조인은 where을 쓸때랑 같아서 보통 외부 조인에서 사용함.

- 연관관계 없으면 On 절을 이용해 조인할 수 있다.
- 연관 관계가 없기 때문에 join 에 연관 필드를 사용하지 않고 엔티티 이름(join Team t)을 사용한다.

## 폐치 조인

SQL에 있는 조인 종류가 아니라 JPQL에서 성능 최적화를 위해 제공하는 기능이다.

연관된 엔티티나 컬렉션은 한번에 같이 조회하는 기능으로 join fetch로 사용할 수 있다.

### 일반 조인과의 차이

- 일반 조인 실행 시 연관된 엔티티를 함께 조회하지 않음
- 페치 조인을 사용할 때만 연관된 엔티티도 함께 조회(즉시 로딩)
- 페치 조인은 객체 그래프를 SQL 한번에 조회하는 개념

### 엔티티 페치 조인

@ManyToOne

예제) 멤버를 가져올때 소속된 팀도 같이 가져오고 싶다.

```java

select m from Member m join fetch m.team
//sql
SELECT M.*, T.*
FROM MEMBER M
INNER JOIN TEAM T
ON M.TEAM_ID = T.ID

//코드로 나타내면

String query = "select m from Member m join fetch m.team";
List<Member> members = em.createQuery(query, Member.class)
        .getResultList();
```

위와 같이 JPQL로 join fetch를 사용하고 select로 m만 가져와도

sql은 멤버와 팀을 가져오는 sql문이 된다.

만약에 JPQL로 “select m from Member m”으로 쿼리 보내고 결과 값으로 받은 객체들에서 member.getTeam으로 팀을 가져오게 되면 쿼리가 N+1이 되기 때문에 비효율적이다. 그래서 join fetch를 써야한다.

페치 조인으로 함께 조회했기때문에 지연 로딩이 아니다.

### 컬렉션 페치 조인

@OneToMany

일대다 관계

상황: 팀 가져올때 소속된 팀에 소속된 멤버들을 가져온다.

```java
select t
from Team t join fetch t.members
where t.name = '팀A'
//sql
SELECT T.*, M.*
FROM TEAM T
INNER JOIN MEMBER M ON T.ID = M.TEAM_ID
WHERE T.NAME = '팀A'

//코드로 나타내면
String query = "select m from Member m join fetch m.team";
List<Member> members = em.createQuery(query, Member.class)
        .getResultList();
```

페치 조인으로 함께 조회했기때문에 지연 로딩이 아니다.

이렇게 쿼리를 날리면 팀 A가 두번 나온다고 합니다.

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/e25d30d8-3a8c-44e1-ab67-6563ea11997f)

그이유는

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/96412a67-cd23-4951-90ec-f91489424abe)

이렇게 팀 A의 팀원이 2명이기때문에 조회하면 그수만큼 중복되어 나오게됩니다.

그래서 이때 필요한게 DISTINCT입니다.

### **DISTINCT**

원래 SQL에서도 중복된 결과를 제거하는 방법으로 쓰인다.

JPQL에서 2가지 기능 제공하는데

1. SQL에 DISTINCT를 추가
2. 애플리케이션에서 엔티티 중복 제거

```java
select distinct t
from Team t
join fetch t.members
where t.name = '팀A'
```

이렇게 distinct를 넣어주면 중복값을 제거해준다.

그런데 sql에서의 DISTINCT은 데이터가 완전 일치해야 중복제거 해준다. 그런데 위 그림같이 같은 소속된거라면 제거 해주지 않는다.

그래서 JPA에서는 이런 작업을 해준다.

- DISTINCT가 추가로 애플리케이션에서 중복 제거시도
- 같은 식별자를 가진 **Team 엔티티 제거**

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/44f00a05-b2e6-4fd1-97a2-6ee22b704f9f)

조회한 결과는 이렇게 나온다.

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/1fe6aa51-4743-4e13-a9ab-994a697798ea)

결과적으로 다대일은 중복값이 생기지 않지만 일대다는 중복값이 있는경우 데이터 양이 이상해진다. 그래서 distinct를 해줘야한다.

### 특징과 한계

- 페치 조인 대상엔 별칭을 줄수 없다.
    - 하이버네이트는 가능 하지만 가급적 사용 X
- 둘 이상의 컬렉션은 페치 조인 할 수 없다.
- 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없다.
    - 일대일, 다대일 같은 단일 값 연관 필드들은 페치 조인 해도 페이징가능
    - 하이버네이트는 경고 로그 남기고 메모리에서 페이징(매우 위험하다고함)
        
        → 페이징은 원래 데이터베이스에서 해야하기때문인데 
        
        → 근데 왜? : 모든 DB 데이터를 읽어서 메모리에서 페이징을 시도하는데 최악의 경우 장애 발생한다.
        

그래도 페이징 하고싶다 하면

```java
@BatchSize(size=10)
@OneTomany
~~~

//이렇게 하면 쿼리가 N+1만큼 나가지 않는다.
```

- 연관된 엔티티들을 SQL 한번으로 조회 →성능 최적화
- 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선함
    - @OneToMany(fetch = FetchType.LAZY) : 글로벌 로딩 전략
- 실무에서 글로벌 로딩 전략은 모두 지연 로딩
- 최적화가 필요한 곳은 페치 조인 적용

정리하면 

- 모든걸 페치 조인으로 해결 못함
- 객체 그래프를 유지할 때 사용하면 효과적
- 여러 테이블을 조인해서 엔티티가 가진 모양이 아닌 전혀 다른 결과를 내야 하면, 페치 조인보다는 일반 조인을 사용
- 필요한 데이터들만 조회해서 DTO로 반환하는 것이 효과적

## 서브 쿼리

예제 코드

```java
//나이 평균보다 많은 회원
select m from Member m
where m.age > (select avg(m2.age) from Member m2);
//한번이라도 주문한 고객
select m from Member m
where (select count(o) from Order o where m=o.member)>0
```

### 서브 쿼리 지원 함수

- [NOT] EXISTS(subquery): 서브쿼리에 결과가 존재하면 참
    - {ALL | ANY | SOME} (subquery)
    - 이중에 ALL : 모두 만족하면 참
    - ANY,SOME : 같은 의미이며 조건 하나라도 만족하면 참
- [NOT] IN(subquery): 서브쿼리의 결과 중 하나라도 같은 것이 있으면 참

예제

```java
//팀 A 소속 회원
select m from Member m
where exists (select t from m.team t where t.name = '팀A')

//전체 상품 각각의 재고보다 주문량이 많은 주문들
select o from Order o
where o.orderAmount > ALL(select p.stockAmount from Product p)

//어떤 팀이든 팀에 소속된 회원
select m from Member m
where m.team = ANY(select t from Team t)
```

### 서브 쿼리 한계

- JPA는 WHERE,HAVING절에서만 서브 쿼리 가능
- SELECT 절도 가능(하이버네이트에서)
- FROM 절의 서브 쿼리는 현재 JPQL에서 불가능
    - 조인 으로 풀수 있으면 풀어서 해결한다.

## JPQL타입 표현과 기타

- 문자: 'HELLO', 'SHE"S'
    - 작은 따옴표 사이에 표현할려면 연속사용 ex) ‘She’’s’
- 숫자: 10L(Long), 10D(Double), 10F(Float)
- Boolean: TRUE, FALSE
- ENUM: jpabook.MemberType.Admin(패키지명 포함)
- 엔티티 타입: TYPE(m) = Member(상속 관계에서 사용함.)

### 기타

SQL과 문법이 같다

- EXISTS, IN
- AND, OR, NOT
- =, >, ≥, <, ≤, <>
- BETWEEN, LIKE, IS NULL

**LIKE**

문자 표현식과 패턴 값을 비교한다.

% : 아무 값들이 입력되도 된다.(없어도됨)

_ : 한글자는 아무 값이 입력되어도 되지만 값이 있어야한다.

예제

```java
//중간에 원이라는 단어가 들어간 회원
select m from Member m
where m.username like '%원%'

//처음에 회원이라는 단어가 포함(회원, 회원1, 회원ABC)
where m.username like '회원%'

//회원A, 회원1
where m.username like '회원_'

//회원 %
where m.username like '회원\%' ESCAPE '\'
```

**IS NULL**

Null 값인지 비교

예제

```java
where m.username is null
```

## 조건식(CASE)

특정 조건에 따라 분기할 때 CASE 식을 사용한다.

```java
//기본 case 식
select
	case when m.age <= 10 then '학생요금'
	     when m.age >= 60 then '경로요금'
	     else '일반요금'
	end
from Member m

//심플 CASE 식
select
	case t.name
		when '팀A' then '인센티브110%'
		when '팀B' then '인센티브120%'
	     else '인센티브105%'
	end
from Team t

//COALESCE
select coalesce(m.username, '이름 없는 회원') from Member m;

//NULLIF
select NULLIF(m.username, '관리자') from Member ;
```

기본 CASE 식은 when에 조건식이 들어간다.

심플 CASE 식은 조건식을 사용할수 없고 대신 조건 대상(t.name)을 지정해줘야 한다.

COALESCE : 하나씩 조회해 null이 아니면 반환

NULLIF : 두 값이 같으면 null 반환, 다르면 첫번째 값 반환

## 벌크 연산

엔티티를 수정하려면 영속성 컨텍스트의 변경 감지 기능이나 병합을 사용하고,

삭제 할려면 `EntityManager.remove()` 메소드를 사용한다.

이렇게 하면 엔티티가 몇백개만 되도 하나씩 처리하면 오래 걸린다.

이럴때 여러 건을 한번에 수정하거나 삭제하는 벌크 연산을 사용하면된다.

예제

```java
//재고가 10개 미만인 모든 상품의 가격을 10% 상승시킨다.
String qlString = 
	"update Product p " +
	"set p.price = p.price * 1.1 " +
	"where p.stockAmount < :stockAmount";
    
int resultCount = em.createQuery(qlString)
				.setParameter("stockAmount", 10)
				.excuteUpdate();
```

이렇게 벌크 연산은 excuteUpdate()메소드를 사용한다.

이 메소드는 벌크 연산으로 영향을 받은 엔티티 건수를 반환한다

삭제도 이 메소드를 사용해 처리할 수 있다.

### 주의해야 할 점

벌크 연산은 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리를 날린다.

따라서 벌크 연산 이후에 1차 캐시에 값이 있는 데이터를 조회 할경우 데이터베이스와 값이 다를 수도 있다.

그래서 해결 방법으로는 **벌크 연산을 가장 먼저 실행**하거나 **벌크 연산 수행 후 영속성 컨텍스트를 초기화하는** 방법이 있다.

## JPQL 함수

- CONCAT : 문자열 연결
- SUBSTRING : 문자열에서 일부를 추출
- TRIM :문자열의 앞뒤에 있는 공백을 제거
- LOWER, UPPER : 문자열 대소문자 변환
- LENGTH : 문자열 길이
- LOCATE : 해당 문자 위치
- ABS, SQRT, MOD : ABS는 절댓값 SQRT는 제곱근 MOD는 나눗셈 나머지
- SIZE, INDEX(JPA 용도) : SIZE는 컬렉션 크기 INDEX는 컬렉션에서 특정 인덱스 위치 요소 가져올때 쓴다.

```java
// CONCAT
select concat('a', 'b') // ab

// SUBSTRING: firstParam의 값을 secondParam 위치부터 thirdParam 갯수만큼 잘라서 반환
select substring('abcd', 2, 3) // bc

// TRIM
select trim(' sergio ramos ') // sergio ramos

// LOWER, UPPER
select LOWER()
select UPPER()

// LENGTH
select LENGTH('sergioramos') // 11

// LOCATE
select LOCATE('r', 'ramos') // 1

// ABS, SQRT, MOD
select ABS(-30) // 30
select SQRT(4) // 2
select MOD(4, 2) // 0

// SIZE, INDEX(JPA 용도)
select SIZE(t.members) from Team t // 0
```

## 사용자 정의 함수 호출

하이버네이트는 사용전에 방언에 추가해야 한다.

- 사용하는 DB방언을 상속받고, 사용자 정의 함수를 등록한다.
실제 소스코드내부에 정의되있는 함수들을 참고해서 작성해주면 된다.

```java
//group_concat이라는 함수를 만들어서 등록한다고 가정한다.
public class MyPostgresDialect extends PostgreSQL94Dialect {
    public MyPostgresDialect() {
        registerFunction("group_concat", new StandardSQLFunction("group_concat", StandardBasicTypes.STRING));
    }
...
...
...
...

//설정 파일 등록
<property name="hibernate.dialect" value="jpql.MyPostgresDialect"/>

//하이버네이트 구현체 사용
select function('group_concat', i.name) from Item i
```

## 엔티티 활용

### 경로 표현식

.(점)을 찍어 객체 그래프를 탐색하는것을 경로 표션식이라고 부른다.

종류는

- 상태 필드 : 단순히 값을 저장하는 필드
- 연관 필드 : 연관관계를 위한 필드 임베디드 타입 포함
    - 단일 값 : ex)m.team
    - 컬렉션 값 : ex)m.orders

특징으로는

- 상태 필드 경로 : 경로 탐색의 끝
- 단일 값 연관경로 : 묵시적으로 내부 조인이 일어나고, 계속 탐색 할 수 있다.
- 컬렉션 값 연관경로 : 묵시적으로 내부 조인,
    - 더는 탐색할수 없으나 FROM 절에서 조인을 통해 별칭을 얻으면 별칭으로 탐색 할 수 있다.

예시 코드

```java
//단일 값 연관 필드 예시

//실행 JPQL
select o.member from Order o

//실제 실행되는 JPQL
select m.* from Orders o inner join Member m on o.member_id=m.id
------------------------------------------------------------------------
//컬렉션 값 연관 필드 예시
select t.members from Team //탐색 가능

select t.members.username from Team t //탐색 실패

select m.username from Team t join t.members m //탐색 가능
```

예시로 보면

단일 값은 o.member를 통해 Order 에서 Member로 단일 값 연관 필드 경로 탐색을 했다.

컬렉션 값은 컬렉션에서 경로 탐색을 시작하는 것을 허락하지 않는다.

만약 컬렉션에서 경로 탐색 하고 싶으면 조인을 사용해 새로운 별칭을 얻어야한다.

### **경로 탐색을 사용한 묵시적 조인 시 주의사항**

- 묵시적 조인은 항상 내부 조인이다.
- 컬렉션은 경로 탐색의 끝이다. 컬렉션에서 경로 탐색을 할려면 명시적 조인으로 별칭을 얻어야한다.
- 경로 탐색은 주로 SELECT, WHERE 절에서 사용하지만 묵시적 조인으로 인해 SQL의 FROM 절에 영향을 준다.

### 다형성 쿼리

예제 그림

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/ca4ad55c-9809-4a42-87d0-42da613a0c21)

이렇게 상속관계를 맺는다고 햇을때

JPQL로 Item 엔티티를 조회하면 자식 엔티티들도 함께 조회한다.

단일 테이블 전략, 조인 전략은 상관없다고 하네요

1. TYPE : 조회 대상을 특정 자식 타입으로 한정할 때 주로 사용한다.

```java
//JPQL
select i from Item i where type(i) in (Book, Movie)

//SQL
SELECT i FROM ITEM i WHERE i.DTYPE in ('B', 'M')
```

1. TREAT : 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰때 사용한다.

```java
//JPQL
select i from Item i where treat (i as Book).author = 'kim'

//SQL
select i.* from Item i
where
	i.DTYPE='B'
	and i.author='kim'
```

이렇게 부모 타입인 Item을 자식 타입인 Book으로 다룹니다. Book의 author 필드에 접근할수 있는것이죠.

JPA 표준은 FROM, WHERE 절에서 사용할 수 있지만, 하이버네이트는 SELECT 절에서도 TREAT를 사용할 수 있다.

## 참고

[https://velog.io/@sa1341/객체지향-쿼리](https://velog.io/@sa1341/%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5-%EC%BF%BC%EB%A6%AC)

https://dev-jsk.tistory.com/150

https://9hyuk9.tistory.com/77

https://catsbi.oopy.io/0361257e-64ad-41b7-b5a6-1dbaf569a77e

[https://velog.io/@songs4805/JPA-객체지향-쿼리-언어1-기본-문법](https://velog.io/@songs4805/JPA-%EA%B0%9D%EC%B2%B4%EC%A7%80%ED%96%A5-%EC%BF%BC%EB%A6%AC-%EC%96%B8%EC%96%B41-%EA%B8%B0%EB%B3%B8-%EB%AC%B8%EB%B2%95)

https://hipopatamus.tistory.com/63
