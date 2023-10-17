# Page 4

먼저 JPA에는 데이터 타입이 크게 2가지입니다.

* 엔티티 타입
* 값 타입

#### 엔티티 타입

@Entity를 붙여서 이 어노테이션을 붙여서 정의하는 객체, 관리하는 클래스들입니다.

pk값으로 관리가 되기 때문에 데이터가 변해도 식별자로 지속적으로 추적이 가능하고 관리도 편리하다.

다른 특징들은

* 생성, 소멸, 영속 등등 생명 주기가 존재한다.
* 다른 객체에서 참조가 가능하다.

#### 값 타입

* int, Integer, String 등 단순히 값으로 사용하는 자바 기본 타입이나 객체
* 식별자가 없고 값만 존재하기 때문에 추적이 변경시 불가능
* 생명주기를 엔티티에 의존, 의존하는 엔티티가 제거 되면 함께 없어짐
* 공유 하지 않는것이 안전하다.

### 값 타입 분류

#### 기본값 타입

int, double 같은 자바 기본 타입, wrapper클래스(Integer,Long),String 을 예로 들수 있다.

```java
@Entity
public class Member{
   
    private String name; // 기본 값 타입
    
    private int age; // 기본 값 타입
    
}
```

*   생명 주기를 엔티티에 의존한다.

    → Member에 있는 age,name같은 데이터는 Member1 객체가 삭제되면 삭제된다.
* 값 타입의 속성은 식별자 값이 없으며, 공유를 막아야 한다.
  * 만약 공유를 허용하면 회원의 이름을 변경할 때 다른 회원의 이름까지 변경될 위험이 존재하기 때문이다.
* 기본 타입은 항상 값을 복사하고 Integer 같은 래퍼클래스나 String같은건 공유 가능하지만, 변경은 안된다.

#### 임베디드 타입(**복합 값 타입**)

JPA를 임베디드 타입이라고 합니다.

주로 기본 값 타입을 모아서 만들기 때문에 복합 값 타입이라고 합니다.

기본 값 타입들인 int string 같은것들을 null이면 매핑한 칼럼 값도 null 이다.

회원 정보에서 비슷한 정보끼리 묶어 관리하고 싶을때 이렇게 묶어서 임베디드 타팁으로 만들고 사용하면 된다.

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

**위 그림은 Member 엔티티에서 사용되는 값들에서 공통적으로 묶을수 있는 변수들을 class로 하나로 묶은것입니다.**

날짜 : startDate,endDate

주소 : City,street,zipcode

최종적으로 이렇게 테이블 매핑이 된다.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

위 그림을 이렇게 코드로 나타내면

```java
@Embeddable
@NoArgsConstructor
public class Address {

    private String city;
    private String street;
    private String zipcode;

}
@Embeddable
@NoArgsConstructor//기본 생성자 필수로 써야한다.
public class Period {

    private LocalDateTime startDate;
    private LocalDateTime endDate;
		
		public boolean isWork(Date date){
				//이렇게 별도의 메소드로도 정의를 해도된다.
	}
}
@Entity
public class Member {

	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;

    	@Column(name = "USERNAME")
    	private String username;
        
        //period
    	@Embedded
    	private Period period;

    	//address
    	@Embedded
    	private Address homeAddress;
        
 }
```

이렇게 Address 라는 공통된 요소를 빼놓으면서 더 객체지향적인 접근이 가능하다.

여기서 쓰인 에너테이션은

@Embeddable : 값 타입을 정의하는곳

@Embedded : 값 타입을 사용하는곳

#### 매핑 재정의

만약 한 엔티티에 같은 임베디드 타입을 추가한다면 매핑하는 칼럼명이 중복하는데, 이때 @**AttributeOverride를 사용하면 임베디드 타입에 정의한 매핑 정보를 재정의된다.**

```java
@Entity 
public class Member {
    
    @Embedded
    private Address homeAddress;
    
    @Embedded
    @AttributeOverrides({ // 새로운 컬럼에 저장 (컬럼명 속성 재정의)
        @AttributeOverride(name = "city", column = @Column(name = "COMPANY_CITY")),
        @AttributeOverride(name = "street", column = @Column(name = "COMPANY_STREET")),
        @AttributeOverride(name = "zipcode", column = @Column(name = "COMPANY_ZIPCODE"))
    })
    private Address companyAddress;
  
}
//
```

이렇게 하면 homeAddress는 기존 껄로 저장되고 companyAddress는 새로 정의된 칼럼명으로 저장된다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2b14d198-fbc6-4926-8ea2-01a2fe60ea61/02ec1919-52d9-43cb-bea8-512822e22bb3/Untitled.png)

#### null값

만약 임베디드 타입을 null로 매핑하면 해당 매핑 칼럼값은 전부 null로 된다.

ex) 회원 주소에 null값으로 하면 CITY, STREET, ZIPCODE 전부 null이 된다.

```java
member.setAddress(null);
em.persist(member);
```

#### 장점

* 재사용성 향상 : 클래스를 따로 나눠서 다양한 곳에서 사용 가능하다.
* 높은 응집도
* Period.isWork()같이 해당 값 타입만 사용하는 메소드를 만들어 쓸수도 있다.
*   임베디드 타입을 포함한 모든 값 타입은 값타입을 소유한 엔티티에 생명 주기를 의존한다.

    → 단순한 엔티티 값 모임이기 때문.

#### 테이블 매핑

임베디드 타입은 엔티티의 값이기 때문에 임베디드 타입을 엔티티에 적용해도 데이터베이스 테이블 형태는 사용하기 전후가 동일하다.

그래서 객체와 테이블을 세밀하게 매핑하는게 가능하며,

잘 설계한 ORM 애플리케이션은 매핑한 테이블 수보다 클래스 수가 더 많다.

#### 연관관계

임베디드 타입도 값 타입을 포함하거나 엔티티 참조가 가능하다.

```java
@Entity 
public class Member {
    
    @Embedded
    private Address address;
    
    @Embedded
    private PhoneNumber phoneNumber;
    
}

@Embeddable
public class Address {
    private String city;
    private String street;
    
    @Embedded
    private Zipcode zipcode; // 임베디드 타입 포함
}

@Embeddable
public class PhoneNumber {
    private String number;
    
    @ManyToOne
    private PhoneServiceProvider provider; // 엔티티 참조
}
```

### 값 타입 공유 및 비교

값 타입은 객체를 단순 화 하려고 만든 것이기 때문에 값 타입은 단순하고 안전하게 다룰 수 있어야 한다.

#### 공유 참조

임베디드 타입 같은 값 타입을 여러 엔티티에서 공유하면 위험하다.

→ 다양한 부작용이 발생할수도있고, 만약 서로 다른 회원이 같은 객체를 참조할 경우 변경했을때 다른 회원 정보가 변경되기도하고 이런 버그는 찾아내기 어렵다.

예제 코드

```java
Address address = new Address("Old City", "Street", "12345");

member1.setAddress(address);
em.persist(member1);

member2.setAddress(address);
em.persist(member2);

member1.getAddress().setCity("New City"); // Member1의 주소를 변경

tx.commit(); // UPDATE 쿼리가 2번 발생해 member1과 2의 주소가 모두 변경
```

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2b14d198-fbc6-4926-8ea2-01a2fe60ea61/dd119af1-e518-4dce-ab62-ca49f50e02f9/Untitled.png)

#### 복사

만약에 같은 값 타입을 사용하고 싶다면 값을 복사해서 사용해야한다.

```java
Address address = new Address("Old City", "Street", "12345");

member1.setAddress(address);
em.persist(member1);
//이런 식으로 객체를 복사
Address copyedAddress = new Address(address.getCity(), address.getStree(), address.getZipcode());

member2.setAddress(copyedAddress); // 복사된 주소로 저장한다.
em.persist(member2);

member1.getAddress().setCity("New City"); // member1의 주소만 변경된다.
```

자바는 객체에 값을 대입하면 참조값을 전달하기 때문에 같은 인스턴스를 조회하게 된다.

값을 복사해 사용하면 버그를 피할수 있다.

그런데 임베디드 타입처럼 직접 정의한 값 타입은 자바의 기본 타입이 아니라 객체 타입이다.

객체 타입은 참조 값을 직접 대입하는 것을 막을 방법이 없다.(공유 참조)

메모리 주소를 복사해서 대입 되기 때문에 동일한 인스턴스가 생성되버리는것이다.

#### 불변 객체

객체 타입을 수정할 수 없는 불변 객체로 설계하면 된다. 그럼 부작용이 해결이 된다.(불변 객체 : 생성 이후로 값 변경 할수 없는 객체)

래퍼 클래스(Integer)와 String은 자바가 제공하는 대표적인 불변 객체이다.

생성방법

* 생성자로만 값을 설정하고 setter를 만들지 않기
* setter를 private으로 설정하기

#### 값 타입 비교

자바의 객체 비교는 2가지가 있습니다.

* 동일성 : 인스턴스의 참조 값을 비교 → `==`
* 동등성 : 인스턴스의 값을 비교 → `equals()`

예제

```java
Address a = new Address("City", "Street", "12345");
Address b = new Address("City", "Street", "12345");

@Override
public int hashCode() {
    return Objects.hash(city, street, zipcode);
}
```

a==b 으로 동일성 비교하면 참조값이 다르기 때문에 false,

값을 비교하려면 equals()를 사용해 동등성 비교 해야한다.

값 타입은 자바 기본타입 제외하고 equals()를 써야한다.

### 컬렉션 값 타입

* 값 타입 여러 개 저장 할때 사용한다.
* @ElementCollection,@CollectionTable 사용한다.
*
* 데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없고 값만 보관할수 있다.
  * 컬렉션을 저장하기 위해선 별도의 테이블이 필요하다.
  * 그래서 @CollectionTable을 사용해 추가한 테이블을 매핑하고, 생략하면 기본 값을 사용해 매핑하게 된다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2b14d198-fbc6-4926-8ea2-01a2fe60ea61/c0fef0dd-0b86-4118-b287-4f6dd96cc311/Untitled.png)

이 그림 처럼 Member 엔티티에 Set,List 같은 컬렉션을 사용하면 Member 엔티티는 Member, Favorite\_food, Address 3가지 테이블을 갖는다.

→ 컬렉션 타입마다 테이블이 하나씩 생기는것이다.

#### 매핑

코드로 나타내면

```java
@Entity
public class Member {
    
    @ElementCollection
    @CollectionTable(
        name = "ADDRESS",
        joinColumns = @JoinColumn(name = "MEMBER_ID"))
    private List<Address> addressHistory = new ArrayList<>();
    
    @ElementCollection
    @CollectionTable(
        name = "FAVORITE_FOOD",
        joinColumns = @JoinColumn(name = "MEMBER_ID"))
    @Column(name = "FOOD_NAME") // 컬럼명 지정
    private Set<String> favoriteFoods = new HashSet<>();
    
}
```

@ElementCollection : 엔티티 해당 값 타입이 컬렉션 타입으로 의미하는 어노테이션

@CollectionTable : Collection 타입과 매핑되는 테이블을 적어두는 어노테이션

여기서 joinColumns : 어떤 칼럼들과 매핑할건지 작성.

@JoinColumn 어노테이션은 name 에 적힌 테이블에서 어떤 칼럼과 조인할건디 작성.(FK에 해당되는 칼럼)

Set\<String>을 보면 String 타입으로 고정되어 있기 때문에 @Column을 쓸수 있다.

#### 저장

```java
Member member = new Member();
Address address = new Address("city", "street", "12345");

member.setHomeAddress(address); // 임베디드 값이기 때문에 회원을 저장할 때 함께 저장한다.

member.getFavoriteFoods().add("피자");
member.getFavoriteFoods().add("치킨");
member.getFavoriteFoods().add("콜라");

member.getAddressHistory().add(new Address("old city1", "street1", "10001"));
member.getAddressHistory().add(new Address("old city2", "street2", "10002"));

em.persist(member);

tx.commit();
```

여기서 em.persist(member)를 호출하면 6개의 INSERT 쿼리가 실행되는데,

값 타입들은 Member 엔티티에 소속되는 값들이기 때문에 생명주기가 Member에 의존한다. 그래서 영속성전이와 고아 객체 제거(자세한건 밑에)를 설정 한것과 동일하다.

#### 조회

컬렉션 값 타입도 Fetch 전략을 사용할수 있는데, 기본값은 LAZY 전략을 사용한다.

그래서 Member를 조회하면 임베디드 값인 Address도 같이 조회되는데, 컬렉션들은 아직 조회 안된 상태고 사용할때 SELECT 쿼리로 가져온다

```java
Member member = em.find(Member.class, id); // SELETE 1

Address address = member.getAddress(); // 이미 값을 가지고 있어 새로운 쿼리 발생하지 않는다.

// SELECT 2
List<Address> addressHistory = findMember.getAddressHistory();
for (Address address : addressHistory) {
}

// SELECT 3
Set<String> favoriteFoods = findMember.getFavoriteFoods();
for (String favoriteFood : favoriteFoods) {
}
```

#### 수정

```java
// 1.임베디드 값
member.setAddress(new Address("New City", "New Street", "54321"));

// 2. Set 컬렉션 값 타입
Set<String> favoriteFoods = member.getFavoriteFoods();
favoriteFoods.remove("치킨");
favoriteFoods.add("뿌링클");

// 3. List 컬렉션 값 타입
List<Address> addressHistory = member.getAddressHistory();
addressHistory.remove(new Address("old city1", "street1", "10001"));
addressHistory.add(new Address("city", "street", "12345"));
```

1. 임베디드 값 타입 수정
   * address는 MEMBER 테이블과 매핑했기 때문에 MEMBER 테이블만 UPDATE
2. 기본 값 타입 수정
   * String은 불변 객체이기 때문에 삭제하고 새로운 객체를 넣어야 한다.
   * 컬렉션의 값만 변경해도 JPA가 변경사항을 알고 UPDATE를 실행
3. 컬렉션 값 타입 수정
   * 불변 객체이기 때문에 기존에 있는 값을 찾아서 삭제후 새로운 값을 넣는 방식이다.
   * \*\*`equals()`\*\*와 \*\*`hashCode()`\*\*가 반드시 오버라이딩 해야한다.

#### 제약 사항

* 값 타입은 식별자가 없어 값을 변경하면, 추적이 어렵다.

그래서 컬렉션 값 타입의 값들은 별도의 테이블에 보관된다. 때문에 보관된 값이 변경되면 데이터베이스에 있는 원본을 찾기 어렵다.

JPA에서는 이러한 문제를 해결하기 위해 **컬렉션 값 타입에 변경 사항이 발생하면 엔티티와 연관된 모든 데이터를 삭제하고 현재 컬렉션에 있는 모든 값을 데이터베이스에 다시 저장한다.**

→ 예를들어 결제 내역과 결재 내역에 있는 상품들 테이블이 존재할때 상품 내역중 1개만 삭제해도 결제 내역에 속한 상품들 전체가 다 삭제되고 새로 변경되는 값을 합쳐서 다시 그 전체가 insert가 된다.

그래서 컬렉션 타입을 매핑하는 테이블은 모든 칼럼을 묶어서 기본키를 구성하는게 좋다. → nullX, 중복 저장X

#### 대안

* 실무에서는 단순한 경우가 아니라면 상황에 따라 값 타입 컬렉션 대신 일대다 관계를 사용하는 것이 좋다고합니다.
* 일대다 관계를 위한 엔티티를 생성하고, 여기서 값타입을 사용하는 방법으로 해결할 수가 있다고 합니다.
* 영속성 전이(cascade)와 고아 객체 제거(orphan remove)를 사용하면 값 타입과 유사하게 사용이 가능하다.(거의 필수)

**orphan remove**

```java
@OneToMany(cascade = {CascadeType.ALL}, fetch = FetchType.EAGER, orphanRemoval = true)
```

이렇게 1:N 관계 테이블 설정할때 이렇게 옵션을 추가하는데

자식엔티티의 변경이 있을때

JPA에서 엔티티 수정은 insert,update,delete 순서로 실행되는데

변경된 자식을 먼저 Insert하고 기존의 자식을 NULL로 update한다.

그러고 orphan Remove 옵션을 true로 했을때 기존 NULL 처리된 자식을 Delete한다.

그래서 PK값이 NULL로 변한 자식을 고아 객체라고 부르며 연결된 점이 없는 객체입니다.

orphan Remove 옵션은 이 고아 객체를 삭제해주는 옵션입니다.

## 참고

[https://zepettoworld.tistory.com/36](https://zepettoworld.tistory.com/36)

[https://girawhale.tistory.com/132](https://girawhale.tistory.com/132)

[https://terianp.tistory.com/174](https://terianp.tistory.com/174)
