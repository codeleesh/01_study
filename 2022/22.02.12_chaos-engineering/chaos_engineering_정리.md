최근에 회사에서 개발중인 시스템에서 아래와 같은 테스트 상황을 개발하게 되었습니다.

내용은 다음과 같습니다.

- 특정 사용자가 접속하여서 API를 호출하면 응답 지연을 할 수 있어야 한다.

해당 내용을 듣자 바로 AOP가 생각나서 그렇게 개발을 하였습니다.

그러다 `Chaos Engineering`이라는 내용을 알게 되었습니다.

그래서 기록하기 위해서 정리를 하게 되었습니다.



# Chaos Engineering

운영 환경에서도 갑작스러운 장애를 견딜 수 있는 시스템을 구축하기 위해 시스템을 실험하는 분야입니다.



## 시스템의 약점

- 서비스가 이용 불가능할때의 부적절한 대응
- 잘못 설정된 타임아웃 값으로 인한 지나친 재시도
- 뒷단의 시스템이 대규모 트래픽을 견디지 못해 발생한 장애
- 한 지점의 장애로 인해 연달아 발생하는 장애



이와 같은 것들이 실제 운영환경에서 고객 경험에 영향을 미치기 전에 적극적으로 해결해야 합니다. 시스템에 내재된 혼란을 관리하고, 유연성과 속도를 높여주고, 프로그램이 복잡할지라도 운영 환경에서 배포에 확신을 가질 수 있는 방법이 필요합니다.

경험이 뒷받침된 시스템 접근 방식은 대규모 분산 시스템에서의 혼란을 해결하고 이러한 시스템이 실제 환경에서의 변수에도 견딜 수 있다는 확신을 심어줄 수 있습니다. 



## Chaos Engineering 원칙

Chaos Engineering은 다음과 같은 원칙을 갖고 있습니다.

- 정상 상태와 동작에 관한 가설을 세운다.
- 다양한 실제 이벤트 변수들을 적용한다.
- 운영 환경에서 실험을 진행한다.
- 자동화를 통해 지속적으로 실험을 진행한다.
- 위험범위를 최소화한다.



## 장애 주입(Failure Injection)

작은 범위에서 시작하여서 점진적으로 신뢰성을 구축해 나가는 것이 포인트입니다.

- 애플리케이션 부하 테스트
- 호스트 서버 이슈 발생
- 데이터베이스 서버 셧다운
- 자원 공격(CPU, memory, ...)
- 네트워크 공격(dependencies, latency, ...)
- 데이터 센터 공격



## Chaos Engineering 도구

설치형 관리도구는 아래와 같습니다.

- [Chaos Mesh](https://chaos-mesh.org/)
- [Gremlin](https://www.gremlin.com/)
- [Litmus](https://litmuschaos.io/)
- [Chaos Monkey](https://netflix.github.io/chaosmonkey/)

간단한 라이브러리 추가로 이용해볼 수 있는 라이브러리도 있습니다.

- [Chaos Monkey for Spring Boot](https://codecentric.github.io/chaos-monkey-spring-boot/)



실험을 위해서는 간단한 라이브러리 추가로 이용해보도록 해보겠습니다.

그 후 설치형 관리도구 중 하나를 선택해서 진행을 해볼 예정입니다.



### Chaos Monkey For Spring Boot (`CM4SB`)

>  앞으로 CM4SB로 명칭을 하도록 표기하겠습니다.

- Netflix에서 만든 카오스 엔지니어링 라이브러리의 영감을 받아서 만들었습니다.
- Spring Boot에서 쉽게 사용할 수 있도록 만든 라이브러리입니다.



### CM4SB 동작 방식

> 참고
>
> [Chaos Monkey for Spring Boot Reference Guide](https://codecentric.github.io/chaos-monkey-spring-boot/latest/)

![sb-chaos-monkey-architecture](./images/sb-chaos-monkey-architecture.png)

- 크게 3가지 영역으로 나누어집니다. 
  - 공격 대상인 `애플리케이션`과 그것을 감시하는 `watchers` 존재하고 감시를 통해 어떠한 `assaults`(공격 종류)를 할지 정할 수 있습니다.

- 공격 대상 : `@Controller`, `@Repository`, `@Service`, `@RestController`, `@Component`
- Watcher : 감시자 (공격 대상 별로 Watcher가 존재)
- Assault : 공격 종류



### Watcher 종류

감시자의 종류는 다음을 항목들을 이용하여서 애플리케이션에서 설정할 수 있습니다.

- Annotation Watchers
- Actuator Watchers
- Outgoing Request Watchers
- Alternative Bean Watcher



#### Annotation Watchers

애노테이션이 애플이케이션의 설정되어 있고, watchers 설정 API를 이용하여 활성화하면 감시자 역할을 할 수 있습니다.

- @Controller
- @RestController
- @Service
- @Repository
- @Component



#### Actuator Watchers

Spring Boot의 auto-configures인 `HealthIndicators`를 볼 수 있습니다.

> 참고
>
> [Production-ready Features (spring.io)](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints.health.auto-configured-health-indicators)



#### Outgoing Request Watchers

RestTemplate 및 WebClient 대한 감시자를 할 수 있도록 제공합니다.

- RestTemplate
- WebClient

> ❗️참고
>
> `new RestTemplate()` 및 `WebClient.create()`를 통해 빈으로 생성되지 않은 RestTemplate 및 WebClient는 대상에서 제외됩니다.



#### Alternative Bean Watcher

Spring profile인 `chaos-monkey` 이름으로 활성화되어 있으면 애플리케이션의 모든 Bean을 처리할 수 있습니다.



### Customize Watcher

`watchCustomServices` 속성을 사용하여 공격 대상인 클래스와 메서드를 직접 지정할 수 있습니다.

`watchedCustomServices`가 설정되어 있지 않으면 감시자가 활성화된 모든 클래스와 메서드가 공격을 받을 수 있습니다.

Spring Boot Actuator Endpoint를 사용하여 런타임에 조정할 수 있습니다.



API 호출 - Chaos Monkey Spring Boot Actuator Endpoint (/actuator/chaosmonkey/assaults)

```
{
  "level": 3,
  "latencyActive": true,
  "latencyRangeStart": 1000,
  "latencyRangeEnd": 3000,
  "watchedCustomServices": [
    "com.example.chaos.monkey.chaosdemo.controller.HelloController.sayHello",
    "com.example.chaos.monkey.chaosdemo.service.HelloService"
  ]
}
```



application.yml 설정

```
chaos:
  monkey:
    enabled: true
    watcher:
      controller: true
    assaults:
      level: 1
      latency-active: true
      watched-custom-services:
        - com.example.chaos.monkey.chaosdemo.controller.HelloController.sayHello
        - com.example.chaos.monkey.chaosdemo.service.HelloService
```

> 참고
>
> 설정된 목록만 감시자가 체크합니다.



### Assaults 종류

사용자의 구성에 따라 공격을 사용하며 다음 항목을 제공합니다.

- Request Assaults
- Runtime Assaults
- Chaos Monkey Assault Scheduler



Assaults 활성화 확인 여부는 다음 API를 통해서 확인 가능합니다.

API 호출 - /chaosmonkey/watchers - Response 200 OK

```
{
  "controller": false,
  "restController": true,
  "service": false,
  "repository": false,
  "component": false,
  "restTemplate": false,
  "webClient": false,
  "actuatorHealth": false
}
```



#### Request Assaults

요청 공격의 항목은 다음과 같습니다.

- Latency Assaults
- Exception Assaults



##### Latency Assaults

- 활성화된 경우 요청에 지연 시간이 추가됩니다.

- 지연 시간에 대한 발생 확률도 제어 가능합니다.



##### Exception Assaults

- 예외 공격, 메소드를 실행할 때 예외 발생 여부를 런타임 시점에 결정할 수 있습니다.

- 거의 모든 종류의 RuntimeException을 던질 수 있습니다.

- Actuator Endpoint를 통해 런타임 시 필요한 예외를 구성할 수 있습니다.



#### Runtime Assaults

전체 애플리케이션의 대한 공격입니다.

공격의 대한 트리거를 사용하기 위해서는 Chaos Monkey Assault Scheduler 또는 Chaos Monkey의 엔드포인트를 활용해야 합니다.

> 참고
>
> [Assault Scheduling](https://codecentric.github.io/chaos-monkey-spring-boot/latest/#_actuator_watchers)
>
> [Chaos Monkey Endpoint](https://codecentric.github.io/chaos-monkey-spring-boot/latest/#assaultsattack)



다음 항목을 제공합니다.

- Appkiller Assaults
- Memory Assaults
- Cpu Assaults



##### Appkiller Assaults

- 특정 메소드 실행시 프로그램 종료 공격을 합니다.



##### Memory Assaults

- 메모리 공격은 Java 가상 머신의 메모리를 공격합니다.

- 메모리 공격은 사용 중인 Java 버전에 따라 크게 달라지며 각 자바 버전의 기본 가비지 수집기로 테스트 합니다.



##### Cpu Assaults

- Java 가상 머신의 CPU를 공격합니다.



#### Chaos Monkey Assault Scheduler

예약(`Cron`)을 통하여 Runtime Assaults(Memory, CPU, AppKiller) 가능합니다.

> 참고
>
> [Configuration](https://codecentric.github.io/chaos-monkey-spring-boot/latest/#chaos_monkey_assault_scheduler)



### Properties

> 참고
>
> [Chaos Monkey properties](https://codecentric.github.io/chaos-monkey-spring-boot/latest/#_properties)

| Property                                                    | Description                                                  | Values                                                       | Default                                                      |
| ----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| chaos.monkey.enabled                                        | chaos.monkey 활성화 유무                                     | TRUE or FALSE                                                | FALSE                                                        |
| chaos.monkey.assaults.level                                 |                                                              | 1-10000                                                      | 1                                                            |
| chaos.monkey.assaults.latencyRangeStart                     | 최소 지연 시간                                               | Integer.MIN_VALUE, Integer.MAX_VALUE                         | 1000                                                         |
| chaos.monkey.assaults.latencyRangeEnd                       | 최대 지연 시간                                               | Integer.MIN_VALUE, Integer.MAX_VALUE                         | 3000                                                         |
| chaos.monkey.assaults.latencyActive                         | latency 공격 활성                                            | TRUE or FALSE                                                | FALSE                                                        |
| chaos.monkey.assaults.exceptionsActive                      | exceptions 공격 활성                                         | TRUE or FALSE                                                | FALSE                                                        |
| chaos.monkey.assaults.exception                             | 사용자 정의 RuntimeException 또는 기본 RuntimeException      | de.codecentric.spring.boot.chaos. monkey.configuration.AssaultException | java.lang.RuntimeException("Chaos Monkey - RuntimeException"") |
| chaos.monkey.assaults.killApplicationActive                 | AppKiller 공격 활성                                          | TRUE or FALSE                                                | FALSE                                                        |
| chaos.monkey.assaults.killApplication.cron.expression       | Cron 표현식은 일정에 따라 AppKiller assaults 활성화 설정 ex) `*/1 * * * * ?` | Any valid cron expression (or OFF)                           | OFF                                                          |
| chaos.monkey.watcher.controller                             | Controller watcher 활성화 유무                               | TRUE or FALSE                                                | FALSE                                                        |
| chaos.monkey.watcher.restController                         | RestController watcher 활성화 유무                           | TRUE or FALSE                                                | FALSE                                                        |
| chaos.monkey.watcher.service                                | Service watcher 활성화 유무                                  | TRUE or FALSE                                                | FALSE                                                        |
| chaos.monkey.watcher.repository                             | Repository watcher 활성화 유무                               | TRUE or FALSE                                                | FALSE                                                        |
| chaos.monkey.watcher.component                              | Component watcher 활성화 유무                                | TRUE or FALSE                                                | FALSE                                                        |
| chaos.monkey.watcher.beans                                  | 감시할 대상자 빈                                             | List of bean names                                           | Empty list                                                   |
| chaos.monkey.assaults.memoryActive                          | Memory assault 활성화 유무                                   | TRUE or FALSE                                                | FALSE                                                        |
| chaos.monkey.assaults.memoryMillisecondsHoldFilledMemory    | 설정된 메모리값에 도달하였을때 공격 시간                     | min=1500, max=Integer.MAX_VALUE                              | 90000                                                        |
| chaos.monkey.assaults.memoryMillisecondsWaitNextIncrease    | 메모리 사용량 증가되는 시간                                  | min=100, max=30000                                           | 1000                                                         |
| chaos.monkey.assaults.memoryFillIncrementFraction           | Fraction of one individual memory increase iteration. `1.0` equals 100 %. | min=0.01, max=1.0                                            | 0.15                                                         |
| chaos.monkey.assaults.memoryFillTargetFraction              | Final fraction of used memory by assault. `0.95` equals 95 %. | min=0.01, max=0.95                                           | 0.25                                                         |
| chaos.monkey.assaults.memory.cron.expression                | Cron 표현식은 일정에 따라 memory assaults 활성화 설정 ex) `*/1 * * * * ?` | Any valid cron expression (or OFF)                           | OFF                                                          |
| chaos.monkey.assaults.cpuActive                             | CPU assault 활성화 유무                                      | TRUE or FALSE                                                | FALSE                                                        |
| chaos.monkey.assaults.cpuMillisecondsHoldLoad               | Duration to assault cpu when requested load is reached in ms. | min=1500, max=Integer.MAX_VALUE                              | 90000                                                        |
| chaos.monkey.assaults.cpuLoadTargetFraction                 | Final fraction of used cpu by assault. `0.95` equals 95 %.   | min=0.1, max=1.0                                             | 0.9                                                          |
| chaos.monkey.assaults.cpu.cron.expression                   | Cron 표현식은 일정에 따라 cpu assaults 활성화 설정 ex) `*/1 * * * * ?` | Any valid cron expression (or OFF)                           | OFF                                                          |
| chaos.monkey.assaults.runtime.scope.assault.cron.expression | Cron 표현식은 일정에 따라 cpu runtime assaults 활성화 설정 ex) `*/1 * * * * ?` | Any valid cron expression (or OFF)                           | OFF                                                          |
| chaos.monkey.assaults.watchedCustomServices                 | 감시 대상자 직접 지정(패키지/클래스/메소드)                  | List of fully qualified packages, class and/or method names  | Empty list                                                   |



## 정리

개발을 하면서 장애의 상황을 만나게 되지만 일어나서는 안된다고만 생각하고 있었고 실제로 일어난다면 어쩔 수 없는 상황이라고만 생각하였습니다. 

하지만 그것 또한 고객의 입장에서 좋은 경험이 아니기 때문에 이를 연습하여서 사전에 대비한다면 좋을 것 같습니다.

Chaos Engineering으로 장애 상황의 대한 경험을 해본다면 문제가 생겼을 때 좀 더 유연하고 속도있게 해결할 수 있을거라 생각됩니다.



다음 블로그에는 직접 실습을 해보면서 어떻게 적용하는지 알아보도록 하겠습니다.



# 참고

https://principlesofchaos.org/ko/

https://codecentric.github.io/chaos-monkey-spring-boot/latest/

https://www.slideshare.net/Channy/chaos-on-microservices