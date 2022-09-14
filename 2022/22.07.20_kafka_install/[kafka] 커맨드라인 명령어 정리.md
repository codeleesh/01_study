# [kafka] 커맨드라인 명령어 정리

<img src="/Users/lsh/Downloads/logo/kafka-logo.png" alt="kafka-logo" style="zoom:50%;" />



요즘 카프카에 대해서 공부를 하고 있습니다. 카프카를 사용하면서 자주 사용하는 커맨드 라인 명령어를 정리해보도록 하겠습니다.

목차는 다음과 같습니다.

- kafka-topics.sh
- kafa-config.sh
- kafka-console-producer.sh
- kafka-console-consumer.sh
- kafka-consumer-groups.sh
- 그외 커맨드 라인 명령어
  - kafka-producer-perf-test.sh
  - kafka-consumer-perf-test.sh
  - kafka-reassign-partitions.sh
  - kafka-delete-record.sh
  - kafka-dump-log.sh
- 기타



## kafka-topics.sh

### 토픽 생성

카프카 클러스터 정보와 토픽 이름만으로 토픽 생성할 수 있습니다.

- 클러스터 정보와 토픽 이름은 토픽을 만들기 위한 필수 값
- 파티션 개수, 복제 개수 등과 같이 다양한 옵션이 포함되어 있지만 모두 브로커에 설정된 기본값으로 생성

```bash
$ bin/kafka-topics.sh --create --bootstrap-server my-kafka:9092 --topic hello.kafka
```

결과는 다음과 같습니다. 

```
WARNING: Due to limitations in metric names, topics with a period ('.') or underscore ('_') could collide. To avoid issues it is best to use either, but not both.
Created topic hello.kafka.
```

- topic 생성 시 `.` 보다는 `_` 를 사용하라고 하니 참고해야 겠습니다. 



### 좀 더 상세한 토픽 생성

파티션 개수, 복제 개수, 토픽 데이터 유지 기간 옵션들을 지정하여 토픽을 생성이 가능합니다.

```bash
$ bin/kafka-topics.sh --create 
	--bootstrap-server my-kafka:9092 
	--partitions 10
	--replication-factor 1
	--topic hello.kafka2
	--config retention.ms=172800000
```

지금까지 생성한 카프카 브로커의 토픽 리스트를 확인할 수 있습니다.

```
bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --list
```

```
hello.kafka
hello.kafka2
```



### 토픽 상세 정보 확인

만들어진 토픽에 상세 정보를 확인할 수 있습니다.

```bash
$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --describe
```

결과는 다음과 같습니다.

```
Topic: hello.kafka	PartitionCount: 3	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: hello.kafka	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: hello.kafka	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: hello.kafka	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
```

- `PartitionCount` : 파티션 개수
- `ReplicationFactor` : 복제 개수
- `segment.bytes` : 



### 기 생성된 토픽의 대한 파티션 개수를 늘리기 위한 방법

먼저 기본 옵션으로 토픽을 생성하도록 하겠습니다.

```bash
$ bin/kafka-topics.sh --create --bootstrap-server my-kafka:9092 --topic test
Created topic test.
```

생성된 토픽의 상세 정보를 확인해보도록 하겠습니다.

```bash
$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --topic test --describe
```

상세 결과는 다음과 같습니다.

```
Topic: test	PartitionCount: 3	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: test	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
```

- 현재 설정된 파티션의 개수는 3입니다.
  - `PartitionCount` : 3 



설정된 파티션의 개수를 변경해보도록 하겠습니다. `alter` 를 이용하면 됩니다.

```bash
$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --topic test --alter --partitions 4
```

변경된 토픽의 상세 정보를 확인해보도록 하겠습니다.

```bash
$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --topic test --describe
```

상세 결과는 다음과 같습니다.

```
Topic: test	PartitionCount: 4	ReplicationFactor: 1	Configs: segment.bytes=1073741824
	Topic: test	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 3	Leader: 0	Replicas: 0	Isr: 0
```

- 변경된 파티션의 개수는 4입니다.
  - `PartitionCount` : 4 



> Consumer의 데이터 처리량을 늘리는 가장 쉬운 방법
>
> 파티션 개수를 기존 1개에서 2로 늘리게 되면 데이터 처리량이 2배가 늘어나개 됩니다.



### 기 생성된 토픽의 대한 파티션 개수를 줄이는 방법

파티션 개수를 늘릴 수 있지만 줄일 수는 없습니다. 다시 줄이는 명령을 내리면 `InvalidPartitionsException` 익셉션이 발생합니다.

*파티션 개수를 늘리는 것은 왜 익셉션이 발생할까요?*

분산 시스템에서 이미 분산된 데이터를 줄이는 방법은 매우 복잡합니다. 삭제 대상 파티션을 지정해야할 뿐만 아니라 기존에 저장되어 있던 레코드를 분산하여 저장하는 로직이 필요합니다. 카프카에서는 파티션을 줄이는 로직은 제공하지 않습니다. 만약 사정상 파티션 개수를 줄여야 한다면 새로 만드는 것이 좋습니다.

아래는 파티션을 줄여보고 에러가 나타나는 상황을 시도하였습니다.

```
bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --topic test --alter --partitions 2
```

```
Error while executing topic command : org.apache.kafka.common.errors.InvalidPartitionsException: Topic currently has 4 partitions, which is higher than the requested 2.
[2022-08-05 14:12:15,545] ERROR java.util.concurrent.ExecutionException: org.apache.kafka.common.errors.InvalidPartitionsException: Topic currently has 4 partitions, which is higher than the requested 2.
	at org.apache.kafka.common.internals.KafkaFutureImpl.wrapAndThrow(KafkaFutureImpl.java:45)
	at org.apache.kafka.common.internals.KafkaFutureImpl.access$000(KafkaFutureImpl.java:32)
	at org.apache.kafka.common.internals.KafkaFutureImpl$SingleWaiter.await(KafkaFutureImpl.java:89)
	at org.apache.kafka.common.internals.KafkaFutureImpl.get(KafkaFutureImpl.java:260)
	at kafka.admin.TopicCommand$AdminClientTopicService.alterTopic(TopicCommand.scala:270)
	at kafka.admin.TopicCommand$.main(TopicCommand.scala:64)
	at kafka.admin.TopicCommand.main(TopicCommand.scala)
Caused by: org.apache.kafka.common.errors.InvalidPartitionsException: Topic currently has 4 partitions, which is higher than the requested 2.
 (kafka.admin.TopicCommand$)
```



## kafa-config.sh

토픽의 일부 옵션을 설정하기 위해서는 kafka-configs.sh 명령어를 사용해야 합니다. `--alter` , `--add-config` 옵션을 사용하여 min.insync.replicas 옵션을 토픽별로 설정할 수 있습니다.

```bash
$ bin/kafka-configs.sh --bootstrap-server my-kafka:9092 --alter --add-config min.insync.replicas=2 --topic test
```

```
Completed updating config for topic test.
```

설정된 옵션을 알아보겠습니다.

```bash
$ bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --topic test --describe
```

```
Topic: test	PartitionCount: 4	ReplicationFactor: 1	Configs: min.insync.replicas=2,segment.bytes=1073741824
	Topic: test	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 1	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 2	Leader: 0	Replicas: 0	Isr: 0
	Topic: test	Partition: 3	Leader: 0	Replicas: 0	Isr: 0
```



브로커에 설정된 각종 기본값(`server.properties`) `--broker` , `--all` , `--describe` 옵션을 사용하여 조회할 수 있습니다.

```bash
$ bin/kafka-configs.sh --bootstrap-server my-kafka:9092
	--broker 0 
	--all
	--describe
```



## kafka-console-producer.sh

### 메시지 값 전송

토픽에 데이터를 넣을 수 있는 kafka-console-producer.sh 명령어를 활용합니다. 키보드로 문자를 작성하고 엔터 키를 누르면 별다른 응답없이 메시지 값이 전송됩니다.

```bash
$ bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka
```



### 메시지 키 전송

`key.separator` 를 선언하지 않으면 기본 설정은 Tab delimiter[₩t]이므로 `key.separator` 를 선언하지 않고 메시지를 보내기 위해서는 메시지 키를 작성하고 탭키를 누른 뒤 메시지 값을 작성하고 엔터를 누릅니다. 여기서는 명시적으로 확인하기 위해 콜론[:]을 구분자로 선언하였습니다.

```bash
$ bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --property "parse.key=true" --property "key.separator=:"
```

-  `property` 옵션을 설정하지 않고 메시지를 전송하면 key는 null로 전송이 됩니다.



메시지키가 있는 경우 : 키의 해시값을 작성하여 존재하는 파티션 중 한개에 할당됩니다. 이로 인해 메시지키가 동일한 경우에는 동일한 파티션으로 전송합니다.

메시지키가 null인 경우 : 프로듀서가 파티션으로 전송할 때 레코드 배치 단위(레코드 전송 묶음)로 라운드 로빈을 전송합니다. 



## kafka-console-consumer.sh

토픽으로 전송한 데이터는 kafka-console-consumer.sh 명령어로 확인이 가능합니다. 필수 옵션으로는 `--bootstrap-server` 에 카프카 클러스터 정보, `--topic` 에 토픽 이름이 필요합니다. 추가로 `--from-beginning` 옵션을 주면 토픽에 저장된 가장 처음 데이터부터 출력합니다.

```
$ bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --from-beginning
```



레코드의 메시지 키와 메시지 값을 확인하고 싶다면  `--property` 옵션을 사용하면 됩니다.

```
$ bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --property print.key=true --property key.separator="-" --from-beginning
```



`--max-messages` 옵션을 사용하여서 최대 Consume 메시지 개수를 설정할 수 있습니다.

```
$ bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --from-beginning --max-messages 1
```



`--partition` 옵션을 사용하면 특정 파티션만 Consume 할 수 있습니다.

```
$ bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --partition 2 --from-beginning
```



`--group` 옵션을 사용하면 `컨슈머 그룹` 을 기반으로 kafka-console-consumer가 동작합니다. 

#### 컨슈머 그룹이란?

- 특정 목적을 가진 컨슈머들을 묶음으로 사용하는 것을 뜻합니다. 
- 컨슈머 그룹으로 토픽의 레코드를 가져갈 경우 어느 레코드까지 읽었는지에 대한 데이터가 카프카 브로커에 저장됩니다. 
- 특정 오프셋까지 데이터를 읽었다는 것을 커밋시키기 위한 용도로 활용합니다. 
- 이 컨슈머그룹이 없으면 커밋이 안된다고 보면 됩니다.

```
$ bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --group hello-group --from-beginning
```

```
hello
9
123
234
1
2
5
6
hi
345
asd
kafka
3
4
7
8
10
lsh
```

- `lsh` 까지 읽었다는 것을 기록합니다. 

그럼, 어디다가 기록을 할까요? `offset` 이라고 하는 곳에 기록을 하게 됩니다. 

토픽 리스트를 확인할 수 있는 명령어를 쳐보면 다음과 같이 `__consumer_offsets` 이 생긴 것을 확인할 수 있습니다.

```
bin/kafka-topics.sh --bootstrap-server my-kafka:9092 --list
```

```
__consumer_offsets
hello.kafka
hello.kafka2
test
```



## kafka-consumer-groups.sh

컨슈머 그룹의 목록을 확인할 수 있는 명령어는 다음과 같습니다. 위에서 `hello-group` 이라는 컨슈머 그룹을 생성하였기 때문에 하나의 목록만 나타나게 됩니다.

```
bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 -list
```

컨슈머 그룹인 `hello-group` 은 컨슈머인 `hello.kafka`  토픽의 데이터를 가져갑니다. 컨슈머 그룹은 따로 생성하는 명령을 날리지 않고 컨슈머를 동작할 때 컨슈머 그룹 이름을 지정하면 새로 생성됩니다. 생성된 컨슈머 그룹의 리스트는 kafka-consumer-groups.sh 명령어로 확인할 수 있습니다.



컨슈머 그룹의 상세 정보를 확인하기 위해서는 `--describe` 옵션을 통해서 확인할 수 있습니다.

```
$ bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 -group hello-group --describe
```

`--describe` 옵션을 사용하면 해당 컨슈머 그룹이 어떤 토픽을 대상으로 레코드를 가져갔는지 상태를 확인할 수 있습니다. 

- 파티션 번호
- 현재까지 가져간 레코드의 오프셋
- 파티션 마지막 레코드의 오프셋
- 컨슈머 랙
- 컨슈머 ID
- 호스트

```
Consumer group 'hello-group' has no active members.

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
hello-group     hello.kafka     2          4               4               0               -               -               -
hello-group     hello.kafka     1          6               6               0               -               -               -
hello-group     hello.kafka     0          8               8               0               -               -               -
```



### 오프셋 리셋

컨슈머그룹이 특정 지점까지 데이터를 읽고 처리를하였을때, 어떤 토픽에 대해서 이 파티션의 데이터를 다시 재처리를 하고 싶을때 사용할 수 있습니다.

```
bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 --group hello-group --topic hello.kafka --reset-offsets --to-earliest --execute
```



#### 오프셋 리셋 종류

- `--to-earliest` : 가장 처음 오프셋(작은번호)으로 리셋
- `--to-latest` : 가장 마지막 오프셋(큰 번호)으로 리셋
- `--to-current` : 현 시점 기준 오프셋으로 리셋
- `--to-datatime {YYYY-MM-DDTHH:mmSS.sss}` : 특정 일시로 오프셋 리셋(레코드 타임스탬프 기준)
- `--to-offset {long}` : 특정 오프셋으로 리셋
- `--shift-by {+/- long}` : 현재 컨슈머 오프셋에서 앞뒤로 옮겨서 리셋



### 컨슈머 그룹 실습

현재 컨슈머 그룹의 상세 정보를 확인해보도록 하겠습니다.

```
bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 -group hello-group --describe
```

컨슈머 그룹의 상세 정보 내용은 다음과 같습니다.

```
Consumer group 'hello-group' has no active members.

GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
hello-group     hello.kafka     2          4               4               0               -               -               -
hello-group     hello.kafka     1          6               6               0               -               -               -
hello-group     hello.kafka     0          8               8               0               -               -               -
lovethefeel@lovethefeel-H310H5-M7:~/dev_program/kafka_2.12-2.5.0$
```

- `PARTITION` 은 3개가 있으며, `CURRENT-OFFSET` 은 4, 6, 8 입니다.
- 현재 처리되지 않은 데이터는 없습니다.



컨슈머 그룹의 상세 정보 변화를 위해 메시지 전송 테스트를 해보도록 하겠습니다.

```
bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka
```

```
>1
>2
>3
>4
>5
>6
>8
>7
```



메시지 전송 후 현재 상태 확인을 해보도록 하겠습니다.

```
bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 -group hello-group --describe
```

```
GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
hello-group     hello.kafka     2          4               7               3               -               -               -
hello-group     hello.kafka     1          6               8               2               -               -               -
hello-group     hello.kafka     0          8               11              3               -               -               -
```

- `PARTITION` 은 3개가 있으며, `CURRENT-OFFSET` 은 4, 6, 8 입니다.
- `LOG-END-OFFSET`  은 처리되지 않은 데이터의 오프셋 길이이며, 7, 8, 11 입니다.
- `LAG` 는 현재 처리되지 않은 데이터의 양을 말하며 3, 2, 3 입니다.



처리되지 않은 데이터를 처리하기 위해 아래 명령어 수행하도록 하겠습니다.

```
bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --group hello-group
```



메시지 수신 후 현재 상태 확인을 해보도록 하겠습니다.

```
bin/kafka-consumer-groups.sh --bootstrap-server my-kafka:9092 -group hello-group --describe
```

```
GROUP           TOPIC           PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG             CONSUMER-ID     HOST            CLIENT-ID
hello-group     hello.kafka     2          7               7               0               -               -               -
hello-group     hello.kafka     1          8               8               0               -               -               -
hello-group     hello.kafka     0          11              11              0               -               -               -
```

- `PARTITION` 은 3개가 있으며, `CURRENT-OFFSET` 은 7, 8, 11 입니다.
- `LOG-END-OFFSET`  은 `CURRENT-OFFSET` 과 동일합니다.
- 현재 처리되지 않는 데이터는 없습니다.



## 그외 커맨드 라인 명령어

### kafka-producer-perf-test.sh

kafka-producer-perf-test.sh는 카프카 프로듀서로 퍼포먼스를 측정할 때 사용할 수 있습니다.

```
bin/kafka-producer-perf-test.sh --producer-props bootstrap.servers=my-kafka:9092 --topic hello.kafka --num-records 10 --throughput 1 --record-size 100 --print-metric
```



### kafka-consumer-perf-test.sh

kafka-consumer-perf-test.sh는 카프카 컨슈머로 퍼포먼스를 측정할 때 사용할 수 있습니다. 카프카 브로커와 컨슈머(여기서는 해당 스크립트를 돌리는 호스트)간의 네트워크를 체크할 때 사용할 수 있습니다.

```
bin/kafka-consumer-perf-test.sh --bootstrap.servers=my-kafka:9092 --topic hello.kafka --messages 10 --show-detailed-stats
```



### kafka-reassign-partitions.sh

카프카 클라이언트와 직접적으로 통신하는 리더 파티션으로 몰리는 상황에서 해당 명령어를 이용해서 파티션들을 분배할 수 있습니다.

kafka-reassign-partitions.sh 를 사용하면 리더 파티션과 팔로워 파티션이 위치를 변경할 수 있습니다. 카프카 브로커에서는 auto.leader.rebalance.enable 옵션이 있는데 이 옵션의 기본값은 true로써 클러스터 단위에서 리더 파티션을 자동 리밸런싱하도록 도와줍니다. 브로커의 백그라운드 스레드가 일정한 가격으로 리더의 위치를 파악하고 필요시 리더 리밸런싱을 통해 리더의 위ㅏ치가 알맞게 분배됩니다.

```
cat partition.json
{
	"partitions": [{
		"topic": "hello.kafka", "partition": 0, "replicas": [0]
	}]
}
```

```
bin/kafka-reassign-partitions.sh --zookeeper my-kafka;2182 --reassignment-json-file partition.json --execute
```



### kafka-delete-record.sh

설정한 오프셋 번호를 포함한 이전 데이터를 지울 수 있습니다.

```
cat delete.json
{
	"partitions": [{
		"topic": "hello.kafka", "partition": 0, "offset": 5
	}], "version": 1
}
```

```
bin/kafka-delete-record.sh --bootstrap-server my-kafka:9092 --offset-json-file delete.json
```

- `offset` 인 5인 데이터만 지우는 것이 아닌 처음부터 5까지의 데이터를 지우게 됩니다.



### kafka-dump-log.sh

실제 브로커를 운영 및 관리하는 조직에서 사용하는 명령어로, 개발자가 사용해야하는 경우는 거의 없습니다. 해당 명령어를 이용해서 데이가 저장된 위치에서 *.log 파일의 오프셋의 상세한 정보등을 확인할 수 있습니다.

아래 경로에서 파일명 및 위치를 확인합니다.

```
ls data/hello.kafka-0
00000000000000000000.index  00000000000000000000.log  00000000000000000000.timeindex  leader-epoch-checkpoint
```

`kafka-dump-log.sh` 명령어를 이용하여 오프셋과 관련한 상세 정보를 확인할 수 있습니다.

```
bin/kafka-dump-log.sh --files data/hello.kafka-0/00000000000000000000.log --deep-iteration
```

```
Dumping data/hello.kafka-0/00000000000000000000.log
Starting offset: 0
baseOffset: 0 lastOffset: 0 count: 1 baseSequence: -1 lastSequence: -1 producerId: -1 producerEpoch: -1 partitionLeaderEpoch: 0 isTransactional: false isControl: false position: 0 CreateTime: 1659753405920 size: 71 magic: 2 compresscodec: NONE crc: 393831435 isvalid: true
| offset: 0 CreateTime: 1659753405920 keysize: -1 valuesize: 3 sequence: -1 headerKeys: []
...
```



## 기타

### 토픽을 생성하는 두가지 방법

1. 카프카 컨슈머 또는 프로듀서가 카프카 브로커에 생성되지 않은 토픽에 대해 데이터를 요청할 때
2. 커맨드 라인 툴로 명시적으로 토픽을 생성하는 방법



토픽을 효과적으로 유지보수하기 위해서는 토픽을 명시적으로 생성하는 것을 추천합니다. 토픽마다 처리되어야 하는 데이터의 특성이 다르기 때문입니다. 토픽을 생성할 때는 데이터 특성에 따라 옵션을 다르게 설정할 수 있습니다. 



### 카프카 브로커와 로컬 커맨드 라인 툴 버전을 맞춰야 하는 이유

카프카 브로커로 커맨드 라인 툴 명령을 내릴 때 브로커의 버전과 커맨드 라인 툴 버전을 반듯기 맞춰서 사용하는 것을 권장합니다. 브로커의 버전이 업그레이드 됨에 따라 커맨드 라인 툴의 상세 옵션이 달라지기 때문에 버전 차이로 인해 명령이 정상적으로 실행되지 않을 수 있습니다. 



## 정리

- 토픽 생성은 카프카 클러스터 정보와 토픽 이름이 필수값이며 그외 정보는 브로커에 설정된 기본값으로 생성합니다.
  - 기본값을 변경하기 위해서는 토픽 생성시 옵션 정보를 설정하면 됩니다.
- 기 생성된 토픽의 대한 파티션 개수를 늘리기 위한 방법은 존재하나 줄일 수는 없습니다. 
  - 줄여야 한다면 새로운 토픽을 생성하는 방법이 있습니다.
- 토픽에 데이터 전송시 메시지만 또는 메시지 키-속성 형식으로 전송할 수 있습니다.
- 컨슈머그룹을 통해 현재 처리되지 않은 데이터의 양을 확인할 수 있습니다.
- 컨슈머그룹의 오프셋 리셋을 통해 파티션의 데이터를 재처리할 수 있습니다.
- 토픽을 생성하는 두 가지 방법 중 토픽을 명시적으로 생성하는 것을 추천합니다. 토픽마다 처리되어야 하는 데이터의 특성이 다르기 때문에 옵션을 다르게 설정해야 합니다.



## 참고

- [데브원영님의 아파치 카프카 강의](https://www.inflearn.com/course/%EC%95%84%ED%8C%8C%EC%B9%98-%EC%B9%B4%ED%94%84%EC%B9%B4-%EC%95%A0%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)
