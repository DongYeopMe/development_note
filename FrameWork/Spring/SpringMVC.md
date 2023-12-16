# **Spring MVC와 Dispatcher Servlet**

## Spring MVC란

우선 Spring Framework 모듈 중에는 웹 계층을 담당하는 몇 가지 모듈이 있는데, 

웹 계층에 서블릿 API를 기반으로 클라이언트의 요청을 처리하는 모듈이 스프링 웹 MVC이다.

여기서 서블릿은 클라이언트의 요청을 처리하도록 특정 규약에 맞춰 Java 코드로 작성하는 클래스 파일이다.

→ 아파치 톰캣은 이러한 서블릿들이 웹 애플리케이션으로 실행할 수 있도록 해주는 서블릿 컨테이너 중 하나이다.

Spring MVC 내부에는 서블릿을 기반으로 웹 애플리케이션이 동작하며, 스프링 부트는 기본적으로 아파치 톰캣이 내장되어 있다.

## MVC**(Model, View, Controller)**

MVC는 애플리케이션을 구성하는 요소를 역할에 따라서 3가지 모듈로 나누어 구분하는 패턴이다.

### Model

Spring MVC 기반의 웹 애플리케이션이 클라이언트의 요청을 전달 받으면 요청 사항을 처리하기 위한 작업을 한다.

처리한 작업의 결과 데이터를 클라이언에게 돌려줘야 하는데, 이때 클라이언트에게 응답으로 돌려주는 작업의 처리 결과 데이터를 Model이라고 한다.

- 사용자가 이용하려는 모든 데이터를 가지고 있어야 하며, View 또는 Controller에 대해 어떠한 정보도 알 수 없어야 한다.
- 변경이 일어나면 처리 방법을 구현해야 한다.

### View

View는 Model을 이용해 웹 브라우저와 같은 애플리케이션의 화면에 보이는 리소스를 제공하는 역할 한다.

- Model이 가지고 있는 데이터를 저장하면 안된다.
- Model이나 Controller에 대한 정보를 알면 안되며 단순히 표시를 해주는 역할을 가진다.
- 변경이 일어나면 처리 방법을 구현해야 한다.

### Controller

Controller는 클라이언트 측의 요청을 직접적으로 전달받는 엔드포인트이며, Model 과 View를 연결해주는 역할 한다.

클라이언트 측의 요청을 전달 받아 비즈니스 로직을 거친 후, Model 데이터가 만들어 지면, 이 Model를 View에게 전달하는 역할 한다.

- Model 또는 View에 대한 정보를 알아야한다.
- Model 또는 View의 변경을 인지하여 대처해야 한다.

### MVC 구조

Spring MVC의 주요 구성요소는 Model, View, Controller지만, 이 구성요소들이 유기적으로 동작하도록 하기 위해 다양한 구성요소가 함께한다.

- DispatcherServlet(Front Controller) : 제일 앞에서 HTTP Reequest를 처리하는 컨트롤러
- Controller(Handler) : HTTP Request를 처리해 Model을 만들고 View를 지정
- Model,View : Controller에 의해 반환된 Model,View가 Wrapping 된 객체
- ViewResolver : Model,View를 처리해서 View를 그린다.

### MVC 패턴

**MVC1**

MVC1 패턴은 사용자로부터 요청이 들어오면 DB로부터 필요한 데이터를 받은 Model 객체를 JSP 페이지에 담아 응답으로 보내는 패턴이다.

![스크린샷 2023-08-27 오후 11 32 15](https://github.com/mo2-Study-Group/StudyGroup/assets/112863029/79b78bab-742d-4ed7-9abc-378817c97181)

이 구조는 **JSP가 View와 Controller 역할을 모두 담당**하기 때문에 JSP page 내에 너무 많은 코드가 들어가 가독성도 떨어지고, 복잡해진다.

이러한 문제점을 보완해 Controller 역할을 하는 서블릿이 추가된 **MVC2** 패턴이 나왔다.

**MVC2**

![스크린샷 2023-08-27 오후 11 43 01](https://github.com/mo2-Study-Group/StudyGroup/assets/112863029/306b25de-929f-4c0f-9aea-0cba4c6d2b1f)

**MVC2** 패턴의 **서블릿은 요청에 대한 비즈니스 로직을 처리한 후**, 이를 **JSP 파일에 반영**하는 역할을 수행한다.

**Spring Framework에서 MVC2 기반**으로 좀 더 발전시켜 나온 것이 **Spring MVC**인 것이다. 자세한 설명은 아래 프론트 컨트롤러를 보면서 순차적으로 살펴보겠다.

### 프론트 컨트롤러 패턴

프론트 컨트롤러를 사용하지 않은 패턴 그림

![스크린샷 2023-08-27 오후 9 46 56](https://github.com/mo2-Study-Group/StudyGroup/assets/112863029/51ffa97d-a328-47a0-b388-4ca591a96c27)

각 클라이언트들은 Controller A, B, C에 대해 각각 호출한다. 

공통 코드들은 별도로 처리되어 있지 않고, 각 Controller에 포함되어 있다. 

이 경우 공통 코드를 수정할 때 하나 하나 다 수정 해야하고, 관리가 힘들어 유지보수 측면에서 좋지 않다.

여기서 도입을 하면

![스크린샷 2023-08-27 오후 9 55 11](https://github.com/mo2-Study-Group/StudyGroup/assets/112863029/3195f784-1987-4a34-918d-c64d142543c6)

이렇게 도입하면, 각 클라이언트들은 프론트 컨트롤러에 요청을 보내고 프론트 컨트롤러는 각 요청에 맞는 컨트롤러를 찾아서 호출시킨다.

공통 코드에 대해서는 프론트 컨트롤러에서 처리하고, 서로 다른 코드들만 각 컨트롤러에서 처리할 수 있도록 해준다.

장점으로는

- 공통 코드 처리가 가능해진다.
- 프론트 컨트롤러 외 다른 컨트롤러에서 서블릿을 사용하지 않아도 된다.

스프링 웹 MVC 의 DispatcherServlet이 프론트 컨트롤러이다.

## **Dispatcher Servlet**

Dispatcher Servlet는 스프링 기반 웹 애플리케이션 내에서 들아오는 요청을 처리하고, 제어 흐름을 관리하는 데 중요한 역할을 하는 Spring Framework의 핵심 구성 요소이다.

HTTP 프로토콜로 들어오는 모든 요청을 가장 먼저 받아 적합한 컨트롤러에 위임해주는 프론트 컨트롤러 이다.

즉 그림은 아래와 같이 공통 기능을 DispatcherServlet이 관리하는 것이다.

![스크린샷 2023-08-27 오후 10 06 21](https://github.com/mo2-Study-Group/StudyGroup/assets/112863029/bdb56169-e1d4-4740-8220-bd3b377dc9db)

그림을 보면 클라이언트로부터 어떠한 요청이 들어 오면 톰캣과 같은 서블릿 컨테이너가 요청을 받게 된다. 

모든 요청은 프론트 컨트롤러인 Dospatcher Servlet이 가장 먼저 받게 되는데, Dispatcher Servlet은 공통적인 작업을 먼저 처리한 후에 해당 요청을 처리해야 하는 컨트롤러를 찾아서 작업을 위임한다.

<aside>
💡 프론트 컨트롤러 : 주로 서블릿 컨테이너의 제일 앞에서 서버로 들어오는 클라이언트의 모든 요청을 받아서 처리해주는 컨트롤러로써, MVC 구조에서 함께 사용되는 디자인 패턴이다.

</aside>

### 장점

Spring MVC는 디스패처 서블릿이 등장함에 따라 web.xml의 역할을 상당히 축소시켰다. 

기존에는 모든 서블릿에 대해서 URL 매핑을 하기 위해 web.xml 파일에 모두 등록해야 했지만, Dispatcher Servlet이 해당 어플리케이션으로 들어오는 모든 요청을 핸들링하고 공통 작업을 처리해줘서 상당히 편리해졌다.

즉, 개발자는 컨트롤러를 구현해두기만 하면 **Dispatcher Servlet**이 알아서 컨트롤러로 위임을 해주는 구조가 된 것이다.

Dispatcher Servlet의 기능처리 그림

![스크린샷 2023-08-27 오후 10 33 50](https://github.com/mo2-Study-Group/StudyGroup/assets/112863029/cd5b7d8e-f376-4765-8ca8-74535025c5ca)

Dispatcher Servlet이 모든 요청을 받아 각각의 컨트롤러로 매핑해주는 방식은 효율적으로 보인다. 

그러나 Dispatcher Servlet이 모든 요청을 처리하다 보니 이미지나 HTML 등과 같은 정적 리소스에 대한 요청까지도 모두 가로채 정적 리소스를 불러오지 못하는 상황도 발생하기도 한다.

이러한 문제를 해결하기 위해 2 가지 방안이 나왔는데,

1. **정적 리소르에 대한 요청과 애플리케이션에 대한 요청 분리**

첫 번째 방안은 클라이언트의 요청 자체를 2개로 분리하는 것이다.

- **/apps**의 URL로 접근할 경우 **Dispatcher Servlet**이 **처리**를 담당한다.
- **/resources**의 URL로 접근할 경우 Dispatcher Servlet이 컨트롤할 수 없는 요청이므로 담당하지 않는다.

이러한 방식은, 앞에 문제를 해결할 수 있지만, 소스 코드가 복잡해져서 **모든 요청에 대해 /apps 나 /resources URL을 붙여야 하므로 직관적인 설계가 힘들다.**

1. **애플리케이션에 대한 요청을 탐색하고, 없을 경우 정적 리소스에 대한 요청으로 처리**

두 번째 방안은 모든 요청에 대해 Dispatcher Servlet이 적합한 컨트롤러를 탐색하고, 해당 요청에 대한 컨트롤러를 찾을 수 없는 경우 2차적으로 설정된 정적 리소스 경로를 탐색해 리소스를 찾는 방식이다.

이렇게 영역을 분리하면 효율적인 리소스 관리가 가능해지고, 추후에 확장이 용이해진다.

## 스프링 다수의 요청을 처리 방식

Spring Web MVC는 스레드 풀을 사용하여 스레드 사용을 관리하고 최적화하는데, 스레드 풀은 스프링 애플리케이션이 배포되는 서블릿 컨테이너에 의해 제어된다.

**Tread Pool** 

- 동시에 여러 작업을 효율적으로 실행 및 관리하기 위해 서버에서 만드는 스레드의 모음이다.
- 이걸 사용하면 각 작업에 대해 새 Thread를 사용하는 대신에 이미 생성된 Thread PooL에 있는 Thread를 재사용한다.

Spring web MVC의 컨텍스트 내에서 단일 컨트롤러가 여러 요청을 동시에 처리할 수 있는 방법을 보면

1. **스레드 풀 관리**
    
    요청이 Spring Web MVC 애플리케이션에 도착하면 서블릿 컨테이너의 Thread PooL이 요청 처리를 담당한다. 들어오는 요청을 처리하기 위해 풀에서 사용한 Thread를 할당한다.
    
2. **컨트롤러 인스턴스**
    
    **컨트롤러의 단일 인스턴스**는 일반적으로 **여러 요청을 동시에 처리**하는데 사용한다. 
    
    이는 여러 Thread가 동일한 컨트롤러 인스턴스 내에서 메소드에 동시에 접근하고 실행할 수 있음을 뜻한다.
    
3. **스레드 안전성**
    
    **Spring Web MVC**의 **컨트롤러**는 Thread로부터 **안전하도록 설계**되었는데, 이는 여러 Thread 가 **동시에 액섹스한다고 가정하여 컨트롤러 메소드를 작성할 수 있음**을 뜻한다. 
    
    프레임워크는 **각 요청에 대해 컨트롤러 상태를 격리시켜 동시 요청 간의 충돌을 방지**한다.
    
4. **요청 속성**
    
    각 요청과 관련된 데이터는 메서드 매개변수로 컨트롤러 메소드에 전달되거나 HttpServletRequest 및 HttpServletResponse와 같은 요청 범위 개체를 통해 접근할 수 있다.
    
5. **동시 실행**
    
    **여러 요청이 동시에 처리되는 경우 각 요청에는** Thread Pool**에서 자체** Thread가 **할당**되는데, 이를 통해 컨트롤러 메소드가 동일한 컨트롤러 인스턴스에 의해 처리되더라도 서로 다른 요청에 대해 독립적으로 실행할 수 있다.
    
6. **응답 생성**
    
    각 컨트롤러 메소드는 요청 데이터를 기반으로 응답을 생성한다. 
    
    응답이 생성되면 서블릿 컨테이너에 의해 클라이언트로 다시 전송된다.
    
7. **스레드 재사용**
    
    요청이 처리된 후 Thread는 Thread  Pool로 반환되어 다른 수신 요청을 처리하는 데 사용할 수 있게 된다. 
    
    이 Thread 재사용 메커니즘은 Thread 생성 및 소멸의 오버헤드를 최소화하므로 각 요청에 대해 새 Thread를 생성하는 것보다 더 효율적이다.
    
    정리하면
    
     Spring Web MVC의 단일 컨트롤러는 서블릿 컨테이너의 Thread Pool을 활용하여 여러 요청을 동시에 효과적으로 처리할 수 있다. 
    
    프레임워크는 Thread 안전성과 요청 데이터의 격리를 보장하여 동일한 컨트롤러 인스턴스가 충돌 없이 여러 요청을 처리할 수 있도록 한다. 
    
    이 설계를 통해 시스템 리소스를 효율적으로 활용하고 애플리케이션 응답성을 향상할 수 있다.
    
    # 참고
    
    - [https://velog.io/@suhongkim98/Front-Controller-패턴과-spring-MVC](https://velog.io/@suhongkim98/Front-Controller-%ED%8C%A8%ED%84%B4%EA%B3%BC-spring-MVC)
    - https://yeonyeon.tistory.com/103
    - https://mangkyu.tistory.com/18
    - https://mozzi-devlog.tistory.com/8
    - https://kotlinworld.com/326
    - [https://ittrue.tistory.com/234#:~:text=Intro,또는 스프링 MVC라고 한다](https://ittrue.tistory.com/234#:~:text=Intro,%EB%98%90%EB%8A%94%20%EC%8A%A4%ED%94%84%EB%A7%81%20MVC%EB%9D%BC%EA%B3%A0%20%ED%95%9C%EB%8B%A4).
