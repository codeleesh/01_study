# [Logback] 4.SpringBoot에서 Logback 사용하기 - Encoder

로그를 작성하기 위해서 자주 사용하는 로그백의 대한 내용을 정리하고 있습니다.

지난 블로그 내용에서는  `Appender` 와 `Policy` 에 대해서 알아보았습니다.

오늘은 로그백 환경설정 파일에서 중요한 역할을 하는 `Encoder` 에 대해서 알아보도록 하겠습니다.

- [Logback] 1.SpringBoot에서 Logback 사용하기 - 주의점, 구성, 상속, 파라미터 처리
- [Logback] 2.SpringBoot에서 Logback 사용하기 - 환경설정 파일
- [Logback] 3.SpringBoot에서 Logback 사용하기 - Appender와 Policy
- *[Logback] 4.SpringBoot에서 Logback 사용하기 - Encoder*
- [Logback] 5.SpringBoot에서 Logback 사용하기 - MDC
- [Logback] 6.SpringBoot에서 Logback 사용하기 - Filter
- [Logback] 7.SpringBoot에서 Logback 사용하기 - 웹 어플리케이션 구성



## Encoder

인코더의 특징으로는 다음과 같습니다.

- 인코더는 이벤트를 바이트 배열로 변환하고 해당 바이트 배열을 OutputStream에 쓰는 역할을 합니다.
- Appender에 포함되어 사용자가 지정한 형식으로 표현될 로그 메세지를 변환하는 역할을 담당합니다.
- 인코더는 logback 버전 0.9.19에서 도입되었습니다. 이전 버전에서 대부분의 어펜더는 이벤트를 문자열로 변환하고 java.io.Writer를 사용하여 작성하기 위해 레이아웃에 의존했습니다. 
  - 이전 버전의 logback에서 사용자는 FileAppender 내에 PatternLayout을 중첩했습니다. 
- logback 0.9.19부터 FileAppender 및 하위 클래스는 인코더를 예상하고 더 이상 레이아웃을 사용하지 않습니다.



### LayoutWrappingEncoder

인코더 인터페이스를 구현하며, Layout를 내부에 가져 이벤트를 문자열로 변환하는 작업을 Layout에게 위임합니다.



### PatternLayoutEncoder

`PatternLayoutEncoder` 는 default 설정이라서 생략 가능합니다. 간단한 예시는 다음과 같습니다.

`pattern` 정보를 출력하기 위해서 `outputPatternAsHeader` 정보를 true로 설정하면 로그 출력 시 같이 나오게 됩니다.

```xml
<appender name="FILE" class="ch.qos.logback.core.FileAppender"> 
  <file>foo.log</file>
  <encoder>
    <pattern>%d %-5level [%thread] %logger{0}: %msg%n</pattern>
    <outputPatternAsHeader>true</outputPatternAsHeader>
  </encoder> 
</appender>
```



로그는 어떤 상황에서 잘 동작하는지 그렇지 않다면 어떤 에러가 발생했는지 파악하고 대처하기 위해서 기록을 합니다.

예를 들어서, 지하철 경로 찾기와 같은 이벤트 로그는 데이터 파악이 좀 더 쉽도록 `json` 로거로 수집하는 것도 좋은 방법입니다. 그래서 관련 `encoder` 에 대해서 알아보도록 하겠습니다.



### LogstashEncoder

`LogstashEncoder` 는 로그를 `json` 으로 변경해서 해당 로그를 `Elasticsearch` 라고 하는 통합 로그 모니터링 시스템으로 전송을 하는 역할을 합니다. 

여기서는 해당 Encoder를 이용해서 로그를 `json` 으로 수집을 하는 방법에 대해서 알아보도록 하겠습니다. 그래서 아래 라이브러리를 dependency에 추가해줘야 합니다.

```groovy
implementation 'net.logstash.logback:logstash-logback-encoder:6.1'
```



#### json의 표준 필드

| 필드          | 설명                                                         |
| ------------- | ------------------------------------------------------------ |
| `@timestamp`  | 로그 이벤트가 발생한 시간을 기록하며, 기본 TimestampPattern은  `yyyy-MM-dd'T'HH:mm:ss.SSSZZ` 이며 기본 TimeZone은 기본 시스템 TimeZone  입니다.<br />TimestampPattern과 TimeZone을 설정할 수 있습니다.<br />`<timestampPattern>` : `[UNIX_TIMESTAMP_AS_NUMBER]`, `[UNIX_TIMESTAMP_AS_STRING]` 등 설정 가능합니다.<br />`<timeZone>` : `UTC`, `GMT+10` 등 설정 가능합니다. |
| `@version`    | Logstash 포맷 버전을 설정 가능하며 기본 설정은 `문자열값인 1` 입니다.<br />숫자로도 설정가능하며 다음과 같이 설정하면 됩니다.<br />`<writerVersionAsInteger>true</writerVersionAsInteger>` |
| `message`     | 설정한 로그 메시지를 보여줍니다.                             |
| `logger_name` | 설정된 로그 이름을 보여줍니다.                               |
| `thread_name` | 실행된 쓰레드의 이름을 보여줍니다.                           |
| `level`       | 로그 레벨을 보여줍니다.                                      |
| `level_value` | 로그 레벨의 숫자값을 보여줍니다. ex) INFO - 20000, ERROR - 40000 |
| `stack_trace` | 해당 throwable 객체와 표준 오류 스트림에서 해당 객체의 스택 트레이스를 출력합니다. |
| `tags`        |                                                              |



#### 컨텍스트 필드

컨텍스트 필드는 로그백 버전 1.1.10 포함한 이전 버전은 `HOSTNAME` 프로퍼티는 기본으로 설정되어 있었습니다. 그러나 로그백 1.1.10 버전에서 `HOSTNAME` 프로퍼티의 성능 이슈로 기본으로 설정되지 않았습니다. 



#### 호출 정보 필드

encoder/layout/appender는 기본적으로 발신자 정보를 포함하지 않습니다. 해당 옵션을 이용하여 아래와 같은 정보들을 얻을 수 있습니다. `단 이것은 계산 비용이 들기 때문에 운영 환경에서는 사용하지 않아야 합니다.` 

사용방법은 다음과 같이 설정하면 됩니다.

```xml
<encoder class="net.logstash.logback.encoder.LogstashEncoder">
    <includeCallerData>true</includeCallerData>
</encoder>
```

| 필드                 | 설명                           |
| -------------------- | ------------------------------ |
| `caller_class_name`  | 이벤트를 기록한 클래스의 이름  |
| `caller_method_name` | 이벤트를 기록한 메소드의 이름  |
| `caller_file_name`   | 이벤트를 기록한 파일의 이름    |
| `caller_line_number` | 이벤트를 기록한 파일의 줄 번호 |



#### 커스텀 필드

위의 필드 외에도 로깅 이벤트는 전역적으로 설정 또는 이벤트별로 추가할 수 있습니다.

##### 전역 설정

```xml
<encoder class="net.logstash.logback.encoder.LogstashEncoder">
    <customFields>{"appname":"myWebservice","roles":["customerorder","auth"],"buildinfo":{"version":"Version 0.1.0-SNAPSHOT","lastcommit":"75473700d5befa953c45f630c6d9105413c16fe1"}}</customFields>
</encoder>
```

##### 이벤트별 설정

메시지를 로깅할 때 다음을 사용하여 JSON 출력에 필드를 추가할 수 있습니다.

- `StructureArguments` 는 로그 이벤트의 대한 정형화된 메세지를 출력하며, json 형식의 로그도 출력합니다.
- `Markers` 는 오직 json 형식의 로그만 출력합니다.



`StructureArguments` 를 이용하여 다음과 같은 로그를 출력해보겠습니다.

```java
jsonLogger.info("log message {}", value("name", "value"));
```

`console` 결과는 다음과 같습니다.

```bash
[isanghoui-MacBookPro.local] - [2022.05.21 00:37:34.409] - [INFO ] - [main] - [] - [json] - log message value
```

`json` 로그는 다음과 같습니다.

```json
{
  "@timestamp" : "2022-05-21 00:37:34.409",
  "@version" : "1",
  "message" : "log message value",
  "logger_name" : "json",
  "thread_name" : "main",
  "level" : "INFO",
  "level_value" : 20000,
  "HOSTNAME" : "isanghoui-MacBookPro.local",
  "name" : "value",
  "caller_class_name" : "com.lovethefeel.webflux.SpringBoot2WebfluxApplication",
  "caller_method_name" : "run",
  "caller_file_name" : "SpringBoot2WebfluxApplication.java",
  "caller_line_number" : 40
}
```



`Markers` 를 이용하여 다음과 같은 로그를 출력해보겠습니다.

```java
jsonLogger.info(append("name", "value"), "log message");
```

`console` 의 경우는 다음과 같습니다.

```
[isanghoui-MacBookPro.local] - [2022.05.21 00:37:34.487] - [INFO ] - [main] - [] - [json] - log message
```

`json` 로그는 다음과 같습니다.

```json
{
  "@timestamp" : "2022-05-21 00:37:34.487",
  "@version" : "1",
  "message" : "log message",
  "logger_name" : "json",
  "thread_name" : "main",
  "level" : "INFO",
  "level_value" : 20000,
  "HOSTNAME" : "isanghoui-MacBookPro.local",
  "name" : "value",
  "caller_class_name" : "com.lovethefeel.webflux.SpringBoot2WebfluxApplication",
  "caller_method_name" : "run",
  "caller_file_name" : "SpringBoot2WebfluxApplication.java",
  "caller_line_number" : 56
}
```



#### pretty print

json 로그 출력 시 `pretty` 하게 보기 위해서는 다음 내용을 참고하여서 이쁘게 나올 수 있도록 설정할 수 있습니다.

```xml
<encoder class="net.logstash.logback.encoder.LogstashEncoder">
	<jsonGeneratorDecorator class="net.logstash.logback.decorate.CompositeJsonGeneratorDecorator">
		<decorator class="net.logstash.logback.decorate.PrettyPrintingJsonGeneratorDecorator"/>
	</jsonGeneratorDecorator>
</encoder>
```

지금까지 `json` 로그를 처리하기 위해서는 알아보았습니다. `json` 로그백 환경설정 파일을 살펴보겠습니다.



#### json-appender 설정 파일

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<included>
    <property name="LOG_HOME" value="./logs" />
    <property name="JSON_LOG_FILE" value="json-logFile.log" />
    <appender name="json" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_HOME}/${JSON_LOG_FILE}</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/rolling/json-logFile.%d{yyyy-MM-dd}.%i.gz</fileNamePattern>
            <maxFileSize>50MB</maxFileSize>
            <maxHistory>7</maxHistory>
            <totalSizeCap>2GB</totalSizeCap>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeContext>true</includeContext>
            <includeCallerData>true</includeCallerData>
            <!-- 로그를 이쁘게 출력하기 위해서 -->
            <jsonGeneratorDecorator class="net.logstash.logback.decorate.CompositeJsonGeneratorDecorator">
                <decorator class="net.logstash.logback.decorate.PrettyPrintingJsonGeneratorDecorator"/>
            </jsonGeneratorDecorator>
            <timestampPattern>yyyy-MM-dd HH:mm:ss.SSS</timestampPattern>
        </encoder>
    </appender>
</included>
```



## 정리

- 인코더는 이벤트를 바이트 배열로 변환하고 해당 바이트 배열을 OutputStream에 쓰는 역할을 합니다.
- 인코더의 기본 설정은 `PatternLayoutEncoder` 로 기본적으로 생략이 가능합니다.
- `LogstashEncoder` 를 이용하여 json 로그를 남기기에 적합한 이벤트 로그에 대해서 설정이 가능합니다.



다음 시간에는 로그백의 주요 설계 목표 중 하나인 복잡한 분산 응용 프로그램을 추적 및 디버그하기 위한 방법을 해결하기 위한  `MDC` 에 대해서 알아보도록 하겠습니다.



## 참고 자료

- [Logback Project](https://logback.qos.ch/index.html)

