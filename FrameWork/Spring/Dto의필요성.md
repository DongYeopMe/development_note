만약 dto를 안쓰면 어떻게 될까?
일어나는 상황 : 예(Member, MemberResponse)
엔티티와 API를 1:1 대응 관계를 가지면 엔티티 칼럼? 데이터를 수정을했을때 API 스펙이 변경을 항상해야한다.
→ 여기서 API스펙이란? : 소프트웨어 구성 요소들이 상호 작용하기 위한 정의된 규약 (규칙)을 문서화 한 것
엔티티 하나에 여러개의 API 스펙이 필요하기 때문에 데이터들을 알맞게 넘겨 줄수 없다.
→ 기능 동작은 문제가 없는데, PW 같이 감추어야 하는건 숨길수 없기에 보안 문제가 발생한다.
엔티티 그대로 넘기면, 엔티티가 가진 정보 말고, 다른 정보들? 다른 것들을도 같이 넘겨줄수 없다.
만약 엔티티에 변경사항이 있을때 dto가 있다면 파악할수 있다.
우리가 main project를 할때 써먹을때 조금씩 생각이 났다.
dto, domain을 분리하는 이유
여기서 MVC 패턴을 사용하는 이유가 뭘까?
Meder,controller,View 각자 역할을 수행하기에 나중에 유지 보수가 편하기 때문이다.
Controoler는 clinet와 request/response 해주는 책임이 있고,
Moder은 DataBase에서 받아온 데이터를 다루는 책이 있다.
이렇게 관심사 분리를 통해 복잡한 시스템을 효율적으로 작동시킨다.
여기서 관심사 수평적 분리가 나와야한다.
![image](https://github.com/DongYeopMe/development_note/assets/70151275/b5822972-3d75-4ead-b665-9326fa648dfd)

이 관심사 수평적 분리는 애플리케이션 내에서 동일한 역할을 수행하는 논리적 계층으로 애플리케이션을 나눈것이다.
또 프로세스에 따라 MVC 패턴을 녹이면
![image](https://github.com/DongYeopMe/development_note/assets/70151275/741662ef-679c-439e-acea-b2e3f8ba69b6)

이런 그림이 나오는데
Presentation Layer는 애플리케이션의 기능과 데이터를 사용자에게 제공한다.
Business Layer : 애플리케이션의 핵심 비즈니스 로직 및 나머지 두 계층 간에 전달되는 데이터 처리한다.
Data Access Layer : 데이터 베이스와 상호 작용하는 역할한다.
여기서 DTO는
데이터를 전달하는 목적으로 1번 Presentation Layer에서 활동하고
Domain은 비즈니스 로직을 담는 Business Layer에서 활동한다.
만약에 관심사 분리를 안하면 레이어가 혼합된 god class가 된다.
약간 개발자들이 영업과 개발을 같이 하는 느낌임.
god class는 코드 변경이 있을때 두 레이어에 영향이 간다.
그래서 좋은 객체지향 설계는 하나의 객체에는 하나의 책임만 존재해야한다.
변경이 있을때 영향을 최소한으로 하기 위해서이다.
마틴 파울러의 DTO는 무슨 목적이냐??
주요 목적은 호출을 한번 했을대 여러 매개 변수를 일괄 처리해서 서버의 왕복을 줄이는거다.
그래서 모든 데이터를 가지고 있는 DTO 객체를 만들어서 네트워크 비용을 줄인다는 것이다.
참고
[Spring Boot] DTO 는 왜, 언제 사용할까?
https://www.inflearn.com/questions/182197/dto의-필요성이-와닿지-않습니다
