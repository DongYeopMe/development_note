### IPv4 주소의 기본 구조
<img width="1110" alt="image" src="https://github.com/mo2-Study-Group/StudyGroup/assets/107467750/d8d05cc4-5f34-44f5-881d-fa0e363b9876"><br/>
<br/>
2계층 즉 Ethernet에서의 Mac은 48bit를 사용하고 있지만 이번 시간에 다룰 3계층 IP 중 IPv4는 32bit로 구성되어 있다. 이는 8bit 가 총 4가지로 이루어 져있는데 
하나의 8bit 당 0 부터 255까지 총 256가지로 표현이 가능하다.<br/>
예를 들어 "1111 1111"로 되어 있는 8bit가 있을때 이는 255로 불리며 이전 시간에 배운 broadCast를 나타내는것을 알 수 있게된다. 이와 같이 총 8자리로 이루어진 요소가 4개로 이루어진것이
IPv4를 나타낼때 사용된다는 것으로 알 수 있다.<br/>
<br/>
<img width="1002" alt="image" src="https://github.com/mo2-Study-Group/StudyGroup/assets/107467750/a95e03f1-5538-4308-a673-4e5ca03a4a71"><br/>
<br/>
IPv4는 인터넷 망에서 나의 컴퓨터를 식별하기 위한 IP 주소 이다. 이는 총 4가지의 8bit로 이루어져 있는데
이를 크게 나누어 앞의 3개의 8bit 즉 Netword Id와 남은 하나의 8bit, HostId로 나눌 수 있다 ( 여기서 HostId는 내 컴퓨터를 뜻한다 ).
쉽게 말해서 NetworkId는 우리가 택배를 받기 위해 작성할 시 적는 주소지에서 "XX시 XX구 XX동"에 해당한다. 내가 사는 지역을 대략적으로 특정할 수 있도록 도와주는 역할을 하는 것이다.
또한 뒤에 HostId는 구체적인 나의 번지를 나타내며 구체적인 나의 위치를 특정 할 수 있도록 한다. 이와 같이 NetworkId 오 HostId를 합쳐서 나의 IPv4 주소로 나타낼 수 있다.
