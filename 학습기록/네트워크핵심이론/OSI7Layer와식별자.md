## **OSI 7 layer와 식별자**

우리가 흔히 생각하는 유선 네트워크가 **Ethernet**이다.

HTTP와 SSL을 합치면 **HTTPS**가 된다.

**식별자**란 이름과 같은 것인데, 어떠한 것을 식별하는 정보이다.

![스크린샷 2023-11-19 오후 6 21 08](https://github.com/mo2-Study-Group/StudyGroup/assets/112863029/3e36db2b-c499-4385-9078-14dd7ee8cb1f)

- L2 Frame에서는 **MAC주소(physical address)**
    - 48bit이며 보통 16진수로 표기
    - **NIC**를 식별
- L3 Packet에서는 **IP주소**
    - IPv4에서는 32bit이며 10진수(8bit)씩 끊어 점으로 구분해 표기
    - 인터넷을 사용하는 컴퓨터에 대한 식별자인데, 이것을 **Host**라고 한다.
- L4 수준에서는 **Port번호**
    - 16bit 양의 정수
    - L2에서는 **인터페이스 식별자**, L3, L4에서는 **서비스 식별자**, **프로세스 식별자**라고 한다.

**참고 자료**

- [https://www.inflearn.com/course/네트워크-핵심이론-기초/dashboard](https://www.inflearn.com/course/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC-%ED%95%B5%EC%8B%AC%EC%9D%B4%EB%A1%A0-%EA%B8%B0%EC%B4%88/dashboard)
