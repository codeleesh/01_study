# [Logback] 5.SpringBoot에서 Logback 사용하기 - MDC

로그를 작성하기 위해서 자주 사용하는 로그백의 대한 내용을 정리하고 있습니다.

지난 블로그 내용에서는  `Encoder` 에 대해서 알아보았습니다. 

오늘은 `MDC` 에 대해서 알아보도록 하겠습니다.

- [Logback] 1.SpringBoot에서 Logback 사용하기 - 주의점, 구성, 상속, 파라미터 처리
- [Logback] 2.SpringBoot에서 Logback 사용하기 - 환경설정 파일
- [Logback] 3.SpringBoot에서 Logback 사용하기 - Appender와 Policy
- [Logback] 4.SpringBoot에서 Logback 사용하기 - Encoder
- *[Logback] 5.SpringBoot에서 Logback 사용하기 - MDC*
- [Logback] 6.SpringBoot에서 Logback 사용하기 - Filter
- [Logback] 7.SpringBoot에서 Logback 사용하기 - 웹 어플리케이션 구성



## Mapped Diagnostic Contexts

로그백의 주요 설계 목표 중 하나는 복잡한 분산 응용 프로그램을 추적 및 디버그하는 것입니다. 대부분의 실제 분산 시스템은 여러 클라이언트를 동시에 처리해야 합니다. 예를 들어, 요청 A와 B가 호출되어 각각 다른 쓰레드에서 실행이 되었을때, 로그 메시지기는 섞이게 됩니다. 요청 A에 대해서 로그만 분리해서 보기는 어렵습니다. 이러한 문제를 해결하기 위해서는 로그를 기록할때, 요청마다 고유의 ID를 부여해서 로그를 기록하게 된다면 고유의 ID를 이용해서 각 요청마다 로그를 묶어서 볼 수 있습니다. 자바에서는 이러한 부분을 구현하기 위해서 `ThreadLocal` 변수를 통해서 해결하였습니다.



### ThreadLocal

`ThreadLocal` 은 쓰레드의 로컬 컨텍스트 변수입니다. Thread가 존재하는 한 계속해서 남아 있는 변수입니다. 하나의 작업 요청이 들어와서 처리한다면, 하나의 쓰레드가 생성이 되고 작업이 끝나게 되면 쓰레드가 없어집니다. 쓰레드가 살아있는 동안에 계속 유지되는 변수입니다. 이 `ThreadLocal` 의 ID 변수에 고유한 ID를 저장해놓고, 각 메서드에서 이 값을 불러서 로그 출력시 함께 출력하는 방법입니다.



### 로그백의 MDC

일일이 구현하기에는 불편할 수 있기 때문에, 이러한 기능을 `MDC` 로 제공합니다.

- 스레드 단위로 관리되는 맵을 사용할 수 있게 한다.
  - 실행 스레드들에 공통값을 주입하여 의미있는 정보를 추가해 로깅 할수있도록 제공한다.
- 서버에서 여러 요청을 동시에 처리할 때 이를 구분할 수 있게 해준다. 



### 예시

요청과 응답의 대해서 공통적으로 로그를 설정하고, `MDC`를 활용하여 요청과 응답의 대한 로그 관리를 하려고 합니다.

아래는 `WebFlux` 에서 사용하기 위해 작성한 `WebMDCFilter` 입니다. 요청지점(`doBeforeRequest`)에서 UUID를 발급하여서 MDC에 저장해놓습니다. 응답지점(`doAfterRequest`)에서 MDC를 모두 지우게 됩니다. AOP를 활용하는 방법 또한 있습니다.

```java
@Slf4j
@Component
public class WebMDCFilter implements WebFilter {

    private static final String TRACE_ID = "traceId";

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, WebFilterChain chain) {
        return chain.filter(exchange).transformDeferred(this::doFilter);
    }

    private Publisher<Void> doFilter(Mono<Void> call) {
        // before
        return Mono.fromRunnable(this::doBeforeRequest)
                // do request
                .then(call)
                // after (success)
                .doOnSuccess(done -> doAfterRequest());
    }
    private void doBeforeRequest() {
        log.info("===================== START =====================");
        final String traceId = UUID.randomUUID().toString();
        MDC.put(TRACE_ID, traceId);
        log.info("traceId : {}", traceId);
    }
    private void doAfterRequest() {
        log.info("MDC GET : {}", MDC.get(TRACE_ID));
        MDC.clear();
        log.info("MDC CLEAR : {}", MDC.get(TRACE_ID));
        log.info("===================== END =====================");
    }
}
```



아래는 파일 롤링을 위해서 설정한 환경설정파일에 `traceId` 를 설정한 것입니다.

```xml
<?xml version="1.0" encoding="UTF-8"?>

<included>
    <property name="LOG_HOME" value="./logs" />
    <property name="LOG_FILE" value="logFile.log" />
    <!--  appender이름이 file인 RollingFileAppender 선언  -->
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
            <pattern>[%d{yyyy.MM.dd HH:mm:ss.SSS}] - [%-5level] - [%X{traceId}] - [%logger{5}] - %msg%n</pattern>
        </encoder>
    </appender>

</included>
```



상품을 등록하는 API를 실행하였고 아래 로그는 요청부터 응답까지의 결과입니다. `traceId : b550143a-ba2c-4aab-88b6-1c79def13417` 를 통해서 로그 추적이 가능해집니다. 통합로그모니터링 시스템과 연계한다면 해당 `traceId` 를 통해서 관리가 더 쉬워집니다.

```
[2022.05.21 21:48:04.903] - [INFO ] - [] - [c.l.w.c.WebMDCFilter] - ===================== START =====================
[2022.05.21 21:48:04.903] - [INFO ] - [b550143a-ba2c-4aab-88b6-1c79def13417] - [c.l.w.c.WebMDCFilter] - traceId : b550143a-ba2c-4aab-88b6-1c79def13417
[2022.05.21 21:48:05.067] - [INFO ] - [b550143a-ba2c-4aab-88b6-1c79def13417] - [c.l.w.p.a.ProductService] - saveUser - productRequest [ProductRequest{productName='치킨', productAmount=10000, productStatus=ENABLE, productCount=10}]
insert into product (id, product_amount, product_count, product_name, product_type) values (default, ?, ?, ?, ?)
insert into product (id, product_amount, product_count, product_name, product_type) values (default, 10000, 10, '치킨', 'ENABLE');
[2022.05.21 21:48:05.123] - [INFO ] - [b550143a-ba2c-4aab-88b6-1c79def13417] - [c.l.w.p.a.ProductService] - saveUser - productResponse [ProductResponse{productId=1, productName='치킨', productType=ENABLE, productCount=10}]
[2022.05.21 21:48:05.170] - [INFO ] - [b550143a-ba2c-4aab-88b6-1c79def13417] - [c.l.w.c.WebMDCFilter] - MDC GET : b550143a-ba2c-4aab-88b6-1c79def13417
[2022.05.21 21:48:05.171] - [INFO ] - [] - [c.l.w.c.WebMDCFilter] - MDC CLEAR : null
[2022.05.21 21:48:05.171] - [INFO ] - [] - [c.l.w.c.WebMDCFilter] - ===================== END =====================
```



## 정리

- 복잡한 분산 응용 프로그램을 추적 및 디버그하기 위해서 로그 프레임워크에서는 `MDC` 를 사용합니다.
- `ThreadLocal` 의 특징을 이용하여서 고유한 ID를 저장 및 로그 출력시 활용합니다.
- 요청과 응답의 대한 로그 관리를 위해서 공통 소스를 활용하면 각 메소드마다 소스를 추가하지 않아도 됩니다.



다음 시간에는 특정 로그에 대해서만 처리를 할 수 있는 `Filter` 에 대해서 알아보도록 하겠습니다.



## 참고 자료

- [Logback Project](https://logback.qos.ch/index.html)
- [조대협의 블로그 - 로그시스템 #4-MDC를 이용하여 쓰레드별로 로그 분류하기](https://bcho.tistory.com/1316)
- [Baeldung - ThreadLocal](https://www.baeldung.com/java-threadlocal)
