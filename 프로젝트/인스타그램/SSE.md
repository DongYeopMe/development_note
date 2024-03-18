# 개요

우리가 댓글이나 좋아요 같은 기능들을 썻을때 요청 없이도 실시간으로 서버 변경사항을 웹 브라우저에 보내줘야하는 서비스입니다. 하지만 Cient-Server 모델의 HTTP 통신으로는 이런 기능을 구현하기도 어렵고, 클라이언트의 요청이 있어야 서버가 응답을 할수 있기 때문이다.

그래서 어떤 방식으로 통신을 해야할지 알아보았습니다.

# 실시간 통신 방법

## Web Socket

![image](https://github.com/DongYeopMe/development_note/assets/70151275/ee770ee8-aeb8-4cd8-955d-ea58cf8254da)

웹소켓은 HTML5 표준 기술로, HTTP 환경에서 클라이언트와 서버 사이에 하나의  TCP 연결을 통해 실시간으로 전이중 통신을 가능하게 하는 프로토콜이다.

HTTP 통신에서는 연결을 지속하지 않고, 클라이언트가 서버로 단방향 요청만 가능하게 구성되어있는데,

이것이 등장한 이후는 클라이언트와 서버간의 실시간 통신이 가능하게 되었습니다.

그래서 클라이언트와 서버간 연결을 지속적으로 유지하고 서버→ 클라이언트로, 클라이언트→서버로 데이터를 보낼수 있게 됨으로 양방향 통신을 할수 있다.

서비스 : 게임,채팅 등등(실시간성!)

<aside>
💡 참고로 웹소켓은 요청과 응답 개념이 아닌 서로 데이터를 주고 받는 방식.

</aside>

## HTTP

사실 HTTP 방식도 실시간성을 보장하는 기법이 존재한다.

HTTP 와같은 단방향 프로토콜로 제한된 웹 환경에서 웹 브라우저와 서버 사이의 양방향 실시간 통신을 위한 다양한 기법들이 존재한다.

종류는 Polling,Long Polling, Streaming

### HTTP Polling

![image](https://github.com/DongYeopMe/development_note/assets/70151275/4402e668-4423-4b46-a2ea-d14b7d1428af)

Polling은 웹 브라우저가 주기적으로 서버에게 이벤트가 발생했는지 확인하는 방식이다.

만일 이벤트가 발생하지 않았다면 Response엔 이벤트 정보가 포함되지 않고, 반대로 이벤트가 발생했다면 포함되는 방식이다.

주기적으로 서버에게 이벤트 발생여부를 확인하기에 실시간성이 떨어지며, 주기적으로 요청/응답이 오가서 계속 트래픽이 발생하여 서버에 부담준다.

정리하면

- 클라이언트가 일정 주기로 서버에게 필요한 데이터를 요청및 응답하는 방식입니다.
- 가장 쉬운 방식이지만, 서버에 부담을 주게 됩니다.

### Long Polling

![image](https://github.com/DongYeopMe/development_note/assets/70151275/3ca72c28-2ae9-4a56-8c61-9d7bb353ab08)

Long Polling은 기존 Polling의 실시간성을 보완한 기법이다. 웹 브라우저가 요청을 전송할때 이벤트가 발생하기 전까지는 Response를 보내지않고,

이벤트가 발생 할때만 Response를 이벤트 정보와 같이 담아서 보내는 방식이다.

실시간성을 보완했지만, 서버 이벤트가 자주 발생하지 않는 환경에서 이용하는게 좋다.

이것 또한 이벤트가 자주 발생하면 비효율적인 트레픽이 발생해 오히려 폴링 방식보다 더 서버에 부담주는 경우도 있다.

정리하면,

- 서버는 클라이언트에게 즉시 응답을 주지 않고 이벤트가 발생했을때만 준다.
- 경우에 따라 Long Polling 방식이 Polling 방식보다 서버에 부하를 더 주는 경우도 있다고 한다.

### Server-sent Events

![image](https://github.com/DongYeopMe/development_note/assets/70151275/44675f01-c4a3-4749-b4a9-bb7aeb7ad558)

Server-sent Events 기법은 클라이언트 요청없이 서버에서 클라이언트에게 단방향으로 이벤트를 전송하는 기법이다. 방식은 웹 브라우저에서 서버쪽 특정 이벤트를 구독하면 해당 이벤트 발생시 이벤트를 전송한다.

하지만, 클라이언트에서 서버로 이벤트를 전송할 수 없는방식이며, 최대 동시 접속 횟수가 제한되어있다.

# SSE를 통해 통신 연결 및 알림 기능 구현

## 알림이 발생하는 과정

우리가 만드는 서비스는 다음과 같은 경우에 이벤트(알림)이 발생한다.

- 유저가 팔로우 했을 경우
- 사용자가 본인 댓글 또는 게시물에 댓글을 남길 경우
- 사용자가 유저를 태그 했을 경우
- 메시지가 올경우
- 본인 게시물 또는 댓글에 좋아요 누를경우

## 구현

Spring Framework 4.2부터 SSE 통신을 지원하는 SseEmitter 클래스를 이용해 구현하겠습니다.

## 구독

**Controller**

```java
@RestController
@RequiredArgsConstructor
public class NotificationController {

		private final AlarmService alarmService;

		@GetMapping("/subscribe/{username}")
		public ResponseEntity<SseEmitter> subscribe(@PathVariable("username") String username) {
		  return new ResponseEntity<>(alarmService.connectSubscribe(username), HttpStatus.OK);
		}
}
```

클라이언트에서 구독 요청 보내고 SSEEMITTER를 반환을 받아서 연결을한다.

**Service**

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class AlarmService {

	  private final Map<String, SseEmitter> emitters = new ConcurrentHashMap<>();

		@Transactional
    public SseEmitter connectSubscribe(String username) {
        securityUtil.checkLoginMember();
        memberRepository.findByUsername(username)
                .orElseThrow(() -> new BusinessException(MEMBER_NOT_FOUND));
        SseEmitter emitter = new SseEmitter();
        emitters.put(username, emitter);
        emitter.onTimeout(() -> emitters.remove(username));// 클라이언트 연결 시간이 초과될때 실행 작업
        emitter.onCompletion(() -> emitters.remove(username));// 연결 완료 될대 실행할 작업
        sendNotification(username, username + "님의 통신 연결이 완료되었습니다.");
        return emitter;
    }
}
```

connectSubscribe() 이 메소드는 구독 할때 쓰이는 메소드이다.

emitter.onTimeout(() -> emitters.remove(username))
emitter.onCompletion(() -> emitters.remove(username))

이 작업은 혹시나 중복 연결이 됐을때 오류를 방지하고자 추가했다.

Postman으로 동작 시키면 이렇게 연결이 된다.

![image](https://github.com/DongYeopMe/development_note/assets/70151275/a68283ab-9374-4169-98a5-2beaebb74180)

## 알림

**Service**

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class AlarmService {

	  private final Map<String, SseEmitter> emitters = new ConcurrentHashMap<>();

    public void sendNotification(String username, String message) {
        SseEmitter emitter = emitters.get(username);
        if (emitter != null) {
            try {
                emitter.send(SseEmitter.event().name("notification").data(message));
            } catch (IOException e) {
                emitter.complete();
                emitters.remove(username);
                throw new BusinessException(NOTIFICATION_CONNECTION_ERROR);
            }
        }
    }
}
```

이렇게 특정 이벤트가 발생했을때 거기에 맞는 메세지를 추가해 알림을 보내주는 메소드다.

```java
public enum AlarmType {
    FOLLOW("{agent.username}님이 회원님을 팔로우하기 시작했습니다."),

    LIKE_POST("{agent.username}님이 회원님의 사진을 좋아합니다."),
    MENTION_POST("{agent.username}님이 게시물에서 회원님을 언급했습니다: {post.content}"),

    COMMENT("{agent.username}님이 댓글을 남겼습니다: {comment.text}"),
    LIKE_COMMENT("{agent.username}님이 회원님의 댓글을 좋아합니다: {comment.text}"),
    MENTION_COMMENT("{agent.username}님이 댓글에서 회원님을 언급했습니다: {comment.text}");

    private String messageTemplate;

    public String createAlarmMessage(Alarm alarm) {
        String message = messageTemplate;
        message = message.replace("{agent.username}", alarm.getAgent().getUsername());

        if (message.contains("{post.content}")) {
            message = message.replace("{post.content}", alarm.getPost().getContent());
        }

        if (message.contains("{comment.text}")) {
            message = message.replace("{comment.text}", alarm.getComment().getText());
        }
        return message;
    }
}
```

AlarmType은 이런 형식으로 구성하고, 유저,특정 이벤트에 맞게 메세지를 replace하는 형식으로 가공해서 보내는 형식이다.

## 예시

그래서 게시물 좋아요 로직을 조면

```java
@Transactional
    public void sendPostLikeAlarm(AlarmType type, Member agent, Member target, PostLike post) {
        if (!type.equals(LIKE_POST)) throw new BusinessException(MISMATCHED_ALARM_TYPE);
        final Alarm alarm = Alarm.builder()
                .type(type)
                .agent(agent)
                .target(target)
                .post(post.getPost())
                .build();

        alarmRepository.save(alarm);
        sendNotification(target.getUsername(), alarm.getType().createAlarmMessage(alarm));
    }
```

Alarm을 저장하고createAlarmMessage()를 통해 메세지 처리하고  sendNotification()메소드를 호출한다.

![image](https://github.com/DongYeopMe/development_note/assets/70151275/0b4d09a2-4398-4bf8-b5e2-3e20443232df)
