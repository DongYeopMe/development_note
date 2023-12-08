# AOP란 무엇인가?

**Plain Old Java Object약자, 관점 지향 프로그래밍**

관점 지향이란 핵심 로직과 부가 기능을 분리하여 애플리케이션 전체에 걸쳐 사용되는 부가 기능을 모듈화 하여 재사용할 수 있드록 지원하는 것이다.

- 모듈화란 어떤 공통된 로직이나 기능을 하나의 단위로 묶는 것이다.

→ 즉 AOP는 흩어진 관심사(**Crosscutting Concerns**)를 모듈화 할 수 있는 프로그래밍 기법이다.

![image](https://github.com/DongYeopMe/development_note/assets/70151275/b831d81f-6c13-4428-b876-87451ac5805f)

이처럼 각각의 Service의 핵심 기능에서 바라보면 User와 Order는 공통된 요소가 없다. 

![image](https://github.com/DongYeopMe/development_note/assets/70151275/8cfb95c9-6e05-4716-9b5c-61ec113f2d23)

이 처럼 부가 기능 관점에서 보면 공통된 요소가 보인다.

그 공통된 요소는 각각의 Service의 get메소드를 호출하는 전후에 before()와 after() 라는 메소드가 공통되는 것을 확인할 수 있다.

즉, 기존에 OOP에서 바라보던 관점을 다르게 하여 부가 기능적인 측면에서 공통된 요소를 추출하는 것이다.

- OOP : 문제를 여러 개의 객체 단위로 나눠 작업하는 방식, 객체들이 서로 유기적으로 상호 작용하는 프로그래밍 이론.

→ 이걸 가로 영역의 공통된 부분을 잘라낸다고 하여 AOP를 Cross-Cutting 이라고도 부른다.

여기서 OOP와 AOP를 보자면

OOP : 비즈니스 로직의 모듈화

- 모듈화의 핵심 단위는 비즈니스 로직.

AOP : 인프라 혹은 부가기능의 모듈화

- ex) 동기화, 예외처리, 성능최적화 등
- 각각의 모듈들의 주 목적 외에 필요한 부가적인 기능들.

정리를 하자면, AOP는 공통된 기능을 재사용하는 기법. OOP에선 공통된 기능을 재사용하는 방법으로 상속 이나 위임을 사용한다. 

그런데, 전체 애플리케이션에서 여러 곳에서 사용되는 부가 기능들은 상속이나 위임으로 처리하기엔 깔끔한 모듈화가 어렵다. →대처 : AOP

## AOP 장단점

### 장점

- 애플리케이션 전체에 흩어진 공통 기능이 하나로 관리되어 유지보수에 좋다
- 핵심 로직과 부가기능의 명확한 분리로, 핵심 로직은 자신의 목적 외에 다른 상황에는 신경쓰지 않는다.

### 단점

- 하나로 괸리되다 보니 AOP에서 해주는 Logic을 정화갛게 모르면 힘들수 있다.
    
    → 극복 방법은
    
    - AOP를 사용하는 개발자가 정확하게  AOP가 해주는 Logic을 알고 사용한다.
    - AOP에서 너무 많은 ROLE(역할 또는 책임)을 가지가지 않도록한다.
- AOP가 나중에 파라미터가 변경되거나 추가 될때 다른 로직들에게 영향이 크다.
    
    → 그래서 나중에 변경되거나 추가되었을때 코드 변경 작업이 많을것이다.
    

## AOP 적용 방법

### 1. 컴파일 시점(CTW)

- .java 파일을 컴파일러를 통해 .class를 만드는 시점에 부가 기능 로직을 추가하는 방식이다.
    - 컴파일 해서 클래스 파일로 만들때, Advice코드가 추가된 조작된 바트 코드를 생성한다.
- 모든 지점에 적용이 가능하다
- 로드 타임이나 런타임때 추가적인 작업을 안해서 성능적인 부하가 없다.
- AspectJrk 제공하는 특별한 컴파일러를 사용해야 하기 때문에 특별한 컴파일러가 필요한 점과 복잡하다는 단점이 있다.

### 2. 컴파일 후 시점(PCW)

- 초기 행동이 발생한 후에 작용하는 것처럼, 컴파일된 코드를 수정하면서 원래의 소스는 변경하지 않는다.

### 3. 로드 시점(LTW)

- .class 파일을 JVM 내부의 클래스 로더에 보관하기 전에 조작해서 부가 기능 로직을 추가하는 방식이다.
    
    → 컴파일은 순수하게 작업하고 컴파일 후 **컴파일된 클래스를 로딩하는 시점에 Advice 소스를 끼워 넣은 상태에서 로딩.**
    
- 모든 지점에 적용이 가능하다.
- 특별한 옵션과 클래스 로더 조작기를 지정해야하므로 운영하기 어렵다.
- 다양한 기능을 제공하지만 로딩 시점에 부하가 발생한다는 단점이 있다.

### 4. 런타임 시점(RTW)

- 스프링이 사용하는 방식이다.
- 컴파일이 끝나고 클래스 로더에 이미 다 올라가 자바가 실행된 후 동작하는 런타임 방식이다.
- 실제 대상 코드는 그대로 유지 되고 **프록시를** 통해 부가 기능이 적용된다.
- **프록시는 메서드 오버라이딩 개념으로 동작하기 때문에 메서드에만 적용이 가능하다. → 스프링 빈에만 AOP를 적용할 수 있다.**
- 특별한 컴파일러나, 복잡한 옵션, 클래스 로더 조작기를 사용하지 않아도 스프링만 있으면 AOP를 적용할 수 있기 떄문에 스프링 AOP는 런타임 방식을 사용한다.
    
    → 별도의 컴파일 과정을 거치지 않아도 된다는 장점이 있지만 최초 런타임 시점에 비용이 발생한다
    

<aside>
❓ 스프링 AOP는 AspectJ 문법을 차용하고 프록시 방식의 AOP를 제공한다.
 스프링에서는 AspectJ가 제공하는 어노테이션이나 관련 인터페이스만 사용하고, 실제로 AspectJ가 제공하는 컴파일, 로드타임 위버등은 사용하지 않는다. 
따라서 스프링 AOP는 AspectJ를 직접 사용하는 것은 아니다.

</aside>

## AOP 용어

![image](https://github.com/DongYeopMe/development_note/assets/70151275/f5aa13e8-077e-48cc-aef7-4624af0786e8)

### Join point

- 추상적인 개념으로 advice가 적용될 수 있는 모든 위치를 말한다.
- 예를 들면  메서드 실행 시점, 생성자 호출 시점, 필드 값 접근 시점 을 말한다.
- 스프링 **AOP는 프록시 방식을 사용하므로 join point는 항상 메서드 실행 지점을 의미한다.**

### Pointcut

- Join point 중에서 advice가 적용될 위치를 선별하는 기능이다.
- 스프링 AOP는 프록시 기반이기 때문에 join point가 메소드 실행 시점 뿐이 없고, pointcut도 메소드 실행 시점만 가능하다.

### Target

- advice의 대상이 되는 객체이며, pointcut으로 결정된다.

### Advice

- 실질적으로 어떤 일을 하는것. → 실질적인 부가 기능을 담은 구현체

### Advisor

- 스프링 AOP에서만 사용되는 용어.
- advice+ pointcut 한쌍.

### Weaving

- pointcut으로 결정한 타겟의 join point에 adivce를 적용하는 것이다.

### AOP 프록시

- AOP 기능을 구현하기 위해 만든 프록시 객체이다.
- 스프링 AOP 프록시는 JDK 동적 프록시 또는 CGLIB 프록시이다.
- 스프링 **AOP의 기본값은 CGLIB 프록시이다.**
    - 처음엔 CGLIb가 권장하지 않았지만, 어느 시점 부터 개선을 해서 기본값으로 변경했다.
    - 권장하지 않은 이유는 이런 단점들 때문이었습니다.
        - net.sf.cglib.proxy.Enhancer 의존성 추가해야 개발 가능
        - default 생성자 필요.
        - 타깃의 생성자 두 번 호출된다.

## Spring AOP가 제공하는 두가지 AOP Proxy

 간단하게 설명하면 JDK 동적 프록시는 인터페이스를 구현해서 프록시 생성, CGLIB는 클래스를 상속 받아 프록시 생성한다.

인터페이스가 없다면 당연히 CGLIB로 동작하지만, 인터페이스가 있는 경우라면 CGLIB과 JDK 동적 프록시 중에서 선택할 수 있다.

![image](https://github.com/DongYeopMe/development_note/assets/70151275/6c76751e-0b91-4438-8f5d-b30012aba572)

### JDK 동적 프록시

![image](https://github.com/DongYeopMe/development_note/assets/70151275/b31bf2c6-cbcc-423d-91ac-85a1c442055d)

JDK 동적 프록시는 구체 클래스로 타입 캐스팅이 불가능하다.

MemberServiceImpl을 대상으로 프록시를 생성한다면 인터페이스인 MemberService를 기반으로 구현체(프록시)로 만들어 빈으로 등록한다.

그래서 프록시는 MemberService로는 타입 캐스팅이 가능하나, MemberServiceImpl로는 타입 캐스팅이 불가능하다.

### CGLIB 프록시

![image](https://github.com/DongYeopMe/development_note/assets/70151275/c6e88e2f-43f9-4c86-9504-8e9240fe39d1)

CGLIB는 구체 클래스로 타입 캐스팅이 가능하다.

CGLIB는 구체 클래스를 상속받아 프록시를 생성한다. 따라서 MemberServiceImpl을 프록시 대상으로 선택하면, MemberServiceImpl을 상속 받아서 프록시를 만들어 빈으로 등록 하기 때문에 프록시는 당연히 MemberServiceImpl로 타입 캐스팅이 가능하다.

그래서, JDK 동적 프록시 설정으로 돌리면 빈에 등록된 프록시는 MemberServiceImpl로 타입 캐스팅이 불가능해 의존관계 주입에 실패한다.

반면에 CGLIB를 사용하면 타입 캐스팅이 가능하기 때문에 의존관계 주입이 가능하다.

정리하면, CGLIB는 구체 클래스가 AOP의 대상이 되고, JDK 동적 프록신느 구체 클래스가 AOP의 대상이 되지 못한다.

그런데 CGLIB의 단점이 있습니다.

- 대상 클래스에 기본 생성자 필수
- 부모 생성자 2번 호출 문제
- final 키워드 클래스 메소드 사용 불가

이 단점들은 spring에서 해결해서 Spring 3.2 버전부터 CGLIB를 Spring Core 패키지에 포함시켜 더이상 의존성을 추가하지 않아도 개발할 수 있게 되었다.

4버전에선 Objensis 라이브러리의 도움을 받아 default 생성자 없이도 Proxy를 생성할 수 있게 되었고, 생성자 2번 호출 되던 상황도 개성이 되었다.

그래서 Spring에선 CGLIB가 기본값으로 Proxy를 생성한다.

## Spring AOP

- 스프링에서 제공하는 스프링 AOP는 프록시 기반의 AOP 구현체이다.
- 프록시 객체를 사용하는 것은 접근 제어 및 부가 기능을 추가하기 위해서 이다.
- 스프링 AOP는 스프링 빈에서만 적용할 수 있다.
- 모든 AOP 기능을 제공하는 것이 목적이 아니라 중복 코드, 프록시 클래스 작성의 번거로움 등 흔한 문제들을 해결하기 위한 솔루션을 제공하는 것이 목적이다.
- 스프링 AOP는 순수 자바로 구현 되었기 때문에 특별한 컴파일 과정이 필요하지 않다.

![image](https://github.com/DongYeopMe/development_note/assets/70151275/db37fff2-7497-4105-8e12-f2ce150c3928)

프록시 패턴에서는 interface가 존재하고 Client는 이 interface 타입으로 프록시 객체를 사용한다. 프록시 객체는 기존의 타겟 객체(Real Subject)를 참조한다. 프록시 객체와 기존의 타겟 객체의 타입은 같고,프록시는 원래 할 일을 가지고 있는 Real Subject를 감싸서 Client의 요청을 처리하는 것이다.

### Spring AOP 적용

Spring AOP를 사용하려면 의존성을 추가해줘야 한다.

maven

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

gradle

```java
implementation 'org.springframework.boot:spring-boot-starter-aop'
```

해당 의존성을 추가하면 자동 프록시 생성기(AnnotationAwareAspectJAutoProxyCreator)를 사용할 수 있게 된다. 이 생성기는 Advisor 기반으로 프록시를 생성하는 역할 한다. 그리고 자동 프록시 생성기는 @Aspect를 보고 Advisor로 변환해서 저장하는 작업을 한다.

![image](https://github.com/DongYeopMe/development_note/assets/70151275/b4c680c8-4bbe-495e-9c44-ef7b6a532571)

이 자동 프록시 생성기에 의해 @Asepct에서 Advisor로 변환된 Advisor는 @Aspect Advisor 빌더 내부에 저장된다.

### 동작 과정

![image](https://github.com/DongYeopMe/development_note/assets/70151275/18630218-c6bf-4ffc-baf6-69cff985acde)

1. 스프링 빈 대상이 되는 객체를 생성한다.(@Bean, 컴퍼넌트 스캔 대상)
2. 생성된 객체를 빈 저장소에 등록하기 직전에 빈 후처리기에 전달한다.
3. 모든 Advisor 빈을 조회한다.
4. @Aspect Advisor 빌더 내부에 저장된 모든 Advisor를 조회한다.
5. 3,4에서 조회한 Advisor에 포함되어 있는 Pointcut을 통해 클래스와 메소드 정보를 매칭하면서 프록시를 적용할 대상인지 아닌지 판단한다.
6. 여러 Advisor의 하나라도 포인트컷의 조건을 충족한다면 프록시를 생성하고 프록시를 빈 저장소로 반환한다.
7. 만약 프록시 생성 대상이 아니라면 들어온 반 그대로 빈 저장소로 반환한다.
8. 빈 저장소는 객체를 받아서 빈으로 등록한다.

Advisor 빈을 조회하고 이후에 @Aspect Advisor 빌더 내부에 저장된 모든 Advisor를 조회하는 로직이 추가된 것을 확인할 수 있다.

@Aspect는 Advisor를 쉽게 만들 수 있도록 도와주는 역할을 할 뿐, 컴포넌트 스캔이 되는게 아니기때문에 반드시 스프링 빈으로 등록해줘야 한다.

3가지 방법으로 등록한다.

1. `@Bean`으로 수동 등록
2. `@Component`로 컴포넌트 스캔을 사용해서 자동 등록
3. `@Import`를 사용해서 파일 추가

### Advice 종류

Advice는 실질적으로 프록시에서 수행하게 되는 로직을 정의하게 되는 곳이다. 스프링에서는 Advice에 관련된 5가지 어노테이션을 제공하는데, 이 어노테이션은 메소드에 붙여서 사용한다.

해당 메소드는 advice의 로직을 정의하게 되고, 어노테이션의 종류에 따라 **포인트컷에 지정된 대상 메소드에서 Advice가 실행되는 시점을 정할 수 있다.** 또한 속성값으로 포인트컷을 지정 할 수 있다.

- `@Around`
    - **아래 4가지 어노테이션을 모두 포함하는 어노테이션**
    - 메서드 호출 전후 작업 명시 가능
    - 조인 포인트 실행 여부 선택 가능
    - **반환값 자체를 조작 가능**
    - **예외 자체를 조작 가능**
    - 조인 포인트를 여러번 실행 가능(재시도)
- `@Before`
    - 조인 포인트 실행 이전에 실행(실제 target 메서드 수행 전에 실행)
    - 입력값 자체는 조작 불가능
    - 입력값의 내부에 setter같은 수정자가 있으면 내부값은 수정 가능
- `@AfterReturning`
    - 조인 포인트가 정상 완료 후 실행(실제 target 메서드 수행 완료 후 실행)
    - 반환값 자체는 조작 불가능
    - 반환값 내부에 setter같은 수정자가 있다면 내부값은 수정 가능
- `@AfterThrowing`
    - 타켓 메서드가 수행 중 예외를 던지면 Advice 기능 수행
    - 예외는 조작 불가능
- `@Afeter`
    - 타켓 메서드의 결과에 관계없이 타켓 메서드가 완료되면 Advice 기능 수행

사용하는 예제 코드를 보면

```java
@Component
@Aspect
public class PerfAspect {

    @Around("execution(* com.example..*.EventService.*(..))")
    public Object logPerf(ProceedingJoinPoint pjp) throws Throwable {
        long begin = System.currentTimeMillis();
        Object reVal = pjp.proceed();
        System.out.println(System.currentTimeMillis() - begin);
        return reVal;
    }
}
```

스프링 AOP는 빈에서만 동작한다. 따라서 아까 말한 세가지 방법 중 선택해서 스프링 빈으로 등록해준 뒤 사용하면 된다. @Aspect 어노테이션을 붙이면 해당 클래스가 Aspect라는 것을  명시해준다.

logPerf()메소드는 @Around 어노테이션의 execution을 통해 Advice를 적용할 범위를 지정할 수 있다.

예제 코드로 보면 com.example 밑의 모든 클래스에 적용하고, EventService 밑의 모든 메서드에 적용한다

다른 코드를 보면

```java
@Component
@Aspect
public class PerfAspect {

  @Around("@annotation(PerfLogging)")
  public Object logPerf(ProceedingJoinPoint pjp) throws Throwable {
    long begin = System.currentTimeMillis();
    Object retVal = pjp.proceed();
    System.out.println(System.currentTimeMillis() - begin);
    return retVal;
  }
}
```

위와 같이 @Around 어노테이션에 @annotation(PerfLogging)처럼 적용될 어노테이션을 명시할 수 있다.

그럼 해당 메소드를 적용시킬 특정 메소드에 @PerfLogging 어노테이션을 붙여주기만 하면 logPerf() 기능이 동작한다.

이 코드는 Bean 전체에 기능을 적용 시킨다.

```java
@Component
@Aspect
public class PerfAspect {

  @Around("bean(simpleServiceEvent)")
  public Object logPerf(ProceedingJoinPoint pjp) throws Throwable {
    long begin = System.currentTimeMillis();
    Object retVal = pjp.proceed();
    System.out.println(System.currentTimeMillis() - begin);
    return retVal;
  }
}
```

@Around 어노테이션에 bean(simpleServiceEvent)처럼 적용될 빈을 명시할 수 있다.

이렇게 하면 해당 빈이 가지고 있는 모든 public 메서드에 해당 기능이 적용된다.

이거 말고도 사용법 예시는 무수히 많다

## 정리

코드 정의와 공정한 소프트웨어를 추구함에 있어서 **관점 지향 프로그래밍**은 중요한 역할을 한다.

즉, AOP는 공통 관심사의 체계적인 적용을 통해 코드의 균형 잡힌 측면을 달성하려는 목표를 가지고 있다.

AOP 유형을 이해하고 응용 프로그램의 라이프사이클 다양한 지점에서 어떻게 개입하는지 파악함으로 개발자는 코드 베이스 내에서 기능 및 비 기능적 측면 사이의 조화로운 균형을 유지할수 있다.

# 참고

https://github.com/cs-wiki/cs-wiki/issues/9

https://gmoon92.github.io/spring/aop/2019/04/20/jdk-dynamic-proxy-and-cglib.html

https://jaimemin.tistory.com/2025

https://docs.spring.io/spring-framework/reference/core/aop.html

https://code-lab1.tistory.com/193

[https://velog.io/@backtony/Spring-AOP-총정리#cglib과-jdk-동적-프록시-중-spring의-선택](https://velog.io/@backtony/Spring-AOP-%EC%B4%9D%EC%A0%95%EB%A6%AC#cglib%EA%B3%BC-jdk-%EB%8F%99%EC%A0%81-%ED%94%84%EB%A1%9D%EC%8B%9C-%EC%A4%91-spring%EC%9D%98-%EC%84%A0%ED%83%9D)
