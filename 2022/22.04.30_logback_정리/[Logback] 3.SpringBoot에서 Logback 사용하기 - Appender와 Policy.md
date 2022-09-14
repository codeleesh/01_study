# [Logback] 3.SpringBoot에서 Logback 사용하기 - Appender와 Policy

로그를 작성하기 위해서 자주 사용하는 로그백의 대한 내용을 정리하고 있습니다.

지난 블로그 내용에서는  `logback-spring.xml`  환경설정 파일에 대해서 알아보았습니다.

오늘은 로그백 환경설정 파일에서 중요한 역할을 하는 `Appender` 와 `Policy` 에 대해서 알아보도록 하겠습니다.

- [Logback] 1.SpringBoot에서 Logback 사용하기 - 주의점, 구성, 상속, 파라미터 처리
- [Logback] 2.SpringBoot에서 Logback 사용하기 - 환경설정 파일
- *[Logback] 3.SpringBoot에서 Logback 사용하기 - Appender와 Policy*
- [Logback] 4.SpringBoot에서 Logback 사용하기 - Encoder
- [Logback] 5.SpringBoot에서 Logback 사용하기 - MDC
- [Logback] 6.SpringBoot에서 Logback 사용하기 - Filter
- [Logback] 7.SpringBoot에서 Logback 사용하기 - 웹 어플리케이션 구성



## Appender

로그의 형태를 설정하며, 로그 메세지가 콘솔에 출력할지, 파일에 출력할지 등의 설정을 통해 출력될 대상을 결정하는 요소입니다.

`appender` 는 자주 사용하는 아래 목록에 대해서 정리하도록 하겠습니다.

- `ConsoleAppender` 는 로그를 `OutputStream` 에 작성하여 콘솔에 출력합니다.
- `FileAppender` 는 파일에 로그를 출력하며, 최대 보관 일수, 파일 용량  등을 지정할 수 있습니다.
  - `RollingFileAppender` 는 `FileAppender` 를 상속 받으며, 여러 개의 파일을 롤링, 순회하면서 로그를 파일에 출력합니다.
    - 지정 용량이 넘어간 로그 파일을 넘버링하여서 나눠서 저장 가능합니다.



이외에도 다양한 `Appender` 가 존재합니다. 

- SocketAppender and SSLSocketAppender
- ServerSocketAppender and SSLServerSocketAppender
- SMTPAppender
- DBAppender
- SyslogAppender
- SiftingAppender
- AsyncAppender



### ConsoleAppender

콘솔에 로그를 추가할 수 있습니다. 간단한 예제는 다음과 같습니다.

```xml
<configuration>

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg %n</pattern>
    </encoder>
  </appender>

  <root level="DEBUG">
    <appender-ref ref="STDOUT" />
  </root>
  
</configuration>
```

잠시 패턴에 대해서 알아보고 가겠습니다. 해당 내용은 외우는 것보다는 잘 정리해놓아서 적절하게 사용하면 될것 같습니다.



#### Pattern

로그 출력하기 위해서 어떠한 포맷을 정의할 수 있습니다. 패턴에 사용되는 요소는 다음과 같습니다.

- %Logger{length} - Logger name을 축약할 수 있다. {length}는 최대 자리 수

- %thread - 현재 Thread 이름

- %-5level - 로그 레벨, -5는 출력의 고정폭 값

- %msg - 로그 메시지 (=%message)

- %n - new line

- ${PID:-} - 프로세스 아이디

- %d : 로그 기록시간
- %p : 로깅 레벨
- %F : 로깅이 발생한 프로그램 파일명
- %M : 로깅이 발생한 메소드의 이름
- %l : 로깅이 발생한 호출지의 정보
- %L : 로깅이 발생한 호출지의 라인 수
- %t : 쓰레드 명
- %c : 로깅이 발생한 카테고리
- %C : 로깅이 발생한 클래스 명
- %m : 로그 메시지
- %r : 애플리케이션 시작 이후부터 로깅이 발생한 시점까지의 시간



### FileAppender

로그 이벤트를 파일에 추가합니다. 

- 로그 파일에 대한 옵션을 지정할 수 있으며, 파일을 분리할 수 있습니다. 
- 기본적으로 각 로그 이벤트는 기본 출력 스트림으로 즉시 플러시됩니다. 이 기본 접근 방식은 어펜더를 제대로 닫지 않고 애플리케이션이 종료되는 경우 로깅 이벤트가 손실되지 않는다는 점에서 안전합니다. 그러나 로깅 처리량을 크게 늘리기 위해 `immediateFlush` 속성을 `false`로 설정할 수 있습니다.

```xml
<configuration>

  <appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>testFile.log</file>
    <append>true</append>
    <!-- set immediateFlush to false for much higher logging throughput -->
    <immediateFlush>true</immediateFlush>
    <!-- encoders are assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender>
        
  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
</configuration>
```

파일 이름을 설정할때 날짜와 시간을 조합하여서 설정하는데, 로그백에서는 타임스탬프를 설정하는 변수를 제공하고 있습니다.

다음 링크에서 `타임스탬프 설정` 을 참고해주세요.





### RollingFileAppender

로그 파일을 롤오버하는 기능으로 FileAppender를 확장하여서 사용합니다. `RollingFileAppender` 를 사용하기 위해서는 중요한 하위 구성 요소는 `RollingPolicy` 와 `TriggeringPolicy` 입니다.

- RollingPolicy
  - 롤오버에 필요한 작업을 수행합니다.
  - 무엇을 할지 담당합니다.
- TriggeringPolicy
  - 롤오버가 발생하는지 여부와 정확한 시기를 결정합니다.
  - 언제할지 담당합니다.

`RollingFileAppender`를 사용하기 위해서는 `RollingPolicy`와 `TriggeringPolicy`가 모두 설정되어 있어야 합니다.

아래는 `RollingFileAppender` 를 사용하여 적용한 예시입니다.

```xml
<configuration>

  <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
  <!--로깅이 기록될 위치-->
		<file>${LOG_HOME}/${LOG_FILE}</file>
		<!--로깅 파일이 특정 조건을 넘어가면 다른 파일로 만들어 준다.-->
		<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
			<fileNamePattern>${LOG_HOME}/rolling/logFile.%d{yyyy-MM-dd}.%i.gz</fileNamePattern>
			<maxFileSize>50MB</maxFileSize>
			<maxHistory>7</maxHistory>
			<totalSizeCap>2GB</totalSizeCap>
		</rollingPolicy>
		<encoder>
			<charset>utf8</charset>
				<Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %thread [%X{traceId}] %-5level %logger - %m%n</Pattern>
		</encoder>
	</appender>
        
  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
  
</configuration>
```

- `appender` 는 `RollingFileAppender` 를 사용하였고 `rollingPolicy` 는 `SizeAndTimeBasedRollingPolicy` 를 사용하였습니다.
- `fileNamePattern` 은 `yyyy-mm-dd` 를 설정하여 일자별로 `gz` 파일 형태로 롤링됩니다.
- `maxFileSize` 를 50MB로 설정하여서 로그 파일이 50MB를 넘어가면 `%i` 숫자 증가되면서 롤링됩니다.
- `maxHistory` 를 7로 설정하여서 로그 파일을 현재 날짜 기준으로 7일까지 보관하며 그 이후 로그 파일은 오래된 파일부터 삭제합니다.
- `totalSizeCap` 을 2GB로 설정하여서 로그 파일 전체가 2GB를 넘어가면 오래된 로그 파일부터 삭제합니다. `totalSizeCap` 을 사용하기 위해서는 `maxHistory` 가 설정이 되어 있어야 합니다. 



### RollingPolicy

보관 정책에 대해서 `TimeBasedRollingPolicy`, `SizeAndTimeBasedRollingPolicy`,  `FixedWindowRollingPolicy` 등이 있습니다.



#### TimeBasedRollingPolicy

가장 많이 사용하는 롤링 정책으로 일별 또는 월별과 같이 시간을 기준으로 롤오버 정책을 정의합니다. 롤오버와 해당 롤오버의 트리거에 대한 책임을 담당하며 RollingPolicy와 TriggeringPolicy 인터페이스를 모두 구현합니다. 즉, 무엇을 할지와 언제 할지를 모두 담당하고 있습니다.

| 구성                | 필수값 | 값                                                           |
| ------------------- | ------ | ------------------------------------------------------------ |
| fileNamePattern     | YES    | 파일의 이름을 정의합니다.<br />해당 값은 파일 이름과 `%d` 변환지정자로 구성됩니다.<br /><br />`SimpleDateFormat` 클래스에서 지정한 날짜 및 시간패턴을 포함합니다.<br />날짜 및 시간 패턴이 생략되면 기본 패턴 `yyyy-MM-dd` 사용됩니다.<br />롤오버 시간은 fileNamePattern의 값에서 유추됩니다.<br /><br />*Multiple %d specifiers*<br />`/var/log/%d{yyyy/MM, aux}/application.%d{yyyy-MM-dd}.log` <br /><br />*TimeZone*<br />`folder/test.%d{yyyy-MM-dd-HH, UTC}.log` |
| maxHistory          | NO     | 보관할 파일의 최대수를 제어하여 오래된 파일을 비동기식으로 삭제합니다.<br />일간 롤오버가 지정되어 있고 maxHistory를 6으로 설정하면 6일 분량의 로그 파일은 유지가 되고 6일 이전의 로그 파일은 삭제가 됩니다. |
| totalSizeCap        | NO     | 모든 로그 파일의 전체 크기를 제어합니다.<br />전체 크기 제한을 초과하면 가장 오래된 파일을 비동기식으로 삭제합니다.<br />해당 속성을 사용하기 위해서는 `maxHistory`가 설정되어 있어야 합니다. |
| cleanHistoryOnStart | NO     | 기본값은 `false` 입니다.<br />`ture`이면 appender 시작시 로그 파일 제거가 실행이 됩니다.<br />로그 파일 제거는 일반적으로 롤어부 중에 수행이 되는데, 롤오버가 트리거되지 않을 만큼 오래 지속되지 않을 수 있기 때문에 해당 옵션을 설정하여 로그 파일 제거 가능합니다. |

`fileNamePattern` 에 대해서 몇가지 알아보도록 하겠습니다.

| fileNamePattern                            | Rollover schedule                                     | Example                                                      |
| ------------------------------------------ | ----------------------------------------------------- | ------------------------------------------------------------ |
| */wombat/foo.%d*                           | 일일 롤오버                                           | 파일 속성이 설정되지 않음<br />2006년 11월 23일 동안 로깅 출력은 /wombat/foo.2006-11-23 파일로 이동하며 자정이 되면 /wombat/foo.2006-11-24로 이동합니다.<br />파일 속성이 /wombat/foo.txt로 설정됨<br />2006년 11월 23일 동안 로깅 출력은  /wombat/foo.txt 파일로 생성되며 자정이 되면 /wombat/foo.2006-11-23 으로 이름이 변경됩니다. 그리고 새로운 /wombat/foo.txt 파일이 생성되고 2006년 11월 24일 로깅 출력은 foo.txt로 생성됩니다. |
| */wombat/%d{yyyy/MM}/foo.txt*              | 매월 롤오버                                           | 롤오버는 매월 발생합니다.                                    |
| */wombat/foo.%d{yyyy-ww}.log*              | 매주 첫째날 롤오버<br />첫째날은 locale에 따라 상이함 | 롤오버는 매주 발생합니다.                                    |
| */wombat/foo%d{yyyy-MM-dd_HH}.log*         | 매시간 롤오버                                         | 롤오버는 매시간 발생합니다.                                  |
| */wombat/foo%d{yyyy-MM-dd_HH-mm}.log*      | 매분 롤오버                                           | 롤오버는 매분 발생합니다.                                    |
| */wombat/foo%d{yyyy-MM-dd_HH-mm, UTC}.log* | 매분 롤오버                                           | 파일 이름이 UTC로 표시됩니다.                                |
| */foo/%d{yyyy-MM,**aux**}/%d.log*          | 매일 롤오버<br />연도와 월이 포함된 폴더 아래에 생성  | 롤오버는 매일 발생하며(기본값은 %d) 폴더 이름은 연도와 월에 따라서 달라집니다. |

TimeBasedRollingPolicy는 자동 파일 압축을 지원합니다. 이 기능은 fileNamePattern 옵션의 값이 .gz 또는 .zip으로 끝나는 경우 실행됩니다.

| fileNamePattern     | Rollover schedule                               | Example                                                      |
| ------------------- | ----------------------------------------------- | ------------------------------------------------------------ |
| */wombat/foo.%d.gz* | 로그 파일의 자동 GZIP 압축 및 일일 롤오버(자정) | 파일 속성이 설정되지 않음<br />2006년 11월 23일 동안 로깅 출력은 /wombat/foo.2006-11-23 파일로 이동합니다. 자정에 해당 파일은 압축되어 /wombat/foo.2009-11-23.gz가 됩니다.<br />파일 속성이 /wombat/foo.txt로 설정됨<br />2006년 11월 23일 동안 로깅 출력은  /wombat/foo.txt 파일로 생성되며 자정이 되면 /wombat/foo.2006-11-23.gz 으로 압축됩니다. 그리고 새로운 /wombat/foo.txt 파일이 생성되고 2006년 11월 24일 로깅 출력은 foo.txt로 생성되며 자정이 되면 압축합니다. |

다음은 예시입니다.

```xml
<configuration>
  
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>logFile.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- daily rollover -->
      <fileNamePattern>logFile.%d{yyyy-MM-dd}.log</fileNamePattern>
      <!-- keep 30 days' worth of history capped at 3GB total size -->
      <maxHistory>30</maxHistory>
      <totalSizeCap>3GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender> 

  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
  
</configuration>
```



#### SizeAndTimeBasedRollingPolicy

날짜별로 파일을 보관하면서 각 로그 파일의 크기를 제한할 수 있습니다. 해당 정책도 무엇을 할지와 언제 할지를 모두 담당하고 있습니다.

```xml
<configuration>
  
  <appender name="ROLLING" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>mylog.txt</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
      <!-- rollover daily -->
      <fileNamePattern>mylog-%d{yyyy-MM-dd}.%i.txt</fileNamePattern>
       <!-- each file should be at most 100MB, keep 60 days worth of history, but at most 20GB -->
       <maxFileSize>100MB</maxFileSize>    
       <maxHistory>60</maxHistory>
       <totalSizeCap>20GB</totalSizeCap>
    </rollingPolicy>
    <encoder>
      <pattern>%msg%n</pattern>
    </encoder>
  </appender>

  <root level="DEBUG">
    <appender-ref ref="ROLLING" />
  </root>

</configuration>
```

- `fileNamePattern` 요소의 `%d`, `%i` 두 값은 모두 필수값입니다.
- `%i` 는 현재 기간이 끝나기 전에 현재 로그 파일이 `maxFileSize` 에 도달할 때마다 0에서 시작하여서 증가하는 인덱스입니다.
- 애플리케이션이 중지되었다가 다시 시작되면 현재 로그파일 위치에서 가장 큰 인덱스 번호에서 로깅을 계속 합니다.
- `SizeAndTimeBaseFNATP` 보다 `SizeAndTimeBasedRollingPolicy` 가 더 간단한 구성을 제공합니다.
- `TimeBasedRollingPolicy` 와 차이점은 `maxFileSize` 옵션을 사용할 수 있다는 점입니다.



#### FixedWindowRollingPolicy

롤오버할 때 알고리즘에 따라 파일 이름을 변경합니다. 해당 정책은 무엇을 할지만 담당합니다. 그래서 `triggeringPolicy` 가 필요합니다.

| 구성            | 필수값 | 값                                                           |
| --------------- | ------ | ------------------------------------------------------------ |
| minIndex        | NO     | 시작값                                                       |
| maxIndex        | NO     | 종료값                                                       |
| fileNamePattern | YES    | 아카이브된(롤오버된) 로그 파일의 파일 이름 패턴을 나타냅니다. 이 옵션은 필수이며 패턴 내 어딘가에 정수 토큰 %i를 포함해야 합니다.<br />파일 압축도 이 속성을 통해 지정됩니다. |

예제는 다음과 같습니다.

```xml
<configuration>
  <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>test.log</file>

    <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
      <fileNamePattern>tests.%i.log.zip</fileNamePattern>
      <minIndex>1</minIndex>
      <maxIndex>3</maxIndex>
    </rollingPolicy>

    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
      <maxFileSize>5MB</maxFileSize>
    </triggeringPolicy>
    <encoder>
      <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
  </appender>
        
  <root level="DEBUG">
    <appender-ref ref="FILE" />
  </root>
  
</configuration>
```

- `rollingPolicy` 을 이용해서 `index` 번호를 1부터 3까지 증가하여서 로그 파일을 롤링합니다.
- `triggeringPolicy` 을 이용해서 로그 파일이 5MB가 되면 로그 파일을 롤링합니다.



### TriggeringPolicy

#### SizeBasedTriggeringPolicy

현재 로그 파일의 크기를 확인하며, 지정된 크기보다 커지면 appender에 신호를 보내 기존 로그 파일의 롤오버를 트리거 합니다.

옵션으로는 `maxFileSize` 매개변수 하나만 허용합니다.

- 기본값은 `10MB` 입니다.
- 숫자 값에 각각 KB, MB, GB를 추가할 수 있습니다.



### 제약 사항

로그 롤오버는 시간 기반이이 아니라 로깅 이벤트의 따라 생성됩니다. 일별 롤오버에서 하루가 지났음에도 로그 이벤트가 생성되지 않았다면 롤오버는 작동하지 않습니다.



## 정리

- Appender는 로그의 형태를 정의하며, 다양한 Appender가 존재합니다.
- `ConsoleAppender` 는 콘솔에 로그를 출력할 수 있으며, 로그 출력하기 위해서 어떠한 포맷을 정의할 수 있습니다.
- `RollingFileAppender` 는 파일에 로그를 출력하며, 전체 및 단일 파일 사이즈와 파일 보유 기간을 정의할 수 있습니다.
- 로그 롤링은 시간 기반이 아니라 해당 일의 로그 이벤트가 발생하여야 롤링 합니다.



다음 시간에는 로그백 환경설정 파일에서 중요한 역할을 하는 `Encoder` 에 대해서 알아보도록 하겠습니다.



## 참고 자료

- [Logback Project](https://logback.qos.ch/index.html)
- [Spring-Boot Logback Docs](https://docs.spring.io/spring-boot/docs/2.1.8.RELEASE/reference/html/howto-logging.html)
