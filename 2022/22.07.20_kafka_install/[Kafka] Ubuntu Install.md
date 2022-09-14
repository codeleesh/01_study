# [Kafka] Ubuntu Install

<img src="/Users/lsh/Downloads/logo/kafka-logo.png" alt="kafka-logo" style="zoom:50%;" />



요즘 카프카에 대해서 공부를 하고 있습니다. 카프카 실습을 위해서 로컬에서 카프카 설치 및 실행을 해보도록 하겠습니다.

목차는 다음과 같습니다.

- 설치 파일 다운로드
- 사전 설치
- 본 설치 
- 실행
- 문제 해결



## 설치 파일 다운로드

카프카 바이너리 파일 다운로드는 아래 사이트에서 할 수 있습니다

- https://kafka.apache.org/downloads
- Binary downloads : kafka_2.12-2.5.0.tgz



### 파일 버전 설명

`kafka_2.12-2.5.0`

- kafka_ 다음으로 나오는 `2.12` 는 Scala 버전입니다.
- `2.5.0` 버전에서 제일 앞에 있는 번호는 0, 1, 2, 3 버전까지 나왔으며, 1 ~ 3 버전까지는 호환이 가능합니다.



## 사전 설치

- jdk 1.8 이상



## 본 설치



### 바이너리 압축 해제

공식 사이트에서 다운로드 받은 `kafka_2.12-2.5.0.tgz` 파일을 압축해제합니다.	

```bash
$ tar -xvzf kafka_2.12-2.5.0.tgz
```



압축을 해제하면 다음과 같은 폴더 구조를 확인할 수 있습니다.

```bash
$ tree -L 1
.
├── LICENSE
├── NOTICE
├── bin
├── config
├── libs
└── site-docs
```

- bin : 실행할 바이너리 파일과 shell script 파일
- config : 설정의 필요한 `properties` 파일 등 여러 설정 파일
- libs : 브로커를 실행하기 위한 라이브러리 파일



브로커에서 적재할 폴더를 생성합니다. 프로듀서를 통해 전송된 데이터는 해당 폴더에 저장이 됩니다.

```bash
$ mkdir data
$ ls
LICENSE  NOTICE  bin  config  data  libs  site-docs
```



### properties 파일 확인

`server.properties`

```bash
$ vi config/server.properties
```



아래 내용을 확인 후 수정합니다.

```properties
# 수신 주소 및 포트 설정
listeners=PLAINTEXT://localhost:9092
advertised.listeners=PLAINTEXT://localhost:9092

# 프로듀서가 보낸 데이터를 브로커에서 파일시스템에 저장하는 경로
log.dirs=/home/lovethefeel/dev_program/kafka_2.12-2.5.0/data

# 카프카 토픽을 만들때 기본적인 파티션의 갯수, 설정을 안하였을 경우 브로커에 설정된 기본 갯수로 설정
num.partitions=3

# 카프카 브로러카 파일시스템에 저장할때 관리 정보들
log.retension.hours=168
log.segment.bytes=1073741824
log.retention.check.interval=300000
```



###  참고 

properties 내용 수정 후 cat을 이용해서 해당 키워드 내용 확인

```bash
$ cat config/server.properties | grep log.dirs
log.dirs=/home/lovethefeel/dev_program/kafka_2.12-2.5.0/data
```



## 실행



### 주키퍼 실행

`zookeeper.properties`

```bash
$ cat config/zookeeper.properties
dataDir=/tmp/zookeeper
clientPort=2181
maxClientCnxns=0
admin.enableServer=false
```



주키퍼는 다음 명령어로 실행할 수 있습니다.

```bash
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

- 주키퍼에서는 내부적으로 가장 알맞은 데이터를 고르는 과정
- 주키퍼의 경우 3개 이상의 서버에서 앙상블로 묶어서 연동하는 것을 일반적
- 로컬에서 1대의 서버에서 설치 및 테스트 가능



### 카프카 브로커 실행

카프카 브로커는 다음 명령어로 실행할 수 있습니다.

```bash
$ bin/kafka-server-start.sh config/server.properties
```

로그가 다음과 같이 나오면 정상적으로 기동되었습니다.

```bash
[2022-07-20 23:26:36,529] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
```



### 카프카 정상 실행 여부 확인

브로커가 정상적으로 확인하기 위한 shell script 제공합니다.

```bash
$ bin/kafka-broker-api-versions.sh --bootstrap-server localhost:9092
```

실행 후 로그는 다음과 같습니다.

```bash
lovethefeel-H310H5-M7:9092 (id: 0 rack: null) -> (
	Produce(0): 0 to 8 [usable: 8],
  ...
	OffsetDelete(47): 0 [usable: 0]
)
```



추가적으로 카프카 브로커의 토픽 리스트를 확인할 수 있습니다.

```bash
$ bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```



## 문제 해결



### 카프카 브로커 실행시 cluster ID 에러

어느 날 브로커 서버를 시작하니 아래와 같은 에러메세지가 나왔습니다.

```
[2022-08-05 13:33:16,867] INFO [ZooKeeperClient Kafka server] Connected. (kafka.zookeeper.ZooKeeperClient)
[2022-08-05 13:33:17,073] INFO Cluster ID = 3KDCmkqlSnyppX7wnrwBTQ (kafka.server.KafkaServer)
[2022-08-05 13:33:17,084] ERROR Fatal error during KafkaServer startup. Prepare to shutdown (kafka.server.KafkaServer)
kafka.common.InconsistentClusterIdException: The Cluster ID 3KDCmkqlSnyppX7wnrwBTQ doesn't match stored clusterId Some(JGMp1-dTTFmaUsprUubzWQ) in meta.properties. The broker is trying to join the wrong cluster. Configured zookeeper.connect may be wrong.
	at kafka.server.KafkaServer.startup(KafkaServer.scala:223)
	at kafka.server.KafkaServerStartable.startup(KafkaServerStartable.scala:44)
	at kafka.Kafka$.main(Kafka.scala:82)
	at kafka.Kafka.main(Kafka.scala)
```

- 다음과 같은 에러가 발생한 것은 cluster ID 가 맞지 않는다는 것입니다.
  - 현재 Cluster ID : 3KDCmkqlSnyppX7wnrwBTQ
  - meta.properties에 등록된 Cluster ID : JGMp1-dTTFmaUsprUubzWQ



#### 해결 방법

프로듀서가 보낸 데이터를 브로커에서 파일시스템에 저장하는 경로에서 meta.properties 안에 내용을 수정 또는 해당 파일을 삭제하면 됩니다.

파일 경로는 다음과 같습니다.

```properties
log.dirs=/home/lovethefeel/dev_program/kafka_2.12-2.5.0/data
```

`<log.dirs>/meta.properties` 에서 client ID 를 수정해 주면 된다.

```bash
$ cd /home/lovethefeel/dev_program/kafka_2.12-2.5.0/data
$ tree
.
├── cleaner-offset-checkpoint
├── log-start-offset-checkpoint
├── meta.properties
├── recovery-point-offset-checkpoint
└── replication-offset-checkpoint
```

```bash
rm -rf meta.properties
```



## 정리

- kafka 브로커는 데이터를 저장할 폴더가 필요합니다.
- kafka의 주키퍼와 브로커 환경설정 파일은 따로 분리되어 있습니다.
  - 주키퍼의 환경설정 파일은 `zookeeper.properties` 입니다.
  - 브로커의 환경설정 파일은 `config/server.properties` 입니다.
- kafka 브로커 실행 후 정상 실행 여부를 확인할 수 있는 script가 있습니다.
- kafka 브로커의 토픽 리스트를 확인할 수 있는 script도 제공합니다.