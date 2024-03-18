## 방식

간단하게 말하면 pub,sub 이라는 개념이 있고, 클라이언트→ 서버 가 pub, 서버→ 클라이언트 특정 메시지를 구독한다는 개념으로 sub이다.

## 기능

- 채팅방 입장,나가기
- 메시지 보내기
- 메시지 삭제
- 채팅방 조회 및 메시지 조회

## 구조

1. 메세지 보내는 유저(Client)가 채팅방 생성.
2. 생성되면 메세지 보내서 서버에 메세지 전달
3. Controller의 @MessageMapping에 의해 메세지를 받는다.
4. 따로 @SendTo를 사용하는게 아니라 직접 서비스 로직으로 구독처리

# 구현

## 공통

먼저 라이브러리 추가합니다

```java
implementation 'org.springframework.boot:spring-boot-starter-websocket'
implementation 'org.webjars:webjars-locator-core'
implementation 'org.webjars:sockjs-client:1.0.2'
implementation 'org.webjars:stomp-websocket:2.3.3'
```

WebSockerConfig

```java
@Configuration
@EnableWebSocketMessageBroker // WebSocket 메시지 핸들링 활성화
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/sub"); // 해당 주소를 구독하고 있는 클라이언트들에게 메시지 전달
        registry.setApplicationDestinationPrefixes("/pub"); // 클라이언트에서 보낸 메시지를 받을 prefix 설정
    }

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws-connection") // SockJS 연결 주소
                .setAllowedOriginPatterns("*") // CORS 허용
                .withSockJS(); // 버전 낮은 브라우저에서도 적용 가능
    }

    @Bean
    public SimpMessagingTemplate messagingTemplate(SimpMessagingTemplate messagingTemplate) {
        messagingTemplate.setDefaultDestination("/sub/messages"); //기본적으로 메시지를 전송할 주제를 설정
        return messagingTemplate;
    }
}
```

특징으로는 

`WebSocketMessageBrokerConfigurer` 를 상속하는게 Stomp, WebSocket은 `WebSocketConfigurer` 상속한다.

**`configureMessageBroker()`**: 메시지 브로커를 설정합니다.
**`registerStompEndpoints()`**: Stomp 엔드포인트를 등록합니다.

**`messagingTemplate()` : SimpMessagingTemplate을 이용해 /sub/messages 채널 구독하면 해당 채널로 메시지를 전송할수 있게 설정한다.**

## 채팅

### Entity

**Room**

```java
@Getter
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "rooms")
public class Room {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "room_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @OneToMany(mappedBy = "room")
    private List<RoomMember> roomMembers = new ArrayList<>();

    public Room(Member member) {
        this.member = member;
    }
}

```

RoomMember

```java
@Getter
@Entity
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "room_members")
public class RoomMember {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "room_member_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "room_id")
    private Room room;

    @Builder
    public RoomMember(Member member, Room room) {
        this.member = member;
        this.room = room;
    }
}
```

소속되어있는 유저들

ChatRoom

```java
@Getter
@Entity
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "chat_rooms")
public class ChatRoom {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "chat_room_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "room_id")
    private Room room;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @CreatedDate
    @Column(name = "chat_room_created_date")
    private LocalDateTime createdDate;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "message_id")
    private Message message;

    @Builder
    public ChatRoom(Room room, Member member, Message message) {
        this.room = room;
        this.member = member;
        this.message = message;
    }

    public void updateMessage(Message message) {
        this.message = message;
    }
}

```

Message

```java
@Getter
@Entity
@EntityListeners(AuditingEntityListener.class)
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Inheritance(strategy = InheritanceType.JOINED)
@DiscriminatorColumn
@Table(name = "messages")
public class Message {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "message_id")
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "member_id")
    private Member member;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "room_id")
    private Room room;

    @Lob
    @Column(name = "message_content")
    private String content;

    @CreatedDate
    @Column(name = "message_created_date")
    private LocalDateTime createdDate;

    @Builder
    public Message(String content, Member member, Room room) {
        this.content = content;
        this.member = member;
        this.room = room;
    }

}
```

이렇게 4가지 구성되어있습니다.

### **Controller**

```java
@Validated
@RestController
@RequiredArgsConstructor
public class ChatController {

    private final ChatService chatService;

    @PostMapping("/chat/rooms")//채팅방 만들기 메소드
    public ResponseEntity<ResultResponse> createChatRoom(@RequestParam List<@NotEmpty @Size(max = 12) String> usernames) {
        final ChatRoomCreateResponse response = chatService.createRoom(usernames);

        return ResponseEntity.ok(ResultResponse.of(CREATE_CHAT_ROOM_SUCCESS, response));
    }

    @MessageMapping("/messages")//메시지 보내기
    public void sendMessage(@Valid @RequestBody MessageRequest request) {
        chatService.sendMessage(request);
    }

    @MessageMapping("/messages/delete")// 메시지 삭제
    public void deleteMessage(@Valid @RequestBody MessageSimpleRequest request) {
        chatService.deleteMessage(request.getMessageId(), request.getMemberId());
    }
}

```

채팅방들 가져오기등등도 있습니다.

### Service

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class ChatService {

    private final SecurityUtil securityUtil;
    private final MemberRepository memberRepository;
    private final RoomRepository roomRepository;
    private final RoomMemberRepository roomMemberRepository;
    private final MessageRepository messageRepository;
    private final ChatRoomRepository chatRoomRepository;
    private final SimpMessagingTemplate messagingTemplate;

    @Transactional//방이 있으면 기존 걸 사용, 아니면 새로 만드는 방식
    public ChatRoomCreateResponse createRoom(List<String> usernames) {
        final Member inviter = securityUtil.getLoginMember();
        usernames.add(inviter.getUsername());
        final List<Member> members = memberRepository.findAllByUsernameIn(usernames);

        final Room room;
        final boolean status;
        final Optional<Room> roomOptional = getRoomByMembers(members);
        if (roomOptional.isEmpty()) {
            status = true;
            room = roomRepository.save(new Room(inviter));
            roomMemberRepository.saveAllBatch(room, members);
        } else {
            status = false;
            room = roomOptional.get();
        }

        final List<MemberSimpleInfo> memberSimpleInfos = members.stream()
                .map(MemberSimpleInfo::new)
                .collect(Collectors.toList());

        return new ChatRoomCreateResponse(status, room.getId(), new MemberSimpleInfo(inviter), memberSimpleInfos);
    }

    @Transactional
    public void sendMessage(MessageRequest request) {
        final Member sender = memberRepository.findById(request.getSenderId())
                .orElseThrow(() -> new BusinessException(MEMBER_NOT_FOUND));
        final Room room = roomRepository.findById(request.getRoomId()).orElseThrow(() -> new BusinessException(CHAT_ROOM_NOT_FOUND));
        final List<RoomMember> roomMembers = roomMemberRepository.findAllWithMemberByRoomId(room.getId());
        if (roomMembers.stream().noneMatch(r -> r.getMember().getId().equals(sender.getId())))
            throw new BusinessException(CHAT_ROOM_NOT_FOUND);

        final Message message = messageRepository.save(new Message(request.getContent(), sender, room));
        updateRoom(request.getSenderId(), room, roomMembers, message);

        roomMembers.forEach(r -> messagingTemplate.convertAndSend("/sub/" + r.getMember().getUsername()));
    }
    @Transactional
    public void deleteMessage(Long messageId, Long memberId) {
        final Member member = memberRepository.findById(memberId).orElseThrow(() -> new BusinessException(MEMBER_NOT_FOUND));
        final Message message = messageRepository.findById(messageId).orElseThrow(() -> new BusinessException(MESSAGE_NOT_FOUNT));

        if (!message.getMember().getId().equals(member.getId())) {
            throw new BusinessException(MISS_MATCH);
        }

        final Room room = message.getRoom();
        final List<ChatRoom> joinRooms = chatRoomRepository.findByRoomId(room.getId());
        joinRooms.forEach(joinRoom -> {
            final LocalDateTime createdDateOfMessageToDelete = message.getCreatedDate();
            final LocalDateTime createdDateOfJoinRoom = joinRoom.getCreatedDate();

            if (!createdDateOfMessageToDelete.isBefore(createdDateOfJoinRoom)) {
                if (message.equals(joinRoom.getMessage())) {
                    final LocalDateTime start = createdDateOfJoinRoom.minusSeconds(1L);
                    final LocalDateTime end = createdDateOfMessageToDelete.plusSeconds(1L);
                    final Long total = messageRepository.countByCreatedDateBetweenAndRoom(start, end, room);

                    if (total == 1) {
                        chatRoomRepository.delete(joinRoom);
                    } else {
                        final List<Message> messages = messageRepository.findTop2ByCreatedDateBetweenAndRoomOrderByIdDesc(
                                start, end, room);
                        joinRoom.updateMessage(messages.get(1));
                    }
                }
            }
        });

        messageRepository.delete(message);

        final List<RoomMember> roomMembers = roomMemberRepository.findAllByRoom(room);
        roomMembers.forEach(r -> messagingTemplate.convertAndSend("/sub/" + r.getMember().getUsername()));
    }
}
```

SimpMessagingTemplate 클래스의 convertAndSend("/sub/" + r.getMember().getUsername())는

채팅방에 있는 username들로 sub(구독)하고 여기 구독되어있는 사람들에게 MessageRequest를 Json형식으로 날린다.

## 기능 테스트

stomp는 postman으로 기능 테스트를 할수 없습니다. 

그래서 처음 계획했던건 stomp 기능 테스트 툴인 apic을 이용할려고 했으나 운이 안좋은건지 모르겠지만, 갑자기 사이트가 막혀서 테스트를 못하는 상황이었습니다.

그래서 간단한 HTML와 JS를 만들어서 테스트를 진행 하였습니다.

**websocket.html**

```java
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>WebSocket Example</title>
    <!-- stomp.js 라이브러리 포함 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/stomp.js/2.3.3/stomp.min.js"></script>
    <!-- SockJS 라이브러리 포함 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/sockjs-client/1.5.1/sockjs.min.js"></script>
    <!-- JavaScript 파일 포함 -->
    <script src="websocket.js"></script>
</head>
<body>
<h1>WebSocket Example</h1>
<button onclick="connect()">Connect</button>
<button onclick="sendMessage()">Send Message</button>
<button onclick="deleteMessage()">Delete Message</button>
<input type="text" id="messageInput" placeholder="Enter your message">
<div id="messages"></div>

</body>
</html>
```

**websocket.js**

```java
// 웹소켓 연결을 위한 함수
function connect() {
    var socket = new SockJS('http://localhost:8080/ws-connection');
    var stompClient = Stomp.over(socket);

    stompClient.connect({}, function(frame) {
        console.log('Connected: ' + frame);

        // 서버에서 보낸 메시지를 화면에 표시하기 위한 콜백 함수
        stompClient.subscribe('/sub/messages', function(response) {
            var message = JSON.parse(response.body);
            document.getElementById('messages').innerHTML += '<p>' + message.content + '</p>';
        });
    });
}

// 서버로 메시지를 보내기 위한 함수
function sendMessage() {
    var socket = new SockJS('http://localhost:8080/ws-connection');
    var stompClient = Stomp.over(socket);

    stompClient.connect({}, function(frame) {
        console.log('Connected: ' + frame);

        // 서버로 메시지를 전송 
        stompClient.send('/pub/messages', {}, JSON.stringify({
            roomId: 6,
            senderId: 4,
            content: document.getElementById('messageInput').value
        }));
    });
}
```

테스트 용으로, sender라는 `memberId` 값이 4L인 계정으로 `roomId` 값이 6L으로 생성했다.

http://localhost:8080/websocket.html 로 접속하면 만들어 놓은 HTML 폼이 보이는걸 확인할 수 있다.

Connect 버튼을 누르면 아래 그림처럼 콘솔에 `/sub/messages`을 구독했음을 알 수 있다.

![image](https://github.com/DongYeopMe/development_note/assets/70151275/e7c8dd97-f7b8-44dd-8491-eeafeb11af63)

text에 메시지를 입력하여 Send Message 버튼을 누르면 아래와 같이 결과가 나온다.

![image](https://github.com/DongYeopMe/development_note/assets/70151275/f5ee88da-7cc2-46ab-b10d-55a093d42282)

그래서 메세지 조회를 하면

![image](https://github.com/DongYeopMe/development_note/assets/70151275/2dac78ab-d111-438b-9848-d88f5b1d7749)

잘나오는것을 확인할수 있다.
