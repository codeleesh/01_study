# [Redis]  개념 정리 및 설치

새로 이직한 회사에서 Redis 의 활용을 굉장히 많이 하고 있습니다. Redis 에 대해서 공부를 하기 위해 관련 내용을 정리중에 있습니다.



## redis란?

- Remote Dictionary Server
- Main Memory 에 저장해서 쉽고 빠르게 데이터를 접근하기 위하 개념
- Database 보다 더 빠른 Memory이며, 더 자주 접근하고 덜 자주 바뀌는 데이터를 저장



이외 에도 다음과 같은 특징들을 갖고 있습니다.

- Database, Cache, Message broker
- In-memory Data Structure Store 
- Open Source(BSD 3 License)
- Support data structures

  - Strings, set, sorted-set, hashes, list
  - Hyperloglog, bitmap, geospatial index
  - Stream




비슷한 종류로는 MemCache 가 있습니다. Redis에 알아가면서 자주 접하는 키워드가 `Cache` 입니다. `Cache` 에 대해서도 한번 접근해보도록 하겠습니다.



## Cache란?

- 나중의 요청에 대한 결과를 미리 저장했다가 빠르게 서비스 해주는 것을 의미
  - ex) Factorial 계산



### CPU Cache

- CPU Register > CPU Cache > Main Memory (DRAM) > Storage (SSD, HDD)



### Cache 전략

Cache 전략은 크게 2가지 방식이 있습니다.

#### Look aside Cache

Memory Cache를 이용하게 되면 훨씬 더 빠른 속도로 서비스가 가능합니다.

- 웹 서버는 데이터가 존재하는지 Cache를 먼저 확인
- Cache에 데이터가 있으면 Cache에서 확인
- Cache에 데이터가 없다면 DB에서 조회
- DB에서 얻어온 데이터를 Cache에 다시 저장



#### Write Back

Memory Cache를 이용하게 되면 훨씬 더 빠른 속도로 서비스가 가능합니다.

쓰기가 굉장히 빈번한 경우에 사용 - 로그를 데이터베이스의 저장하는 경우

- 웹 서버는 모든 데이터를 Cache에만 저장
- Cache에 특정 시간동안의 데이터가 저장
- Cache에 있는 데이터를 DB에 저장
- DB에 저장된 데이터를 삭제



이러한 `Cache` 의 구조를 통해서 다음과 같은 특징들을 갖고 있습니다.



## 특징

- Redis 활용처	
  - Remote Data Store
    - A서버, B서버, C서버에서 데이터를 공유하고 싶을때
  - 한대에서만 필요한다면, 전역 변수를 쓰면 되지 않을까?
    - Redis 자체가 Atomic을 보장해준다. (싱글 쓰레드)
  - 주로 많이 쓰는 곳들
    - 인증 토큰 등을 저장(Strings 또는 Hash)
    - Ranking 보드로 사용(Sorted Set)
    - 유저 API Limit
    - 잡 큐(List)
- Redis는 기본적으로 Single Thread
- 단순한 Key-Value 구조
- Redis 자료구조는 Atomic Critical Section에 대한 동기화를 제공
- 서로 다른 Transaction Read/Write를 동기화



## 활용

- 좋아요 처리
- 사용자 세션관리
- 유저 인증 토큰 저장
- 메세지 큐잉
- 최근 검색 목록



## 장점

- 여러 서버에서 같은 데이터를 공유해야 하는 세션 정보를 활용할 수 있습니다.



## 단점

- In-memory 방식이기 때문에 장애 발생시 데이터 유실이 발생할 수 있습니다.



## 설치

Redis 설치는 공식 사이트에서 관련 정보를 찾아서 쉽게 설치할 수 있습니다. - [Redis 공식사이트](https://redis.io/download/)

Docker를 통해서 설치를 진행하였습니다. [Redis 공식 Docker hub](https://hub.docker.com/_/redis) 를 통해서 쉽게 설치할 수 있었습니다.

```yaml
# redis docker-compose

version: '3'

services:
  redis:
    container_name: redis
    image: redis:latest
    ports:
      - 6379:6379
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - /Users/lsh/Desktop/docker_home/redis/data/:/data
      - /Users/lsh/Desktop/docker_home/redis/conf/redis.conf:/usr/local/etc/redis/redis.conf
      - /Users/lsh/Desktop/docker_home/redis/acl/users.acl:/etc/redis/users.acl
```



## 자료 구조

### String

- key와 value가 일 대 일 관계입니다.
- key와 value 모두 최대 길이는 512MB입니다.
- key를 구성할때 단어 사이에 구분자를 사용하는 것이 좋습니다.
- 예를 들어 `_` 등을 사용해서 key를 구성하면 쉽게 알아볼 수 있습니다.
  특히, Enterprise 버전에서 SQL(select) 사용을 고려한다면 다른 구분자보다 `_` 사용을 추천합니다.

### List

- Lists는 key와 value가 일 대 다 관계입니다.
- value는 입력된 순서대로 저장됩니다.
- value가 저장되면 키(리스트)는 생성됩니다. 키에 value가 하나도 없으면 키는 삭제됩니다.
  즉, 키(리스트)의 생성과 삭제를 위한 별도의 작업은 필요없습니다.

### Set

- Sets는 key와 value가 일 대 다 관계입니다.
- Value는 입력된 순서와 상관없이 저장되며, 중복되지 않습니다. 즉, value A가 2번 저장되도 결과적으로 하나만 남습니다.

### Sorted Set

- key 하나에 여러개의 score와 value로 구성됩니다.
- Value는 score로 sort되며 중복되지 않습니다.
- score가 같으면 value로 sort됩니다.

### Hash

- key 하나에 여러개의 field와 value로 구성됩니다.

- key 하나에 field와 value 쌍을 40억개(4,294,967,295)까지 저장 가능합니다.
- Hashes가 field와 value로 구성된다는 면에서 RDB의 table과 비슷합니다.
- Hash key는 table의 PK, field는 column, value는 value로 보면 됩니다.
- Table의 column 수는 일반적으로 제한이 있는 반면, Hash의 field 수는 40억개로 거의 무제한에 가깝습니다.
- Table에서 column을 추가하려면 alter문으로 미리 table을 변경해야 하나, Hash에서는 그런 사전 작업이 필요없습니다. 따라서 field의 추가/삭제는 자유롭습니다. Field의 추가/삭제는 해당 key에만 영향을 미칩니다.



## 정리

- Database 보다 더 빠른 Memory이며, 더 자주 접근하고 덜 자주 바뀌는 데이터를 저장합니다.
  - 세션 및 토큰, 최근 검색 목록 관리에 활용할 수 있습니다.
- Redis는 String, List, Sorted Set, Hash, Stream 등의 자료구조를 지원합니다.



## 참고

- [Redis - Documentation](https://redis.io/docs/)
- [[우아한테크세미나] 191121 우아한레디스 by 강대명님](https://youtu.be/mPB2CZiAkKM)
- [Redisgate](http://redisgate.kr/redisgate/ent/ent_intro.php)

