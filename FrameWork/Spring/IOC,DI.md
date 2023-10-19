
# IOC

## 개념

모든 종류의 작업을 사용하는 쪽에서 제어하는 구조를 거꾸로 뒤집는 것입니다.

제어권을 상위 템플릿 메소드에 넘기고 자신은 필요할 때 호출 되어 한다는 개념입니다.

IOC가 적용된 대표 기술이 프레임워크이며, 스프링 프레임워크로 예로 들면 Controller,Service 같은 객체들의 동작을 우리가 직접 구현 하지만, 해당 객체들이 어느 시점에 호출될지는 개발자가 제어하지 않는다. 프레임 워크가 해당 객체들을 갖고 생성,호출,소멸 시킨다.

이러한 과정을 개발자의 제어권이 역전됐다 라고 볼수 있습니다.

IOC를 적용하면 객체를 클래스 내부에서 직접 생성하여 사용하지 않고, 미리 생성해놓은 객체를 주입 받아 사용하면된다.

## 스프링에서는?

스프링 쓰기 전엔 개발자가 프로그램의 흐름을 제어해서 일일이 다 설정해줘야 했지만,

스프링에서는 프로그램의 흐름을 프레임워크가 주도한다. 객체(빈)의 생성부터 생명 주기 관리를 컨테이너가 다 해준다.

즉 제어권이 컨테이너로 넘어가게 되기에 이것을 제어권의 흐름이 바뀌었다고 해서 IOC이라고 합니다.

제어권이 컨테이너로 넘어오면서  DI,AOP등이 가능해진것입니다.

## 비교

### 기존 객체 생성과정

1. 객체 생성
2. 의존성 객체 생성 → 클래스 내부에서 생성
3. 의존성 객체 메소드 호출

### 스프링에서 객체 생성 과정

1. 객체 생성
2. 의존성 객체주입 → 스스로 만드는게 아니라 제어권을 스프링에 위임해서 스프링이 만들어 놓은 객체를 주입
3. 의존성 객체 메소드 호출

그래서 스프링이 실행될 때 모든 의존성 객체를 다 만들어주고 필요한 곳에 주입시킨다.

<aside>
💡 Bean들은 싱글톤 패턴의 특징을 가지며, 제어의 흐름을 스프링에게 맡겨 작업을 처리하게 된다.

</aside>

## 장점

- 의존성을 역전 시켜서 객체 간에 결합도를 줄일수 있다.
- 유연한 코드를 작성할 수 있게 될수 있어서 가독성 및 코드 중복, 유지보수를 편하게 할수 있다.

# DI


## 개념

- 객체를 직접 생성하는 것이 아닌 외부에서 생성한 후 주입 시켜주는 방법
- 의존 관계 주입 (의존성 주입)

### 의존성이란?

의존하는 대상 B가 변하면, 이게 A에 영향이 간다는 뜻이다. 

## 의존성 주입 방식

### 필드 주입

필드에 직접 어노테이션을 붙혀 주입하는 방식

```java
public class Main{
		public static void main(String[] args){
			@Autowired private MemberRepository memberRepository;
			@Autowired private DiscountPolicy discountPolicy;
	}
}
```

- DI 프레임 워크 없으면 사용할 수 없는 방식
- 코드가 간결하지만, 외부에서 변경이 불가능하기 때문에 테스트 하기 힘들다.
- 애플리케이션의 실제 코드와 관계없는 테스트 코드나 스프링 설정 파일인 Configuration에서만 간편하게 사용한다. → 실제 코드에선 사용 X

### 생성자 주입

Constructor Injection

순수 자바

```java

public class Controller{
	private Service service;

	public Controller(Service service){
		this.service = service;
	} 
	
}
public class Service {}

public class Main{
		public static void main(String[] args){
			Controller controller = new Controller(new Service());
			Controller controller2 = new Controller(null);
	}
}
```

프레임워크 사용

```java
public class Main{

	public static void main(String[] args) {
			private MemberRepository memberRepository;
			private DiscountPolicy discountPolicy;
		
			@Autowired
			public Main(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
	}

}
```

- 생성자를 통한 주입의 경우 해당 의존 객체를 필수로 주입해주지 않는 한 객체를 생성할 수 없게 된다.
- null을 직접적으로 주입 하지 않는 이상 객체는 NullPonterException을 발생하지 않습니다.
- 추가로 의존 객체에 final 키워드를 사용할 수 있다.
- 불변, 필수 의존관계에 사용한다.
- 생성자 하나가 있으면 @Autowired 생략 가능

### Setter 주입

순수 자바

```java
public class Controller{
	private Service service;

	public void setService(Service service){
		this.service = service;
	}
	public void servicePrint(){
		service.print();
	}
}
public class Service{
	public void print(){
		System.out.println("todo");
	}
}
public class Main{
	public static void main(String[] args){
		Controller controller = new Controller();
		controller.setService(new Service());
		controller.servicePrint();

		Controller controller2 = new Controller();
		controller2.servicePrint(); // NullPointerException
	}
}
```

프레임워크 사용

```java
public class Main{

	public static void main(String[] args) {
			private MemberRepository memberRepository;
			private DiscountPolicy discountPolicy;
		
			@Autowired
			public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
	    }
			@Autowired
			public void setDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy= discountPolicy;
	    }
	}

}
```

- 객체 생성시 의존 관계를 주입 해주지 않아도 객체 생성이 가능하게 된다.
- 그렇기 때문에 사용시에 NullPonterException이 발생할 수 있어 항상 객체의 참조가 null인지 확인하는 로직이 추가해야한다.(@Autowired 필수)
- 선택, 변경 가능성이 있는 의존관계에 사용한다.

## 순환 참조

![순환참조](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/fd511dcc-5a14-4a04-bd7d-babcf701503e)
프로젝트 규모가 커지게 되면 의존성 주입을 서로 하게 되면서 순환 참조가 일어나게 된다.

- 생성자 주입은 앱 구동 단계에서 스프링이 오류를 찾아낸다.
- setter 주입은 런타임시, 해당 순환 참조가 되어 있는 메소드를 호출하기 전까지 알수 없다.
  - 수정 → 2.6버전 부터 컴파일 에서 잡아줄수 있다고 합니다 업데이트 됐다고 하네요
예제 코드

```java
@Service
@AllArgsConstructor
public class TestService1{
	private TestService2 testService2;
}
@Service
@AllArgsConstructor
public class TestService2{
	private TestService1 testService1;
}

```

이렇게 생성자 주입으로 서로 순환 참조를 하도록 설정하고 스프링 실행하면 이런 오류가 발생한다.

`Error creating bean with name 'testService1' : Requested bean is currently in creation: Is there an unresolvable circular reference?`

sertter 주입은 문제 없이 되어서 정상적으로 동작되는 것처럼 보인다.

그래서 나중에 버그가 발생해도 찾기 힘든 문제로 나타날수도 있다.
수정 → 2.6버전 부터 컴파일 에서 잡아줄수 있다고 합니다 업데이트 됐다고 하네요
## 정리

스프링 공식 사이트에서 생성자 기반 주입을 옹호한다고 합니다.

의존성과 설정 값을 생성자 주입으로 해야하는 이유는

1. 불변
    - 대부분의 의존관계 주입은 한번 일어나면 애플리케이션 종료 시점까지 의존관계 변경할 일이 없다. 오히려 대부분의 의존관계는 애플리케이션 종료 전까지 변하면 안된다.
    - setter 주입을 사용하면, set~~ 메소드를 public으로 열어두어야한다.
    - 누군가가 실수로 변경할 수도 있고, 변경하면 안되는 메소드를 열어 두는건 좋은 설계 방법이 아니다.
    - 생성자 주입은 객체를 생성할 때 딱 1번만 호출하면 되므로 이후에 호출 되는 일이 없다. 따라서 불변하게 설계할수 있다.
2. 누락
    - 생성자 주입을 사용하면 Null PointException을 방지할 수 있다.
    - 생성자 주입을 사용하면 순환 참조를 앱 구동 시 검출할 수 있다.
3. fianal 키워드
    - 필드에 final 키워드를 사용해서 생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에 막아준다.
4. SRP(단일 책임 원칙)을 지킬 수 있도록 유도한다.
    - 주입하기 번거롭고 인자가 많아지면 코드가 길어 지기때문에 리펙토링을 하게 한다.

항상 생성자 주입으로 하다가 필수 값이 아닌 경우에만 setter 주입 방식을 옵션으로 부여하면 된다.

## @Autowired

- DI 할때 사용하는 어노테이션이며, 의존 관계의 해당하는 빈을 찾아 주입한다.
- 스프링 서버가 올라갈때(스프링 실행될때) 애플리케이션 컨텍스트가 @Bean이나 @Service 등 이용해 등록된 빈들을 생성하고 @Autowired 어노테이션이 붙은 위치에 의존 관계 주입을 수행하게 된다.

### 코드
![Autowired1](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/22ca6e35-13b7-478a-9bd7-32d4e7f5ddb8)

주석 해석하면 실제 타깃에 Autowired가 붙은 빈을 주입하는 것은 BeanPostProcessor라는 내용을 찾을 수 있고, 그것의 구현체는 AutowiredAnnotationBeanPostProcessor인 것을 확인할 수 있다. 라고 합니다.

@Target : 생성자와 필드, 메소드에 적용 가능하다.

@Retention : 컴파일 이후 JVM에 의해 참조가 가능하다. 런타임 시 이 어노테이션의 정보를 리플렉션으로 얻을 수 있다.

리플렉션 : 구체적인 클래스 타입을 알지 못해도, 그 클래스의 메소드,타입,변수들에 접근할 수 있도록 해주는 자바 API.

### *****AutowiredAnnotationBeanPostProcessor 클래스*****

실제 타깃에 빈을 주입하는 역할 한다.

이 클래스에 processInjection() 메소드를 보면

![Autowired2](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/6aea95c0-536a-4ac0-a1f9-7f1f2d71b42e)

@Autowired로 된 필드나 메소드에 객체를 주입하는 메소드다. 이 메소드 안에 InjectionMetadata.inject()메소드보면

![Autowired3](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/d219c919-0c11-45e8-9207-7edb9bd07f1b)

객체를 주입할 때 ReflectionUtils 클래스를 사용하는 것을 볼 수 있다. 즉, @Autowired는 리플렉션을 통해 수행된다고 나타나는 것이다.

## 옵션 처리

주입할 스프링 빈이 없이도 동작 해야할때가 있는데 @Autowired required 옵션의 기본값이 true로 되어 있어서 오류가 발생한다. 

### 자동 주입 대상 처리하는 방법

- @Autowired(required = false) :자동 주입할 대상이 없으면 setter 메소드 자체가 호출 안된다.
- @Nullable : 자동 주입할 대상이 없으면 null이 입력된다.
- Optional<> : 자동 주입할 대상이 없으면 Optional.empty 가 입력된다.

### 예제 코드

```java
//호출 자체가 안됨
@Autowired(required = false)
public void setNoBean1(Member member) {
 System.out.println("setNoBean1 = " + member);
}
//null 호출
@Autowired
public void setNoBean2(@Nullable Member member) {
 System.out.println("setNoBean2 = " + member);
}
//Optional.empty 호출
@Autowired(required = false)
public void setNoBean3(Optional<Member> member) {
 System.out.println("setNoBean3 = " + member);
}
```

# 스프링 컨테이너

스프링 컨테이너는 스프링 빈을 생성하고 관리한다. 스프링 컨테이너는 IOC Container 혹은 DI Container라고 부릅니다.

왜냐하면 스프링 컨테이너가 IoC,DI를 다 진행하기 때문이다. bean들을 생성하고, 의존 관계까지 연결 해주는 역할을 합니다.

![스프링 컨테이너](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/93ed5942-9f6e-465c-8483-0f0449cfda59)

BeanFactory와 Application Context로 나눌수 있는데

### BeanFactory

- 스프링 컨테이너의 최상위 인터페이스다.
- 스프링 빈을 관리하고 조회하는 역할이다.
- getBean()을 제공한다.

### ApplicationContext

- BeanFactory 기능을 모두 상속 받아서 제공한다.
- Bean Factory, ApplicationContext 둘의 차이점
    - 메세지 소스를 활용한 국제화 기능
        - 한국에서 쓰면 한국어로 영어권에서 쓰면 영어로 출력
    - 환경변수
        - 로컬,개발,운영등을 구분해서 처리
    - 애플리케이션 이벤트
        - 이벤트를 발행하고 구독하는 모델을 편리하게 지원
    - 편리한 리소스 조회
        - 파일, 클래스 패스,외부 등에서 리소스를 편리하게 조회

### 정리

- ApplicationContext는  BeanFactory의 기능을 상속 받기 때문에 빈 관리 기능 + 편리 부가 기능을 제공한다.
- 그래서 BeanFactory를 직접 사용할 일은 거의 없다.
- 이 두가지를 스프링 컨테이너라고 부른다.

# 참고

https://devmango.tistory.com/174

https://steady-coding.tistory.com/600

https://code-lab1.tistory.com/127

[https://www.nowwatersblog.com/springboot/springstudy/IOC & DI](https://www.nowwatersblog.com/springboot/springstudy/IOC%20&%20DI)

https://lee1535.tistory.com/117

[https://velog.io/@ohzzi/Spring-DIIoC-IoC-DI-그게-뭔데](https://velog.io/@ohzzi/Spring-DIIoC-IoC-DI-%EA%B7%B8%EA%B2%8C-%EB%AD%94%EB%8D%B0)

스프링 핵심 원리 -기본편 학습 자료 - 김영한

# 예상 면접 질문 및 답변

## IoC 컨테이너가 무엇이냐?

스프링 애플리케이션은 스프링 컨테이너가 객체(빈)의 생성, 빈들 간의 관계 설정, 빈을 사용 및 삭제 등의 작업을 담당하는데 이를 IoC 컨테이너 라고 한다.

## IoC 컨테이너를 사용하면서 장점은 무엇이냐?

객체(빈)을 IoC 컨테이너가 알아서 관리 해주면서 개발자가 복잡하고 실수하기 쉬운 로우 레벨 기술에 신경 안쓰고, 비즈니스 로직에 더 집중할수 있다.

## DI가 무엇이냐?

객체(빈)들 간의 의존관계를 외부에서 결정하고 주입하는 것이다.

DI를 사용하면 의존대상이 변화했을때 이에 맞게 수정안해도 됩니다.

## DI의 장점은 무엇이냐?

- 의존성이 줄어들어 주입 받는 대상이 변하더라도 코드 수정할 일이 없거나 줄어든다.
- 내부에서만 사용하던걸 다른 클래스에서 재사용할수 있다.(재사용성이 높은 코드가 된다.)
- 모의 객체를 주입할 수 있기 때문에 테스트가 쉬워진다.
- 코드가 더 깔끔해진다.

## 의존성 주입 방식의 종류는?

생성자,setter,필드 주입이 있다.

생성자 주입은 생성자 호출 할때 딱 1번만 호출하고 변경되지 않기때문에 불변,필수 의존관계에 사용되고,

setter 주입은 선택,변경 있는 의존관계에 사용되며, 옵션으로 사용할때 쓰인다.

필드 주입은 애플리케이션 외부에서 변경이 불가능 하기 때문에 테스트 하기 힘들고, 주로 애플리케이션과 관계없는 테스트코드나 @Configuration 같은 스프링 설정 목적으로 쓰인다.

## 생성자 주입을 사용하는 이유는 무엇인가요?

- 해당 의존 객체를 필수로 주입해주지 않는 한 객체를 생성할 수 없기 때문에 NullPointException이 발생하지 않는다.
- final 키워드를 사용할 수 있어서 불변하게 설계할수 있다.
- 컴파일 단계에서 오류가 나타나 순환 참조 현상을 검출 할수 있다.
- 의존성을 주입하기 번거롭고, 생성자 인자가 많아져 코드가 길어지면 SRP 원칙을 생각하게 되고 리팩토링을 하게 유도한다.

## 순환 참조가 무엇이고, 언제 발생하나요?

서로 다른 빈들이 서로를 참조를 주입하면서 생기는 현상입니다.필드 주입 ,setter 주입을 사용하면 애플리케이션 구동 당시엔 발생하지 않고, 객체(메소드)를 호출 하는 시점에 발생한다.

## Spring Ioc/DI 동작 과정

IoC는 외부에서 관리 하는것이므로, bean들을 스프링 컨테이너에서 관리한다.

DI는 스프링 컨테이너가 bean 사이의 의존 관계를 정보를 바탕으로 자동으로 연결 해준다.

## Autowiring  동작 과정

스프링이 실행 될때(스프링 서버가 올라갈 때) 애플리케이션 컨텍스트가 @Bean,@Service  등 같은 어노테이션을 이용해 등록한 스프링 빈들을 생성하고, @Autowired 가 붙은 위치에 의존성 주입을 한다.
