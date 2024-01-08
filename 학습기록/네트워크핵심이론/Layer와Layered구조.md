개발공부를 접하다 보면 Layer와 Layered란 용어를 많이 접하게된다. 이는 각각 층과 계층적이다라는 뜻을 의미하고 있는데 꼭 개발분야 뿐만 아니라 여러곳에서도 사용되고 있다.
예를 들면 헤어스타일링 중 층을 나누어 단차 형태의 스타일을 연출하는 레이어드컷, 피라미드처럼 여러 층들이 계단식으로 쌓여있는 형태 등 우리 주변에서 흔히 볼 수 있는것이 Layer와 Layered이다.
그렇다면 IT분야에서는 어떻게 사용되고 있는지 알아보자

## Layer & Layered
IT에서 구성 요소들 간에 관계를 나타내야 할때 도식화를 그려서 설명한다.
DB에서의 ERD관계를 나타내거나 디자인 패턴 등을 보면 여러 도형들이 선으로 연결되며 그림으로 표현이 많이 되는데 이를 도식화라고 한다. 
그러한 도식화 중 Layer & Layered는 하나의 특징을 가지고 있는데 층들간에 의존적인 관계를 가지고 있다. 

#### 소프트웨어 설계 관점

<img width="400" alt="image" src="https://github.com/mo2-Study-Group/StudyGroup/assets/107467750/a5c58968-f4bb-4d96-9265-02fa6ad8584e">

출처 : https://blog.doctornow.co.kr/tech/layered-architecture


소프트웨어 설계에 있어 보편적으로 사용되는 `Layered Architecture`의 그림이다.
요소들이 들어있으며 각 요소들을 묶어 레이어로 나누며 이것을 통틀어 레이어드 구조로 되어있다. 위와 같은 형태는 서버 개발을 진행하다보면 Controller, Service, Repository로 나누어 개발을 진행하는데 이를 도식화한 그림으로 봐도 무방하다. 
서론에서 설명한것 처럼 이 구조도 `Presentation Layer` 쉽게 비교하면 Controller 계층이 없다면 `Business Layer`가 즉 Service 계층이 존재할 수 있을까? 들어오는 데이터 혹은 요청이 없으니 동작 할수가 없게된다.

#### 네트워크 관점

<img width="400" alt="image" src="https://github.com/mo2-Study-Group/StudyGroup/assets/107467750/40d1c874-be17-4f80-9578-23e09c07d7f0">

출처 : https://shlee0882.tistory.com/110


네트워크하면 빠질 수 없는 OSI 7계층이다. 
이는 이름에서도 알 수 있듯이 7가지의 층으로 나누어 네트워크를 설명하는 도식화인데 이를 크게 나누어 `Application`, `Transport`, `Internet`, `Network Interface`로 4가지 분야가 있다. 
여기서도 마찬가지로 의존적인 특징을 알 수 있는데 Application 계층은 쉽게 말해 우리가 사용하고 있는 구글, MS워드 등 어플리케이션을 뜻하는데 여기서 우리가 검색하여 어떠한 자료를 불러오거나 이메일을 보낼때 Transport 계층에서 데이터를 주고 받을 수 있게 연결해주는 역할을 수행하게 된다.
만약 Application이 없다면 Transport 계층이 존재할 이유가 있을까? 데이터를 보내는 행위나 영상을 불러오는 행위등이 존재하지 않는다면 Transport 또한 일어날일이 없기에 의존적인 관계로 볼 수 있다.

* * *

지금까지 예시를 같이봤는데 여기서 기억해야할것은 Layer & Layered 라는 것은 층을 나누어 계층적인 형태가 띄는것을 뜻하며 또한 각 층끼리 의존적인 성질을 가질 수 있는 특징을 가지고 있다는 것을 기억한다면 될 것 같다.

