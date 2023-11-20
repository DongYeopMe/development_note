## JPA 쓰기전

JPA 쓰기전엔 관계형 데이터에비스(Oracle,Mysql)만 쓰다보니 코드 대부분을 SQL 명령어로 길게 쓰게 됩니다. 

그리고, 각 테이블 마다 기본적인 CRUD SQL를 매번 생성해야 해서 반복적인 SQL들도 늘고, 규모가 커질수록 테이블이 늘기때문에 유지보수도 힘들었습니다.

## ORM(Object-Relational Mapping)

JPA 를 하기전 ORM에 대해 알아야합니다.

ORM이란 클래스와 관계형 데이터베이스의 테이블을 매핑(연결)한다는 의미이고, 기술적으론 어플리케이션의 객체를 관계형 데이터베이스 테이블에 자동으로 영속화(Persistence) 하는 것이다.

### 장점

- SQL문이 아니라, Method로 DB를 조작할수 있고, 객체 모델을 이용해 비즈니스 로직만 집중할수 있습니다. → 내부적으로 쿼리를 생성해줌.
- 선언,할당 같은 부수적인 코드가 줄어들고, 각종 객체에 대한 코드를 변도로 작성해 코드 가독성을 높일수 있다
- DB를 바꿔도 새로 쿼리를 안짜도된다.

### 단점

- 프로젝트 규모가 커지며 복잡해지거나 설계가 잘못된 경우엔, 속도 저하 및 일관성이 무너진다.
- 복잡하고 자세하게 들어가면 SQL문을 써야한다.

## Spring Data JPA

![image](https://github.com/DongYeopMe/development_note/assets/70151275/9e69dd18-bae3-4879-84b3-37449d63498b)

여기서 관계형 데이터베이스, 객체지향 프로그래밍 언어의 패러다임을 중간에서 일치시켜주는 기술이 등장한게 ORM 프레임워크인데, 

하이버네이트(hibernate)라는 오픈소스 ORM 프레임워크가 등장하면서, 이걸 구현체로 쓰는 인터페이스인 JPA이다.

물론 다른 구현체로 쉽게 교체할수 있다.(ex) EclipseLink)

> 패러다임 일치
> 
> 
> 추상화, 상속, 다형성과 같은 특성을 가진 객체지향과 이러한 특성이 없는 데이터베이스는 서로 기능과 표현방법이 모두 다르다. 이게 객체와 관계형 데이터베이스의 패러다임 불일치 문제라고 한다.
> 

동작 과정은

![image](https://github.com/DongYeopMe/development_note/assets/70151275/c872b6dd-be66-4471-b8ca-fe8759f3656c)

JPA는 애플리케이션과 JDBC 사이에서 동작한다. JPA내부엔 JDBC API를 사용해 SQL을 호출해서 DB와 통신한다. 

개발자가 ORM 프레임워크에 저장하면 적절한 INSERT SQL을 생성해 데이터베이스에 저장하고 검색 하면 쿼리를 또 생성해 결과를 객체에 매핑하고 전달한다.

## Spring Data JPA쓰는 이유

1. Hibernate 외에 다른 구현체로 쉽게 교체 할수 있어서 Hibernate가 수명을 다하면 새로운 JPA 구현체로 교체할 수 있다.
2. 관계형 데이터베이스 외에 다른 저장소로 쉽게 교체 할수 있다.
    
    ex) 서버에 트래픽이 많아져 관계형 데이터베이스로는 도저히 감당이 안될때 만약 MongoDB로 교체가 필요하면 Spring Data JPA에서 Spring Data MongoDB로 의존성을 교체하면 된다.
    
    → Spring Data의 하위 프로젝트들의 기본적인 CRUD 인터페이스가 같기 때문.
    

# 참고

https://kbwplace.tistory.com/163

https://girawhale.tistory.com/119
