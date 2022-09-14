# OSI 7계층

Phisical Layer, Data Link Layer, Network Layer, Transport Layer, Session Layer, Presentation Layer, Application Layer



## Phisical Layer

- 0과1의 나열을 아날로그 신호로 바꾸어 전선으로 흘러 보냄(encoding)
- 아날로그 신호가 들어오면 0과 1의 나열로 해석(decoding)
- 물리적으로 연결된 두 대의 컴퓨터가 0과 1의 나열을 주고받을 수 있게 해주는 모듈(module)



단점

- 여러 대의 컴퓨터가 통신하도록 만들 수 없음



## Data-Link Layer

- 같은 네트워크에 있는 여러 대의 컴퓨터들이 데이터를 주고받기 위해서 필요한 모듈
- Framing 은 Data-link Layer 에 속하는 작업들 중 하나입니다.
- 랜카드



라우터, DNS 공부



## Network Layer

- 수많은 네트워크들의 연결로 이루어지는 inter-network 속에서
- 어딘가에 있는 목적지 컴퓨터로 데이터를 전송하기 위해,
- IP 주소를 이용해서 길을 찾고 (routing)
- 자신 다음의 라우터에게 데이터를 넘겨주는 것(forwarding)
- 운영체제의 커널에 소프트웨어적으로 구현



## Transport Layer

- Port 번호를 사용하여
- 도착지 컴퓨터의 최종 목적지인 프로세스에 까지
- 데이터가 도달하게 하는 모듈
- 운영체제의 커널에 소프트웨어적으로 구현



사실 현대의 인터넷은 OSI 모델이 아니라 TCP/IP 모델을 따르고 있습니다.

TCP/IP 모델

Network Interface, Internet Layer, Transport Layer, Application Layer



## Application Layer







## 비즈니스 로직은 어느 곳에 구현하는 것이 좋을까?

다음과 같은 계층형 아키텍처 기반 하에서 핵심 비즈니스 로직은 어디에 구현하는 것이 맞을까?

![img](https://www.petrikainulainen.net/wp-content/uploads/spring-web-app-architecture.png)

응용 애플리케이션을 개발할 때 TDD, OOP를 적용하려면 핵심 비즈니스 로직을 도메인 객체가 담당하도록 구현하는 것이다.
즉, 테스트하기 쉬운 부분과 테스트하기 어려운 부분을 분리해 테스트하기 쉬운 부분에 대한 단위 테스트를 구현하고 지속적인 리팩터링을 한다.