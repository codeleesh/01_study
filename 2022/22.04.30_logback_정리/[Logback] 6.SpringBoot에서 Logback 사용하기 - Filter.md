# [Logback] 6.SpringBoot에서 Logback 사용하기 - Filter

로그를 작성하기 위해서 자주 사용하는 로그백의 대한 내용을 정리하고 있습니다.

지난 블로그 내용에서는  `MDC` 에 대해서 알아보았습니다. 

오늘은 `Filter` 에 대해서 알아보도록 하겠습니다.

- [Logback] 1.SpringBoot에서 Logback 사용하기 - 주의점, 구성, 상속, 파라미터 처리
- [Logback] 2.SpringBoot에서 Logback 사용하기 - 환경설정 파일
- [Logback] 3.SpringBoot에서 Logback 사용하기 - Appender와 Policy
- [Logback] 4.SpringBoot에서 Logback 사용하기 - Encoder
- [Logback] 5.SpringBoot에서 Logback 사용하기 - MDC
- *[Logback] 6.SpringBoot에서 Logback 사용하기 - Filter*
- [Logback] 7.SpringBoot에서 Logback 사용하기 - 웹 어플리케이션 구성



## Filters

`logback-classic` 모듈은 `regular-filters`와 `turbo-filters` 두 가지 타입의 필더를 제공합니다.

- Regular filters
  - LevelFilter
  - ThresHolsFilter
  - EvaluatorFilter
    - GEventEvaluator
    - JanioEventEvaluator
  - CustomFilters
- TurboFilters
  - DuplicateMessageFilter
  - DynamicThresholdFilter
  - MarkerFilter
  - MatchingFilter
  - MDCFilter
  - ReconfigureOnChangeFilter
  - CustomFilters



### Regular filters

`Regular filters` 는  추상 클래스 `Filter` 를 상속하며, `ILoggingEvent` 를 파라미터로 받아오기 위한 `decide()` 메소드를 구현합니다. 등록된 Filter는 차례대로 decide(ILoggingEvent event) 메소드를 호출하며, enum 객체인 FilterReply를 반환합니다.

`FilterReply` 의 종류는 3가지가 존재합니다.

| FilterReply | 설명                                                         |
| ----------- | ------------------------------------------------------------ |
| `DENY`      | 로그 이벤트 동작을 취소하며, 남아있는 Filter들에 대하여 검증하지 않습니다. |
| `NEUTRAL`   | 다음 Filter에게 검증을 넘깁니다. 만약 다음 Filter가 없다면 로그 이벤트는 정상적으로 처리됩니다. |
| `ACCEPT`    | 로그 이벤트를 정상적으로 동작시키며, 다음 Filter들에 대하여 검증하지 않습니다. |

`Regular filters ` 는 `LevelFilter` 와  `ThresholdFilter` 입니다.



#### LevelFilter

로그 레벨이 일치하다면 로그를 필터링합니다. 다음은 로그 중 레벨이 `ERROR` 인 경우에 대해서 따로 파일로 보냅니다. 

> 해당 내용을 활용해볼 수 있는 방법은 개발단계에서는 로그 레벨이 `INFO` 또는 `DEBUG` 를 많이 사용합니다. 해당 Filter를 설정하여서 `ERROR` 인 로그만 따로 파일로 롤링한다면 ERROR가 발생한 로그를 쉽게 확인할 수 있습니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<included>
    <property name="LOG_HOME" value="./logs" />
    <property name="LOG_FILE" value="error-logFile.log" />
    <!--  appender이름이 file인 RollingFileAppender 선언  -->
    <appender name="error-file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <!--로깅이 기록될 위치-->
        <file>${LOG_HOME}/${LOG_FILE}</file>
        <!--로깅 파일이 특정 조건을 넘어가면 다른 파일로 만들어 준다.-->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/rolling/error-logFile.%d{yyyy-MM-dd}.%i.gz</fileNamePattern>
            <maxFileSize>50MB</maxFileSize>
            <maxHistory>7</maxHistory>
            <totalSizeCap>2GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <charset>utf8</charset>
            <pattern>[%d{yyyy.MM.dd HH:mm:ss.SSS}] - [%-5level] - [%X{traceId}] - [%logger{5}] - %msg%n</pattern>
        </encoder>
    </appender>

</included>
```

- `level` - 설정한 `ERROR` 레벨에 대해서
- `onMatch` - 로그 이벤트 중 `ERROR` 레벨과 일치한다면 해당 appender를 적용하고
- `onMismatch` - 일치하지 않다면 해당 appender를 적용하지 않습니다.



#### ThresholdFilter

지정한 로그 레벨보다 같거나 높은 수준의 로그 레벨에 대해서는 로그를 처리합니다. 지정한 로그 레벨보다 낮은 수준의 로그는 거부됩니다. 다음은 `WARN` 로그 레벨 이상의 로그를 보관하기 위해서 설정한 예시입니다. 

> `LevelFilter` 와 활용 목적에는 같지만, 로그 레벨의 범위를 지정할 수 있습니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<included>
    <property name="LOG_HOME" value="./logs" />
    <!--  appender이름이 file인 RollingFileAppender 선언  -->
    <appender name="warn-file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>WARN</level>
        </filter>
        <!--로깅 파일이 특정 조건을 넘어가면 다른 파일로 만들어 준다.-->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/rolling/warn-logFile.%d{yyyy-MM-dd}.%i.gz</fileNamePattern>
            <maxFileSize>50MB</maxFileSize>
            <maxHistory>7</maxHistory>
            <totalSizeCap>2GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <charset>utf8</charset>
            <pattern>[%d{yyyy.MM.dd HH:mm:ss.SSS}] - [%-5level] - [%X{traceId}] - [%logger{5}] - %msg%n</pattern>
        </encoder>
    </appender>

</included>
```

- `level` - 현재 설정된 레벨이 `WARN` 레벨로 `WARN`, `ERROR` 에 대해서만 appender를 적용합니다.



#### CustomFilter

특정 로그 처리에 대해서 Filter를 확장하여서 직접 개발을 해서 적용도 가능합니다. 먼저 아래 소스 먼저 보시면 어렵지 않게 적용할 수 있습니다.

아래 소스에서 특정 로그의 문자열이 `took` 이 포함되어 있으면 로그 이벤트에서 제외하고 그 외 문자열은 로그 이벤트로 처리합니다.

```java
public class CustomLogbackFilter extends Filter<ILoggingEvent> {
    @Override
    public FilterReply decide(ILoggingEvent event) {
        if (event.getMessage().contains("took")) {
            return FilterReply.DENY;
        } else {
            return FilterReply.ACCEPT;
        }
    }
}
```

xml에 아래와 같이 설정하면 됩니다.

```xml
...
<filter class="com.lovethefeel.webflux.config.CustomLogbackFilter" />
...
```



EvaluatorFilter와 TurboFiler에 대해서는 활용 방법에 대해서 좀 더 찾아보고 정리를 할 예정입니다. 



### EvaluatorFilter

#### JaninoEventEvaluator

`<evaluator>` 의 기본값은 `JaninoEventEvaluator` 입니다. `JaninoEventEvaluator` 는 Java 언어의 `boolean expressions` 을 사용합니다. 활용 예시는 다음 내용을 참고 바랍니다.

[Logback - EvaluatorFilter를 이용해서 경우에 따라 로그에 색입히기 예제](https://forgiveall.tistory.com/493)



##### Matchers

String 클래스에서 match() 메서드를 호출하여 패턴 일치를 수행하는 것이 가능하지만 필터가 호출될 때마다 `Pattern` 객체를 컴파일하는 비용이 발생합니다. `Pattern` 객체는 초기 생성시 값이 비싼 객체입니다. 관련하여 자세한 내용은 아래 참고 부탁드립니다.

[[Item6] 불필요한 객체를 생성하지 마라](https://lovethefeel.tistory.com/63?category=778504)

이 오버헤드를 제거하기 위해 하나 이상의 Matcher 개체를 미리 정의할 수 있습니다. 매처가 정의되면 반복적으로 참조될 수 있습니다.



#### GEventEvaluator

Groovy 언어의 `boolean expressions` 사용합니다. 로그 이벤트 필터링을 하기 위해서 유연하게 제공합니다.

예시는 Logback 공식 docs에서 확인하실 수 있습니다. - [예시](https://logback.qos.ch/manual/filters.html#GEventEvaluator)



### TurboFilters

`TurboFilter` 객체들은 모두 추상 클래스 `TurboFilter` 를 상속합니다. `TurboFilter` 는 기존 `Regular filter` 보다 더 넓은 scope를 가지고 있습니다. 그래서 scope이 `appender` 가 아닌 전체 로그의 대한 요청입니다.



#### MarkerFilter

`Marker` 를 이용해서 로그 필터링을 할 수 있으며 `MatchingFilter` 를 확장합니다. `Marker` 를 활용한 예시는 다음 내용을 참고바랍니다.

[로깅할 때 marker로 필터링하기](https://blog.leocat.kr/notes/2018/06/29/java-filtering-log-with-marker)



#### DuplicateMessageFilter

중복되는 로그 이벤트에 대해서 중복되지 않게 처리할 수 있습니다. 

예시는 Logback 공식 docs에서 확인하실 수 있습니다. - [예시](https://logback.qos.ch/manual/filters.html#DuplicateMessageFilter)



#### DynamicThresholdFilter

적절한 예시 찾는 중



#### MDCFilter

`MDC` 를 이용해서 로그 필터링을 할 수 있으며 `MatchingFilter` 를 확장합니다. 

`MDC` 에 대해서 알아보기 위해서는 아래 글 참고해주세요.

[[Logback] 5.SpringBoot에서 Logback 사용하기 - MDC]()



#### ReconfigureOnChangeFilter

적절한 예시 찾는 중



## 정리

- `Filter` 를 이용해서 원하는 로그 레벨에 대해서 출력 및 보관 처리가 가능합니다.
- 특정 클래스 또는 메시지의 문자열에 대해서도 로그에 대해서 변경 처리가 가능합니다. (로그 색 변경 등)



다음 시간에는 지금까지 알아보았던 Logback 관련 설정들을 이용해서 웹 어플리케이션을 구성해보도록 하겠습니다.



## 참고 자료

- [Logback Project](https://logback.qos.ch/index.html)
