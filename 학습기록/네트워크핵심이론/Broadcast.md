## Broadcast

- 로컬 LAN 상에 붙어있는(브로드 캐스트 도메인 범위) 모든 네트워크 장비들에게 보내는 통신
    
    → 이 통신은 효율과 관련 없다. 오히려 효율을 떨어 뜨린다.
    
- 꼭 필요할때만 써야하는 통신 방법
- Broadcast 주소라는 매우 특별한 주소가 존재한다. MAC,IP 모두 존재
    - MAC주소는 48bit 주소 체계를 사용하는데, broadcast는 FF-FF-FF-FF-FF-FF 와같이 모두 1이 사용될 경우 broadcast가 된다.
- 이 주소로 패킷을 CPU가 받으면 무조건 읽어 드린다.
    - 자신의 맥 Address와 목적지 Address가 다르면 무시한다.
- 이 패킷을 받은 CPU는 이 패킷을 처리하게 되고 전체 네트워크가 느려지고, PC의 성능도 떨어진다.
- 효율을 떨어뜨리는 이슈가 있기 때문에 범위를 최소화 시키는게 중요하다.

### 사용 예시

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/2286b8d4-06d6-46ef-838a-553dd7a64ad1)

출발지와 목적지가 있는데 목적지 주소가 broadcast가 있는데 누가 받으라는것이냐? → 전부 다 받는것이다.

1번이 Broadcast를 했을때 나머지 PC는 Broadcast가 끝날때 까지 통신을 못한다.

LAN에서 보내는 단위는 Frame 인데,

뒤에는 데이터 앞에는 Header에  출발지,목적지의 주소가 있다.(주소 2개)

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/9384a538-c0fd-4e34-9324-067de6fade7a)

그런데 만약에 이 distinct switch 스위치가 정책상 1번이 Broadcast했을때 통신이 다른쪽으로 가는걸 막는다면, 1,2,3 번 PC만 범위가 포함되고, 나머지 PC는 영향을 받지않는다.

이렇게 막는다면 오히려 네트워크 효율이 좋아 질수도 있다.

그런데 만약 Broadcast가 꼭 필요할때는 반응을 하지 못하는 단점이 있다.

보통 IP네트워크 기반이기 때문에 Broadcast 범위는 IP 주소 상에서 어떤 주소의 레인지가 잡히면 이 정도만 범위로 한다.

## 네트워크 규모

![image](https://github.com/mo2-Study-Group/StudyGroup/assets/70151275/9e3da082-2a16-4711-94b4-36dc606d2144)

출처 : [https://velog.io/@jhyeom1545/네트워크-LAN과-WAN의-경계-그리고-Broadcast](https://velog.io/@jhyeom1545/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-LAN%EA%B3%BC-WAN%EC%9D%98-%EA%B2%BD%EA%B3%84-%EA%B7%B8%EB%A6%AC%EA%B3%A0-Broadcast)

LAN은 물리적 영역, WAN은 논리적 영역에서 구현 한다고 생각하고 공부하면 도움이 된다고 한다.

# 참고

[https://www.inflearn.com/course/네트워크-핵심이론-기초/dashboard](https://www.inflearn.com/course/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%ED%95%B5%EC%8B%AC%EC%9D%B4%EB%A1%A0-%EA%B8%B0%EC%B4%88/dashboard)
