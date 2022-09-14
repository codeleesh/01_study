# Auth



## 인증(Authentication)

얼마전 회사를 옮기게 되었습니다. 입사 제출서류를 통해 신원확인을 완료하고 출입증을 받고 사무실안으로 들어가게 되었습니다. 바로 이러한 과정이 인증입니다.

- (식별가능한 정보로) 서비스에 등록된 사용자의 신원을 입증하는 과정



## 인가(Authorization)

옮긴 회사는 건물에서 15F ~ 17F 까지 사용을 하고 있었습니다. 발급받은 출입증은 해당 층만 이용할 수 있었습니다. 그외 층은 갈수가 없는 것입니다. 이것이 바로 권한에 대한 허가를 나타내는 인가입니다.

- 인증된 사용자에 대한 자원 접근 권한 확인



## 인증 인가 프로세스

Request Header 활용하기

Browser 활용하기

- 로컬 스토리지
- 세션 스토리지
- 쿠키
  - 보안에 취약

Session 활용하기

- Session 데이터베이스 활용



### 인증을 유지할 수 있는 방법 

HTTP프로토콜은 다음과 같은 특징을 갖고 있습니다.

- Connectionless 
- Stateless

클라이언트가 서버로 요청을 하고 응답을 받으면 접속을 끊고 상태정보를 저장하지 않는 특징을 가지고 있습니다. 그렇기때문에 쿠키와 세션이 없으면 로그인 상태를 유지하지 않아서 페이지 이동을 하거나 게시판을 작성하는 등의 작업을 할때마다 로그인을 해야하는 번거로움이 발생한다.

- 쿠키
- 세션
- 토큰





## JSON WEB TOKEN





## OAuth

다른 웹사이트 상의 자신들의 정보에 대해 접근 권한을 부여할 수 있는 공통적인 수단 

- 구글 로그인
- 페이스북 로그인
- 카카오 로그인 등
- 이러한 로그인을 인증 절차



### 다양한 인증 방식 제공

- Authorization Code Grant
- Implicit Grant
- Resource Owner Password Credentials Grant
- Client Credentials Grant
- Device Code Grant
- Refresh Token Grant