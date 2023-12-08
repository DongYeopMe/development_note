# POJO란

Plain Old Java Object의 약자. 

Java EE 등의 중량 프레임워크들을 사용하면서 해당 프레임워크에 종속된 “무거운 객체”를 만드는 거에 반대인 순수한 자바 객체를 말한다.

→ 무거운 객체 : 특정 기술에 종속되어 동작하는 객체.(각종 기술 여러개 들어간 것)

POJO 예제 코드

```java
public class Person {
    private String name;
    private int age;

    public Person() {
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

POJO는 개인 정보를 private 필드로 캡슐화하고, 해당 데이터에 접근하고 수정할 수 있는 getter 및 setter메소드를 제공하는 간단한 자바 클래스다.

POJO 아닌 코드

```java
import org.springframework.beans.factory.annotation.Autowired;

public class OrderService {
    @Autowired
    private OrderRepository orderRepository;

    public void createOrder(String item) {
        Order order = new Order(item);
        orderRepository.save(order);
    }
}
```

이 코드는 특정 프레임워크나 라이브러리에 종속된 추가적인 의존성과 동작을 가지고 있다.

→ OrderService 클래스는 `@Autowired` 의존성 주입 어노테이션을 가지고 있으며, 이는 Spring 프레임워크에 종속되어 있다.

이렇게 되면 실제 POJO와 비교하여 다른 컨텍스트에서 재사용하기가 어려울수 있다.

POJO스러운 코드로 수정하면

```java
public class OrderService {
    private OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public void createOrder(String item) {
        Order order = new Order(item);
        orderRepository.save(order);
    }
}
```

이렇게 되는데

차이점은 

OrderService 클래스는 Spring 어노테이션과 프레임워크에 종속되어 있지 않다. 대신에 생성자로 OrderRepository 인스턴스를 전달 받도록 변경했다. 

이 접근 방식으로 테스트 하기에 좋아지고, 테스트 중에 OrderRepository의 모의 구현을 제공하기가 쉬워진다.

정리하면 POJO의 정의적 특성은 간결함과 특정 프레임워크가 라이브러리에 종속되지 않는 것이다. 

POJO는 외부 기술과 강하게 결합되지 않고, 쉽게 이해고 테스트하며 여러 응용 프로그램에서 재사용될 수 있는 자체 포함 형 자바 클래스 여야 한다.

## Spring에서 POJO의 목적은 무엇일까?

Spring 프레임워크는 객체 지향적인 개발을 지향하고, POJO를 중심으로 애플리케이션을 구성하는 것에 장려한다. 

Spring은 POJO를 활용해 애플리케이션의 핵심 비즈니스 로직을 담은 클래스를 작성하고, 이 클래스들을 서로 연결하고 구성하면서 애플리케이션의 구조를 유연하고 모듈화된 형태로 구성할 수 있게 도와준다.

### Spring의 POJO 특징

- DI(의존성 주입) : Spring은 POJO에 필요한 의존성을 주입하는 방식을 통해 객체 간의 결합도를 낮추고 유연한 구조를 제공한다.
- AOP(관점 지향 프로그래밍) : Spring은 POJO에 관점 지향 프로그래밍을 적용하여 보안, 로깅, 트랜잭션 관리 등과 같은 횡단 관심사를 분리하여 관리할 수 있게 해준다.
- 라이프사이클 관리 : Spring은 POJO의 생성부터 소멸까지의 라이프사이클을 관리하며, 필요한 초기화 작업이나 정리 작업을 수행할 수 있는 확장 포인트를 제공한다.
- POJO 기반 설정 : Spring은 XML 또는 Java Config와 같은 설정 방식을 통해 POJO들을 구성하고 관리할 수 있다.
- 테스트 용이성 : POJO는 프레임워크에 종속 되지 않아 단위 테스트 및 통합 테스트를 보다 쉽게 진행 할 수 있다.

### 정리

Spring은 엔터프라이즈 서비스들을 POJO 기반으로 만든 비즈니스 오브젝트에서 사용할 수 있게 해준다.

IoC 컨테이너를 제공해서, 인스턴스들의 사이클을 관리하고, 특정 인터페이스를 구현하거나 상속할 필요 없고 라이브러리를 지원하기에 용이하며, 객체또한 가볍다.

OOP를 더 OOP답게 쓸 수 있게 해주는 AOP 기술을 적용해  POJO 개발을 더 쉽게 해준다.

# 참고

https://ko.wikipedia.org/wiki/Plain_Old_Java_Object

https://yoo11052.tistory.com/133

https://siyoon210.tistory.com/120
