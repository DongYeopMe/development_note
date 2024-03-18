# 구현

이번엔 채팅 기능을 구현해야해서 찾아보았습니다. 처음엔 Web Socket 방식으로 하겠구나 했지만, 채팅 기능 구현한 블로그들을 보고 여러가지가 있어서 정리해보았습니다.

일단 많이 쓰이는것이 Kafka,Stomp,MogoDB,WebSocket 등등 여러가지가 있었습니다.

## Web Socket

웹소켓은 HTML5 표준 기술로, HTTP 환경에서 클라이언트와 서버 사이에 하나의  TCP 연결을 통해 실시간으로 전이중 통신을 가능하게 하는 프로토콜이다.

클라이언트와 서버간 연결을 지속적으로 유지하고 서버→ 클라이언트로, 클라이언트→서버로 데이터를 보낼수 있게 됨으로 양방향 통신을 할수 있다.

서비스 : 게임,채팅 등등(실시간성!)

<aside>
💡 참고로 웹소켓은 요청과 응답 개념이 아닌 서로 데이터를 주고 받는 방식.

</aside>

그래서 웹소켓을 선택할수도 있지만, 더 좋은 기술이 있다고 합니다.

### Web Socket vs HTTP

어떤게 실시간성에서 더 유리한가?

![image](https://github.com/DongYeopMe/development_note/assets/70151275/86eee12d-bdad-4d42-a854-616d526e1a32)

HTTP는 

- 비연결성
- 매번 연결 맺고 끊는 과정 비용이 많이 든다.
- Request - Response 의 구조

Web Socket

- 연결지향적
- 한번 연결 맺으면 계속 유지되는 실시간성을 보장
- 양뱡향 통신

1. HTTP는 클라이언트가 데이터를 얻고자 하면 항상 서버에 요청을 보내야하지만, 웹소켓은 연결이 계속 유지 되기 때문에 클라이언트가 보낸 메세지를 서버는 받기만하면된다.

1. HTTP에 비해 웹소켓이 보내야 하는 메시지, 데이터 양이 훨씬적다.
    
    HTTP 요청을 보낼시 Request URL, 상태코드, Request&Response 헤더와 바디 등등 데이터 양이 매번 요청으로 가공해서 서버로 보내지면, 실시간성을 요구하는 서비스에선 꽤 부담이 생길것이다.
    
2. 웹 소켓을 사용할때 HandShake를 하는 과정은 HTTP 프로토콜을 통해 진행하겠지만, 한번 연결이 수립된 후로는 간단한 메시지만 오가는 방식.

그래서 HTTP쪽 보단 웹소켓이 더 좋은거같다.

## STOMP

웹 소켓 프로토콜은 텍스트,바이너리 유형의 메시지를 정의하고 잇지만 메시지 내용까지는 정의하지 않습니다. 그리고 그 메시지를 어떤식으로 주고 받을진 따로 정해진게 없습니다.

프로젝트 투입 인원이 많을수록 클라이언트와 서버가 어떤 형식인지,타입,데이터 구분 등등 따로 정의 해야 복잡하지 않습니다.

여기서 Stomp는

메세지 전송을 효율적으로 하기 위해 등장한 프로토콜이다. 기본적으로 pub/sub구조를 따르고 있기 때문에 메시지를 전송하고 메시지를 받아 처리하는 부분이 명확하다는 장점이 있습니다.

 

Stomp를 이용해 채팅 시스템을 개발한다면 크게 두가지 행동이 있습니다.

1. 채팅방 입장 : 현재 입장한 채팅방 번호를 구독한다.
2. 채팅방에서 메시지를 보내거나 받기 : 구독한 채팅방에서 메시지를 송수신 할수 있다.

Stomp를 사용하는 장점

1. 메시징 프로토콜과 메시징 형식을 개발할 필요가 없다.(하위 프로토콜과 메시지 컨벤션을 정의할 필요없다.)
2. 연결 주소마다 새로운 핸들러를 구현하고 설정할 필요 없다.
3. 메시지 브로커를 사용하면 구독을 관리하고 메시지를 Broadcast하는데 사용할 수 있다.
    1. Publisher으로 부터 전달 받은 메시지를 Subscriber로 전달해줄 때 중간에서 메시지를 주고 받게 해주는 게 메시지 브로커.

stomp를 사용하면 기본적으로 In-Memory Message Broker를 사용하는데 이러면 여러가지 문제가 생긴다. 이부분은 외부 브로커를 사용하면서 문제 해결할 수 있기에 큰제약이 아니다.

외부 브로커 : **RabbitMQ, Kafka, Active MQ**

사용하는 이유는 https://leejincha.tistory.com/263 이렇게 정리 잘 해주셨다.

### Stomp Frame

구조는 

![image](https://github.com/DongYeopMe/development_note/assets/70151275/fd047424-507d-40d7-b006-7a2c40157ef0)

왼쪽은 웹소켓,오른쪽이 Stomp를 사용했을때

웹소켓만 사용했을땐 서버에서 보는 메세지만 오고가고, Stomp는 여러가지 형식의 데이터가 들어오는데,

1. Connect

연결에 관한 구조. 버전정보와 현재의 세션정보를 가져온다.

예제

```
<<<CONNECTED

version:1.1

heart-beat:0.0

user-name:user1

```

1. Subscribe

구독이라는 개념을 이용하여 현재 메세지에 대한 목적지를 설정한다.

구조를 보면 각각의 destination이 있고, connect 이후 subscribe를 설정하게 된다. 

등록되지 않은 subscribe를 호출 시 찾을 수 없기에 정확한 통신이 되지 않는다.

예제

```
>>> SUBSCRIBE

id : sub-0

destination:/topic/roomId

 

>>> SUBSCRIBE

id : sub-1

destination:/topic/out

 

>>> SUBSCRIBE

id : sub-2

destination:/topic/in
```

1. Message

메세지 전송 구조

메세지 전달지와 해당 메세지의 정보들이 출력된다. 

이 데이터 타입은 Json으로 전송된거며, 목적지는 /topic/roomId이며,

메시지 길이는 43, body 부분은 데이터가 정의되어 있다.

데이터는 Key,value로 되어 있고, 다양한 데이터 타입을 가질수 있다.

```
<<<MESSAGE

destination : /topic/roomId

content-type:application/json;charset=UTF-8

subscription:sub-0

message-id:2oy02l5d-71

content-length:43

{"senderName : "user1" , "sendMessage":"hi?"}
```

1. Disconnect

연결을 종료했을 시 구조

### 동작 흐름도

크게보면

![image](https://github.com/DongYeopMe/development_note/assets/70151275/9ae0e02b-ed47-4b86-bf93-bfb86a41a42c)

### 기본 Stomp 기반의 통신 흐름

![image](https://github.com/DongYeopMe/development_note/assets/70151275/36eb3b33-e079-4dd2-90c8-25c9f294cd7c)

Send→ request → response → Message를 받게 된다.

Send는  “/topic”, “/app”으로 보내고,  “/app”은 channel broker를 등록하고,  “/topic”을 통해 통신한다.

자세하게 설명하면.

발신자 : 메시지 송신, 구독자 : 메시지 수신.

구독자는 “/topic” 이라는 경로를 구독하고 있다고 가정을 하면,

발신자는 바로  “/topic” 이라는 경로를 통해 “/topic”이라는 헤더를 destination 헤더로 넣어서 메시지를 송신할수도 있지만, 서버내 에서 어떤 처리, 혹은 가공이 필요하다면 이 “/app” 이라는 주소로 메시지를 송신하게 된다.

그리고 서버가 모두 마쳤다면 처리된 메세지를 “/topic”에 경로에 다시 전송하게 되면, 메시지 브로커에게 전달되고, 그다음 브로커가 메세지를 “/topic”를 구독 하고 있는 구독자들에게 최종적으로 전달 하게 된다.

### Message Queue를 활용한 Stomp

![image](https://github.com/DongYeopMe/development_note/assets/70151275/f671abc0-966a-438e-92e7-d91968e13149)

Message Broker는 채널에 /topic 프로토콜 데이터가 왔을때 가로채어 MQ(Message Queue)에 연결된 모든 어플리케이션에게 메시지를 전달하게 되는 구조다.

Broker는 토픽에 따라 메세지 전달해야하는 사용자를 구분한다.

### 그래서 선택은?

WebSocket으로 하면 채팅방에 현재 누가 접속 중인지와 채팅방에 있는 사용자들에게 모두 메시지를 보내야해서 채팅방마다 세션을 관리 해야하는 번거로움이 있습니다.

Stomp로 채팅을 구현한다면, 사용자가 채팅방에 접속 했을대 해당 채팅방에 구독(소속)하기만하면 접속중인지 따로 관리 안해도 메시지 보낸 사용자가 포함된 채팅방의 모든 이용자에게 메시지가 갈수 있다는 강점을 가지고 있습니다.

그래서 Stomp를 사용하기로 결정하였습니다.

# 참고

[https://velog.io/@msung99/웹소켓이란#server-sent-events](https://velog.io/@msung99/%EC%9B%B9%EC%86%8C%EC%BC%93%EC%9D%B4%EB%9E%80#server-sent-events)

[https://velog.io/@ch4570/Stomp-Kafka를-이용한-채팅-기능-개발하기-with-Spring-Boot-1-Kafka와-Stomp는-무엇일까](https://velog.io/@ch4570/Stomp-Kafka%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EC%B1%84%ED%8C%85-%EA%B8%B0%EB%8A%A5-%EA%B0%9C%EB%B0%9C%ED%95%98%EA%B8%B0-with-Spring-Boot-1-Kafka%EC%99%80-Stomp%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C)

https://faith-developer.tistory.com/194

https://wondongho.tistory.com/73
