# [Logback] SpringBoot에서 Logback 사용하기 - 주의점, 구성, 상속, 파라미터 처리

로그를 작성하기 위해서 자주 사용하는 로그백의 대한 내용을 정리하려고 합니다.

처음에는 하나의 글로 정리하면서 작성을 하였는데 내용이 커지면서 나중에 찾아보기 쉽게 구성하기 위해서 아래와 같이 목차를 나눠서 정리를 하였습니다.

- *[Logback] 1.SpringBoot에서 Logback 사용하기 - 주의점, 구성, 상속, 파라미터 처리*
- [Logback] 2.SpringBoot에서 Logback 사용하기 - 환경설정 파일
- [Logback] 3.SpringBoot에서 Logback 사용하기 - Appender와 Policy
- [Logback] 4.SpringBoot에서 Logback 사용하기 - Encoder
- [Logback] 5.SpringBoot에서 Logback 사용하기 - MDC
- [Logback] 6.SpringBoot에서 Logback 사용하기 - Filter
- [Logback] 7.SpringBoot에서 Logback 사용하기 - 웹 어플리케이션 구성



해당 내용은 로그를 사용하면 주의할 점에 대해서 알아보고 로그백을 구성하기 위한 방법과 로그 상속, 로그 파라미터 처리에 대해서 알아보도록 하겠습니다.



## 로그 사용시 주의할 점

- **Avoid side effects**
  - logging으로 인해 애플리케이션 기능의 동작에 영향을 미치지 않아야 합니다.
  - 예를 들어 logging하는 시점에 NullPointerException이 발생해 프로그램이 정상적으로 동작하지 않는 상황이 발생하면 안됩니다.
- **Be concise and descriptive**
  - 각 Logging에는 데이터와 설명이 모두 포함되어야 합니다.
- **Log method arguments and return values**
  - 메소드의 input과 output을 로그로 남기면 debugger를 사용해 디버깅하지 않아도 됩니다. 특히 debugger를 사용할 수 없는 상황에서는 상당히 유용하게 사용할 수 있습니다.
  - 이를 구현하려면 메소드 앞 부분과 뒷 부분에 지저분한 중복 코드가 계속해서 발생하는 상황이 발생하는데 이는 AOP를 통해 해결할 수 있습니다.
- **Delete personal information**
  - 로그에 사용자의 전화번호, 계좌번호, 패스워드, 주소, 전화번호와 같은 개인정보를 남기지 않습니다.



## 스프링 부트에서 로그백 구성

log4j 이후에 출시된 Java 기반 Logging Framework 중 하나로 가장 널리 사용되고 있습니다. 

Spring Boot 환경에서 사용하려면 아래 dependency를 추가하면 로그백이 포함되어 있습니다. Spring에서 `Reactive` 와 `Web` 으로 분리되어 있기 때문에 둘 중에 하나를 선택하면 됩니다.

### Reactive

```groovy
implementation 'org.springframework.boot:spring-boot-starter-webflux'
```

### Web

```groovy
implementation 'org.springframework.boot:spring-boot-starter-web'
```



## 로그백 구성 모듈

로그백은 3개의 모듈로 나뉘어집니다.

- logback-core
  - `logback-classic`, `logback-access` 두 모듈의 기반 역할을 담당하며, `appender`, `layout` 인터페이스가 이 모듈에 속해 있습니다.
- logback-classic
  - SLF4J API를 구현하여 log4j 또는 java.util.logging(JUL)과 같은 다른 로깅 프레임워크 간에 쉽게 전환 가능하도록 하며, `Logger` 클래스가 이 모듈에 속해 있습니다.
- logback-access
  - Tomcat 및 Jetty 등 Servlet 컨테이너에 HTTP Access 로그 기능을 제공합니다.



## 상위 로거의 상속

모든 로거가 루트 로거를 상속받을 수 있도록 기본 설정이 되어 있습니다.

- Logger Level : `INFO`



로거 상속에 대해서 케이스를 나눠보면 다음과 같습니다.

- 루트 로거 설정으로 인한 모든 하위 로거 상속
- 루트 로거와 다른 로그 레벨 설정인 상위 로거 상속
- 루트 로거 미설정으로 인한 상위 로거 상속

각 케이스에 대하여 예시를 보면서 이해해보도록 하겠습니다.



 `X`, `X.Y`, `X.Y.Z` 로거는 설정된 로그가 없기 때문에 루트 로거를 상속 받게 됩니다.

| 로거 이름 | 설정된 로그 레벨 | 영향받은 로그 레벨 |
| --------- | ---------------- | ------------------ |
| root      | DEBUG            | DEBUG              |
| X         | none             | DEBUG              |
| X.Y       | none             | DEBUG              |
| X.Y.Z     | none             | DEBUG              |

로거가 설정된 `X`, `X.Y.Z` 를 제외한 `X.Y` 는 상위 로거인 `X` 의 로거를 상속 받게 됩니다.

| 로거 이름 | 설정된 로그 레벨 | 영향받은 로그 레벨 |
| --------- | ---------------- | ------------------ |
| root      | DEBUG            | DEBUG              |
| X         | INFO             | INFO               |
| X.Y       | none             | INFO               |
| X.Y.Z     | ERROR            | ERROR              |

루트 로거가 `OFF` 인 경우 상위 로거의 영향을 게 됩니다.

| 로거 이름 | 설정된 로그 레벨 | 영향받은 로그 레벨 |
| --------- | ---------------- | ------------------ |
| root      | OFF              | OFF                |
| X         | INFO             | INFO               |
| X.Y       | none             | INFO               |
| X.Y.Z     | ERROR            | ERROR              |



## 출력 메소드와 규칙

로거의 레벨은 다음과 같습니다. `TRACE` 로 갈수록 더 많은 양의 로그를 포함하고 있습니다. 로그백에서는 `FATAL` 레벨은 없으며 `ERROR` 에 포함되어 있습니다.

```
TRACE > DEBUG > INFO > WARN > ERROR
```

| Level of Reuqest | TRACE   | DEBUG   | INFO    | WARN    | ERROR   | OFF    |
| ---------------- | ------- | ------- | ------- | ------- | ------- | ------ |
| TRACE            | **YES** | **NO**  | **NO**  | **NO**  | **NO**  | **NO** |
| DEBUG            | **YES** | **YES** | **NO**  | **NO**  | **NO**  | **NO** |
| INFO             | **YES** | **YES** | **YES** | **NO**  | **NO**  | **NO** |
| WARN             | **YES** | **YES** | **YES** | **YES** | **NO**  | **NO** |
| ERROR            | **YES** | **YES** | **YES** | **YES** | **YES** | **NO** |

- 기본 로그 레벨이 `INFO` 이라면 요청할 수 있는 로그 레벨은 다음과 같습니다.
  - `INFO`, `WARN`, `ERROR` 
- 기본 로그 레벨이 `WARN` 이라면 요청할 수 있는 로그 레벨은 다음과 같습니다.
  - `WARN`, `ERROR` 
  



## 로깅에서 파라미터 출력

로그백에서는 input과 output을 로그로 남기기 위해서 추천하는 방식을 가이드하고 있습니다. 그전에 추천하지 않는 방식은 어떤 것인지 확인을 해보겠습니다.



### 추천하지 않는 방식

```java
logger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));
```

위의 로그를 처리하는 순서는 다음과 같습니다.

- 매개변수 생성 비용 발생
  - i와 문자열 변환(`String.valueOf(entry[i])`) 의 대한 문자열 연결 및 출력 구성
- 로깅 레벨을 판단

*매개변수 생성하는데 비용을 발생한 다음 로깅 레벨을 판단하기 때문에 불필요한 리소스가 낭비되게 됩니다.*



### 추천하는 방식

```Java
Object entry = new SomeObject(); 
logger.debug("The entry is {}.", entry);
```

위의 로그를 처리하는 순서는 다음과 같습니다.

- 로깅 레벨을 판단
- 매개변수 생성 비용
  - `object` 출력 처리
- 파라미터를 `{}` 항목의 문자열 값으로 변경

*로깅 레벨을 먼저 판단하기 때문에 매개변수를 생성하는데 비용의 대한 낭비가 생기지 않습니다.*



## 정리

- 스프링 부트에서 로그백을 사용하기 위해서는 Starter 모듈에 `Reactive` 또는 `Web` 을 사용하면 됩니다.
- 모든 로거는 루트 로거를 상속 받으며, 루트 로거가 설정되어 있지 않다면 상위 로거를 상속받는다.
- 로그에서 파라미터를 출력하기 위해서는 추천하는 방식으로 처리해야 합니다.



다음 시간에는 `logback.xml` 환경설정 파일과 관련된 구성에 대해서 알아보도록 하겠습니다.



## 참고 자료

- [Logback Project](https://logback.qos.ch/index.html)
- [Spring-Boot Logback Docs](https://docs.spring.io/spring-boot/docs/2.1.8.RELEASE/reference/html/howto-logging.html)
- [NextStep](https://edu.nextstep.camp/)
