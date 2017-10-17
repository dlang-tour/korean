# 데이터베이스 연계(Database connectivity)

vibe.d 프레임워크는 서버 제작을 위해 다양한 데이터베이스를 쉽게 접속할 수 있게 해줍니다.

특히 **MongoDB** 와 **Redis** 는 별도 추가적 라이브러리 없이 접속할 수 있게 vibe.d와 함께 기본 제공됩니다.

다른 데이터베이스도 [code.dlang.org](https://code.dlang.org) 에서 지원하는 데이터베이스 어댑터를 설치하면 이용이 가능합니다.

### MongoDB

[`MongoClient`](http://vibed.org/api/vibe.db.mongo.client/MongoClient) 를 이용해 MongoDB를 이용합니다.

별도의 추가적 라이브러리가 필요하지 않으며, vibe.d의 비동기 I/O 방식을 소켓 통신에 사용합니다.

```d
    auto client = connectMongoDB("127.0.0.1");
    auto users = client.getCollection("users");
    users.insert(Bson("peter"));
```

### Redis

Redis도 MongoDB 클라이언트처럼 vibe.d 비동기 I/O 방식을 소켓 통신에 사용하며, 별도의 라이브러리가 필요하지 않습니다.

Redis 서버에 연결하고, 서버에 명령을 내릴 수 있는 주된 구현은 [`RedisDatabase`](http://vibed.org/api/vibe.db.redis.redis/RedisDatabase) 내에 있습니다. [`RedisList`](http://vibed.org/api/vibe.db.redis.types/RedisList) 처럼, Redis 상에 저장된 리스트를 간편하게 다룰 수 있게 `RedisDatabase` 사용을 보조해주는 래퍼(wrapper)도 있습니다.

### MySQL

MySQL과의 연계는 순수하게 D 언어로 구성된 코드로 동작하는 [mysql-native](http://code.dlang.org/packages/mysql-native) 를 사용합니다. `mysql-native` 설치는 `dub` 을 이용하면 되며, 별도로 OS에 추가적 라이브러리 설치는 필요하지 않습니다.

역시 vibe.d의 비동기 I/O 소켓 통신을 지원합니다.

### Postgresql

반드시 Postgresql의 모든 기능을 활용해야한다면
A full-featured Postgresql client is implemented *libpq* 외부 라이브러리를 사용해 동작하는 [dpq2](http://code.dlang.org/packages/dpq2) 모듈을 사용하면 됩니다. 비동기식처럼 동작하기 위해, vibe.d의 이벤트 처리 구조를 활용합니다.

만약 외부 라이브러리 설치가 망설여진다면, [ddb](http://code.dlang.org/packages/ddb) 와 같은 순수한 vibe.d로 동작하는 구현을 선택할 수 있습니다. 이것의 경우에는 `dpq2` 와는 달리 별도 추가적 라이브러리 설치가 필요하지 않습니다.
