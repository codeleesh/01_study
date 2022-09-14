# [Logback] SpringBoot에서 Logback 사용하기 - 환경설정 파일

로그를 작성하기 위해서 자주 사용하는 로그백의 대한 내용을 정리하고 있습니다.

지난 블로그 내용에서는 로그 사용시 주의점, 로그백 구성과 로그 레벨 상속 그리고 로그 파라미터 처리에 대해서 알아보았습니다.

오늘은 로그백을 사용할 때, `logback-spring.xml` 을 작성하여서 활용을 하는데, 이 환경설정 파일에 대해서 알아보도록 하겠습니다.

- [Logback] 1.SpringBoot에서 Logback 사용하기 - 주의점, 구성, 상속, 파라미터 처리
- *[Logback] 2.SpringBoot에서 Logback 사용하기 - 환경설정 파일*
- [Logback] 3.SpringBoot에서 Logback 사용하기 - Appender와 Policy
- [Logback] 4.SpringBoot에서 Logback 사용하기 - Encoder
- [Logback] 5.SpringBoot에서 Logback 사용하기 - MDC
- [Logback] 6.SpringBoot에서 Logback 사용하기 - Filter
- [Logback] 7.SpringBoot에서 Logback 사용하기 - 웹 어플리케이션 구성



## logback 환경설정 파일

스프링부트에서 로그백을 사용하기 위해서 다음 파일을 로드해서 사용합니다.

- `logback-spring.xml`
- `logback-spring.groovy`
- `logback.xml`
- `logback.groovy`

가능하면 로깅 설정에는 `-spring` 붙어 있는 파일을 사용하는 것을 권장합니다.

- `logback.xml` 파일은 너무 빨리 로드되기 때문에, 이 파일 안에선 Extensions을 사용할 수 없습니다.
- 스프링에서 로그 초기화를 완전히 제어할 수 없습니다.



### 로그백 기본 설정

로그백은 기본적인 설정이 되어 있습니다. Console 및 File 로그 패턴이 설정되어 있으며, 초기 로거 또한 설정되어 있습니다. 그리고 `appender` 별로 각 설정 파일들이 분리되어 있으며, `base.xml` 에서 분리된 설정 파일들 포함하여 제공합니다.

`base.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Base logback configuration provided for compatibility with Spring Boot 1.1
-->

<included>
	<include resource="org/springframework/boot/logging/logback/defaults.xml" />
	<property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
	<include resource="org/springframework/boot/logging/logback/console-appender.xml" />
	<include resource="org/springframework/boot/logging/logback/file-appender.xml" />
	<root level="INFO">
		<appender-ref ref="CONSOLE" />
		<appender-ref ref="FILE" />
	</root>
</included>
```

아래는 기본적으로 설정된 패턴 및 로거 정보 입니다.

`defaults.xml` 

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Default logback configuration provided for import
-->

<included>
	<conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
	<conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
	<conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />

	<property name="CONSOLE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
	<property name="CONSOLE_LOG_CHARSET" value="${CONSOLE_LOG_CHARSET:-${file.encoding:-UTF-8}}"/>
	<property name="FILE_LOG_PATTERN" value="${FILE_LOG_PATTERN:-%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39} : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
	<property name="FILE_LOG_CHARSET" value="${FILE_LOG_CHARSET:-${file.encoding:-UTF-8}}"/>

	<logger name="org.apache.catalina.startup.DigesterFactory" level="ERROR"/>
	<logger name="org.apache.catalina.util.LifecycleBase" level="ERROR"/>
	<logger name="org.apache.coyote.http11.Http11NioProtocol" level="WARN"/>
	<logger name="org.apache.sshd.common.util.SecurityUtils" level="WARN"/>
	<logger name="org.apache.tomcat.util.net.NioSelectorPool" level="WARN"/>
	<logger name="org.eclipse.jetty.util.component.AbstractLifeCycle" level="ERROR"/>
	<logger name="org.hibernate.validator.internal.util.Version" level="WARN"/>
	<logger name="org.springframework.boot.actuate.endpoint.jmx" level="WARN"/>
</included>

```



### 환경 설정 - logback-spring.xml 활용

`ConsoleAppender` 를 활용한 기본 예제는 다음과 같습니다.

```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

- `encoder` 의 기본 설정은 `PatternLayoutEncoder` 입니다.



### 상태 출력

환경설정 파일을 분석하는 동안 경고나 오류가 발생하면 Logback은 콘솔에 상태 메시지를 기록합니다. 해당 로그를 보기 위해서는 다음과 같이 설정하면 됩니다.

```xml
<configuration>
  <statusListener class="ch.qos.logback.core.status.OnConsoleStatusListener" />

  ... the rest of the configuration file
</configuration>
```



### 환경설정파일 위치 지정

시스템 속성을 사용하여 구성 파일의 위치를 지정할 수 있습니다. 

```bash
java jar -Dlogging.config=file:/path/to/config.xml MyApp1.jar
```

> 로컬에서 개발할때는 사용하진 않지만 개발 서버 또는 운영 서버에서 환경 설정 파일을 외부로 관리하면서 어플리케이션을 실행 및 관리할때 자주 사용합니다.



### 자동 로드 스캔 환경설정 파일

구성 파일의 변경 사항을 확인하고 자동으로 재구성됩니다. 기본적으로 구성 파일의 변경 내용은 1분마다 검색합니다.

```xml
<configuration scan="true">
  ...
</configuration>
```

아래 내용을 참고하면 설정을 통해서 시간 변경을 할 수 있습니다.

```xml
<configuration scan="true" scanPeriod="30 seconds" >
  ...
</configuration> 
```

 로그백의 Extensions 기능을 사용한다면 자동 로드 스캔 사용할 수 없습니다.

> Extensions이란?
>
> 스프링 부트의 고급 설정을 제공하며, `<springProfiles>` 태그 및 `<springProperty>` 태그를 활용할 수 있습니다.
>
> 이러한 설정은 `logback-spring.xml`에서만 사용할 수 있습니다.



### 스택트레이스 활성화

로그백은 출력하는 스택 추적 라인에 대한 패키징 데이터를 포함할 수 있습니다. 
- 패키징 데이터는 소프트웨어 버전 관리 문제를 식별하는 데 매우 유용할 수 있습니다. 
- 데이터 패키징은 유용하지만 빈번한 예외가 있는 경우 애플리케이션에서 계산 비용이 많이 듭니다.

```xml
<configuration packagingData="true">
  ...
</configuration>
```



### 파일 작성 참고

- logback 구성 파일의 가장 기본적인 큰 구조는 다음과 같습니다. 

  - `<appender>` 요소 

  - `<logger>` 요소

  - `<root>` 요소


- 규칙과 관련된 태그 이름은 대소문자를 구분하지 않습니다. 
  - `<logger>`와 `<Logger>` , `<LOGGER>` 는 동일한 방식으로 해석됩니다. 


- xml 태그는 작성 규칙을 따라야 합니다.

  - `<xyz>` 로 여는 경우 태그는 `</xyz>` 로 닫아야 합니다.

  - `</XyZ>` 는 작동하지 않습니다.




### 로거 설정 `<logger>`

| 구성       | 필수값 | 값                                         |
| ---------- | ------ | ------------------------------------------ |
| name       | YES    | 이름, 패키지 등                            |
| level      | NO     | TRACE, DEBUG, INFO, WAR, ERROR, ALL or OFF |
| additivity | NO     | true or false                              |

- `<logger>` 요소는 `<appender-ref>` 요소를 포함할 수 있습니다.
  - 이렇게 참조된 각 appender는 해당 로거에 추가됩니다.



### 루트 로거 설정 `<root>`

| 구성  | 필수값 | 값                                         |
| ----- | ------ | ------------------------------------------ |
| level | NO     | TRACE, DEBUG, INFO, WAR, ERROR, ALL or OFF |

루트 로거는 다음과 같은 특징을 갖고 있습니다.

- 이미 `ROOT`로 이름이 지정되어 있으므로 name 속성을 허용하지 않습니다.
- level 속성의 값은 대소문자를 구분하지 않습니다.
- 루트 로거는 상속 또는 NULL로 설정할 수 없습니다.
- `<root>` 요소는 `<appender-ref>` 요소를 포함할 수 있습니다.
  - 이렇게 참조된 각 appender는 루트 로거에 추가됩니다.



아래는 루트 로거와 로거를 이용한 샘플입니다.

```xml
<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>
        %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
     </pattern>
    </encoder>
  </appender>

  <logger name="chapters.configuration" level="INFO" />
  <logger name="chapters.configuration.Foo" level="DEBUG" />

  <root level="DEBUG">
    <appender-ref ref="STDOUT" />
  </root>

</configuration>
```

- `<encoder>` 의 기본 설정은 `ch.qos.logback.classic.encoder.PatternLayoutEncoder` 입니다
- `<root>` 의 log level 설정은 `DEBUG` 입니다.
- 위 샘플 `logback-spring.xml` 의 설정된 로그 레벨을 나타내면 다음과 같습니다.

| 로거 이름                     | 설정된 로그 레벨 | 영향받은 로그 레벨 |
| ----------------------------- | ---------------- | ------------------ |
| root                          | `DEBUG`          | `DEBUG`            |
| chapters.configuration        | `INFO`           | `INFO`             |
| chapters.configuration.MyApp3 | `null`           | `INFO`             |
| chapters.configuration.Foo    | `DEBUG`          | `DEBUG`            |



### 어펜더 설정 `<appender>`

| 구성  | 필수값 | 값                                  |
| ----- | ------ | ----------------------------------- |
| name  | YES    | 이름 지정                           |
| class | YES    | 인스턴스화할 appender 클래스의 이름 |

`<appender>` 요소는 다음을 포함할 수 있습니다.

- `<layout>`
  - 사용자의 요청에 따라 로그 메시지의 포맷을 지정합니다.
  - 레이아웃 클래스가 `PatternLayout`인 경우 기본 클래스 매칭 규칙으로 생략 가능합니다.
- `<encoder>`
  - `Appender`에 포함되어 사용자가 지정한 형식으로 표현될 로그 메세지를 변환하는 역할 담당합니다.
  - 바이트를 소유하고 있는 `appender`가 관리하는 `OutputStream`에 쓸 시간과 내용을 제어할 수 있습니다.
  - `FileAppender` 와 하위 클래스는 `encoder`를 필요로하므로 `layout` 대신 `encoder`를 사용하면 됩니다.
  - 인코더 클래스가 `PatternlayoutEncoder` 인 경우 기본 클래스 매핑 규칙에 지정된 대로 클래스 속성을 생략 가능합니다.
- `<filter>`
  - 해당 패키지에 반드시 로그를 찍지 않고 필터링이 필요한 경우에 사용하는 기능입니다.
  - 예를 들어, 레벨 필터를 추가해서 error 단계인 로그만 찍도록 설정 가능합니다.




#### 어펜더 로깅 중복 

다음과 같이 `appender-ref` 설정하면 로깅이 중복이 됩니다.

```xml
<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
    </encoder>
  </appender>

  <logger name="chapters.configuration">
    <appender-ref ref="STDOUT" />
  </logger>

  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

- `root` 로거와 `chapters.configuration` 로거가 중복됩니다. 이를 위해서는 아래 설정을 통해서 해결할 수 있습니다.



#### 어펜더 로깅 중복 방지

로깅 중복을 방지하기 위해  `additivity` 를 false로 재정의할 수 있습니다.

```xml
<configuration>

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>foo.log</file>
    <encoder>
      <pattern>%date %level [%thread] %logger{10} [%file : %line] %msg%n</pattern>
    </encoder>
  </appender>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>

  <logger name="chapters.configuration.Foo" additivity="false">
    <appender-ref ref="FILE" />
  </logger>

  <root level="debug">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

- `chapters.configuration.Foo` 로거의 `additivity` 가 `false` 로 설정되어 있어서 `chapters.configuration.Foo` 에 해당하는 로그는 `FILE` 로그로만 보냅니다.

  

### 컨텍스트 이름 설정

모든 로거는 로거 컨텍스트에 연결됩니다. 

- 기본값은 `default` 입니다.

- `<contextName>` 구성 요소를 이용하여서 다른 이름을 설정할 수 있습니다. 

- 컨텍스트 이름 설정을 통해서 동일한 대상에 로깅하는 여러 애플리케이션을 간단하게 구별할 수 있습니다.



#### HOSTNAME 프로퍼티

자동 정의되는 속성이며 `HOSTNAME` 으로 사용 가능합니다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<included>
    <contextName>${HOSTNAME}</contextName>
    <!-- appender 이름이 console인 consoleAppender를 선언 -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 해당 로깅의 패턴을 설정 -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>[%contextName] [%d{yyyy.MM.dd HH:mm:ss.SSS}] [%thread] [%X{traceId}] %-5level %logger{5} - %msg %n</pattern>
        </encoder>
    </appender>
</included>
```

출력은 다음과 같습니다.

```
[lovethefeel-MacBookPro.local] [2022.05.18 01:38:06.647] [main] [] INFO  o.s.b.w.e.n.NettyWebServer - Netty started on port 8080 
```



#### CONTEXT_NAME 프로퍼티

`CONTEXT_NAME` 속성은 현재 로깅 컨텍스트의 이름에 해당합니다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<included>
    <contextName>${CONTEXT_NAME}</contextName>
    <!-- appender 이름이 console인 consoleAppender를 선언 -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 해당 로깅의 패턴을 설정 -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>[%contextName] [%d{yyyy.MM.dd HH:mm:ss.SSS}] [%thread] [%X{traceId}] %-5level %logger{5} - %msg %n</pattern>
        </encoder>
    </appender>
</included>
```

출력은 다음과 같습니다. context의 기본값은 `default` 입니다.

```
[default] [2022.05.18 01:40:20.352] [main] [] INFO  o.s.b.w.e.n.NettyWebServer - Netty started on port 8080 
```



### 변수 

#### 변수 정의

변수는 구성 파일 자체에서 정의하여 사용 가능하며, 외부 속성 파일 또는 외부 리소스에서 전체적으로 로드할 수 있습니다. 변수를 정의하기 위한 XML 요소는 `<property>` 이고 logback 1.0.7 이상에서는 `<variable>` 도 사용 가능합니다.



##### XML 설정에서 변수 사용

```xml
<configuration>

  <property name="USER_HOME" value="/home/sebastien" />

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>${USER_HOME}/myApp.log</file>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="FILE" />
  </root>
  
</configuration>
```



##### 외부 리소스에서 사용하는 변수 로드

```bash
java -DUSER_HOME="/home/sebastien" MyApp2
```

```xml
<configuration>

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>${USER_HOME}/myApp.log</file>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="FILE" />
  </root>
  
</configuration>
```



##### 외부 속성파일에서 변수 로드

```properties
USER_HOME=/home/sebastien
```

```xml
<configuration>

  <property resource="resource1.properties" />

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
     <file>${USER_HOME}/myApp.log</file>
     <encoder>
       <pattern>%msg%n</pattern>
     </encoder>
   </appender>

   <root level="debug">
     <appender-ref ref="FILE" />
   </root>
  
</configuration>
```



#### Scopes

scopes의 종류는 다음과 같습니다.

- local
  - scope의 기본값이며, logback 설정 파일을 해석하는 동안만 활성화합니다.
  - 구성 파일 정의 시점부터 해당 구성 파일의 해석/실행이 끝날때까지 존재합니다.
  - 구성 파일이 구문 분석되고 실행될 때마다 로컬 범위의 변수가 새로 정의됩니다.
- context
  - context에서 사용 가능하도록 삽입합니다.
- system
  - JVM의 환경변수 삽입합니다.



`context` 예시는 다음과 같습니다. application.yml 파일에 설정된 `logging.level.root`  프로퍼티를 개별적으로 설정할 수 있습니다.

```xml
<configuration>

  <springProperty scope="context" name="LOG_LEVEL" source="logging.level.root" />

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>/opt/${nodeId}/myApp.log</file>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>

  <root level="${LOG_LEVEL}">
    <appender-ref ref="FILE" />
  </root>
  
</configuration>
```



#### 기본값 변수 설정

변수가 선언되지 않았거나 해당 값이 null인 경우 변수에 기본값을 설정할 수 있습니다.

Bash Shell에서와 같이 `:-` 연산자를 사용하여 기본값을 설정할 수 있습니다.

```xml
${aName:-golden}
```

- aName이라는 변수가 정의되어 있지 않다면 기본값은 golden입니다.



#### 중첩 변수 설정

변수의 값 정의에는 다른 변수에 대한 참조가 포함될 수 있습니다.

##### properties를 활용한 변수 처리

```properties
USER_HOME=/home/sebastien
fileName=myApp.log
destination=${USER_HOME}/${fileName}
```

```xml
<configuration>

  <property file="variables2.properties" />

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>${destination}</file>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>

  <root level="debug">
    <appender-ref ref="FILE" />
  </root>
  
</configuration>
```



#### 타임스탬프 설정

timestamp 요소는 현재 날짜 및 시간에 따라 속성을 정의할 수 있습니다. 

아래와 같이 다양하게 설정하여서 사용할 수 있습니다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<included>
	<timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss" timeReference="contextBirth"/>
	<timestamp key="byDay" datePattern="yyyyMMdd"/>
	<timestamp key="byYear" datePattern="yyyy"/>
	<timestamp key="byMonth" datePattern="MM"/>
	<timestamp key="byDate" datePattern="dd"/>
	<timestamp key="byTime" datePattern="HH"/>
	<timestamp key="byMin" datePattern="mm"/>
	<timestamp key="bySec" datePattern="ss"/>
  
  ...
</included>
```

아래는 `bySecond` 를 이용해서 출력해보는 예제입니다.

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<included>
    <timestamp key="bySecond" datePattern="yyyyMMdd'T'HHmmss" timeReference="contextBirth"/>
    <!-- appender 이름이 console인 consoleAppender를 선언 -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!-- 해당 로깅의 패턴을 설정 -->
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>[${bySecond}] [%d{yyyy.MM.dd HH:mm:ss.SSS}] [%thread] [%X{traceId}] %-5level %logger{5} - %msg %n</pattern>
        </encoder>
    </appender>
</included>
```

출력 예시는 다음과 같습니다.

```
[20220518T015135] [2022.05.18 01:51:39.108] [main] [] INFO  o.s.b.w.e.n.NettyWebServer - Netty started on port 8080 
```



## 정리

- 로그백 환경설정 기본 속성을 사용할 수 있습니다.
- 환경설정 파일 외부에서 로드할 수 있도록 옵션을 제공합니다.
- `Appender` 의 로깅을 중복 방지하는 역할을 합니다.
- 환경설정 파일 안에서 다양한 방법으로 변수를 설정하여서 사용할 수 있습니다.



다음 시간에는 로그백 환경설정 파일에서 중요한 역할을 하는 `Appender` 와 `Policy` 에 대해서 알아보도록 하겠습니다.



## 참고 자료

- [Logback Project](https://logback.qos.ch/index.html)
- [Spring-Boot Logback Docs](https://docs.spring.io/spring-boot/docs/2.1.8.RELEASE/reference/html/howto-logging.html)
