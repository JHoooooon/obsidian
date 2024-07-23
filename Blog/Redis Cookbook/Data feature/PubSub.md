
`Publish-Subscribe` 는  `Wikipedia` 에 따르면 $1987$ 년에서 부터  긴 역사를 가진 `message` 패턴이다.
`PubSub`  은 간단하게 `evnet` 를 발행할 `publisher` 는 `PubSub` 채널에 `message` 를 보내고, 
이 `PubSub` 채널에 관심있는 각 `subscriber` 로 `event` 를 전달받는 구조이다.

`Kafka` 혹은 `ZeroMQ` 같은 많은 대중적인 `message` 미들웨어들은 이러한 패턴을 활용하여 `message` 전달 시스템을 구축한다.

이는 `Redis` 역시 이러한 `message` 전달 시스템을 제공하는데, 이를 살펴보도록 하자

`Channel` 의 `life cycle` 동안, `SUBSCRIBE`  는 `subscribed` 이전에 `channel` 이 없다면, 자동적으로 `Channel` 을 생성한다.

그리고 `Channel` 에 `subscriber` 가 활성화 되어 있지 않다면,`Channel` 은 그와 동시에 삭제된다.

여기서 중요한 부분이 있는데 `Redis` 의  `PusSub` 은 `message`, `channel`, `relationships` 가 `disk` 에 저장되지 않는다. 이는 휘발성이므로, `server` 가 종료되면 모든 개체는 사라진다.

>[!note] `fire and forget` 이라 한다. 즉 `불지르고(지르고) 잊는다.` 라고 해석해도 무방하다.<br>이는 `PusSub`  은 지속유지되지 않으며, 사라지는 휘발성임을 말한다.

또한 `message` 를 `publish` 하고 `Channel` 에 대한 구독자가 없다면, 이 `message` 는 버려진다.
이는, `메시지 전달 보장 매커니즘` (`mechanism to guarantee message delivery`) 가 없다는것과 같다

>[!info] `Kafka`, `RabbitMQ` 같은 경우, `message` 전달을 보장한다.

추가적으로  `Publisher` 에 의해 각 `message` 를 `channel` 에 넣으면, `message` 는 해당 `channel` 에 구독중인 `Redis` 의 `Subscribers` 가 받아 처리한다.

>[!warning] `PubSub` 은 중요한 메시지 전달 시나리오에는 적합하지 않다.<br>이는 `메시지 전달 보장 매커니즘` 이 없기때문이다.<br><br>**반면, 중요한 전달이 아닌 경우에는 빠른 속도로 메시지를 전달할수 있는 이점이 있다.**<br><br>예) `어플리케이션 내의 data 변경 notification`, `어플리케이션 내의 보조적 전달수단의 토스트`

---

다음은 `Subscriber` 들이 `restaurants:Chinese` 와 `restaurants:Thai` 를 구독한다.

`Subscriber1` 는 `restaurants:Chinese` 에 구독한다. 

>[!info] Subscriber 1
```sh
127.0.0.1:6379> SUBSCRIBE restaurants:Chinese 
Reading messages... (press Ctrl-C to quit) 
1) "subscribe" 
2) "restaurants:Chinese" 
3) (integer) 1  
```

`Subscriber2` 는 `restaurants:Chinese`, `restaurants:Thai` 에 구독한다. 

>[!info] Subscriber 2
```sh
127.0.0.1:6379> SUBSCRIBE restaurants:Chinese restaurants:Thai 
Reading messages... (press Ctrl-C to quit) 
1) "subscribe" 
2) "restaurants:Chinese" 
3) (integer) 1 
1) "subscribe" 
2) "restaurants:Thai" 
3) (integer) 2
```

다음은 `Publish` `command` 를 통해 `restaurants:Chinese` 에 `message` `event` 를 게시한다

>[!info] PUBLISH
```sh
127.0.0.1:6379> PUBLISH restaurants:Chinese "Beijing roast duck discount tomorrow" 
(integer) 2 
```

이순간, `restaurants:Chinese`에 구독한 `Subscriber1` 과 `Subscriber2` 는 `message` `event` 를 받는다

>[!info] Subscriber1 와 Subscriber2
```sh
1) "message" 
2) "restaurants:Chinese" 
3) "Beijing roast duck discount tomorrow"
```

반면 `restaurants:Thai` 에서 `message` `event` 를 게시한다면, 어떻게 될까?

>[!info] PUBLISH
```sh
127.0.0.1:6379> PUBLISH restaurants:Thai "3$ for Tom yum soap in this weekend!" 
(integer) 1
```

다음처럼 `Subscriber2` 만 `message` `event` 를 받는다.

>[!info] Subscriber2
```sh
1) "message" 
2) "restaurants:Thai" 
3) "3$ for Tom yum soap in this weekend!"
```

---

## `PubSub` 의 명령어들

### PSUBSCRIBE

>[!info] [PSUBSCRIBE](https://redis.io/docs/latest/commands/psubscribe/) 
```sh
PSUBSCRIBE pattern [pattern ...]
```

```sh
PSUBSCRIBE mail-* 

Reading messages... (Press Ctrl-C to quit)
1) "psubscribe"
2) "mail-*"
3) (integer) 1
```

이는 `glob-style pattern` 으로 `subscribe` 할수 있다.
이때, `message`는 `pmessage` 타입으로 전달된다.

책에서는 `SUBSCRIBE mail-1` 과 `PSUBSCRIBE mail-*` 두개를 구독했다고 가정하면 다음처럼 `message` 를 보낸다고 한다.



>[!note] 이렇게 구분하는 이유는 일반 `subscribe` 와 `psubscribe` 를 구분하기 위한것으로 보인다.


### PMESSAGE




---
## Keyspace-notifications

[Keyspace-notifications](https://redis.io/docs/latest/develop/use/keyspace-notifications/) 는 `Redis` 데이터베이스 내의 이벤트를 실시간으로 모니터링하고 알림을 받을수 있게 해주는 기능이다.

이 기능을 통해 특정 키의 변경, 만료, 삭제 등의 이벤트를 감지하고 이에 반응할수 있다.

다음은, `event` 가 작동할 `Key space` 를 지정하는 유형이다.
이는 `event`  작동시 `space` 공간을 만들어, 구분지어 처리하는데 유용하다.

- **Key Events**: 키 조작 관련 이벤트 (`SET`, `DEL`, `EXPIRE`, ...)
- **Keyspace Events**: 특정 키에 대한 모든 이벤트

`Redis` 는 기본적으로 `notify-keyspace-envets` 가 비활성화 되어있으므로, 처리하기 위해서 `config` 를 사용해 설정해주어야 한다.

다음은 `Redis` 에서 `notify-keyspace-envets` 에 제공하는 `event` 유형의 `paramter` 들이다.

---
- **K**: `Keyspace` 이벤트로, `__keyspace@<db>__` 접두사와 함께 `publish` 된다

- **E**: `Keyevent` 이벤트로, `__keyevent@<db>__` 접두사와 함께 `publish` 된다

- **g**: `type` 없이 지정된 `command`로  `DEL`, `EXPIRE`, `RENAME`... 같은 일반적인 `command` 를 뜻한다

- **$**: `String` `command` 

- **l**: `List` `command`

- **s**: `Set` `command`

- **h**: `Hash` `command`

- **z**: `Sorted Set` `command`

- **t**: `Stream` `command`

- **d**: `Module` `key` 타입 이벤트

- **x**: `Expired` 이벤트 (`key` 가 만료될때마다 발생)

- **e**: `Evicted` 이벤트 <br><br>(`key` 가 `maxmemory` 설정에 도달하면 데이터 공간을 확보하기 위해 기존 키를 제거한다. 이때 발생하는 이벤트다)<br><br>`Evicted` 뜻은 `퇴거하다`, `내쫒다` 라는 의미를 가진다.

- **m**: `Key miss` 이벤트 (`key` 접근시 존재하지 않을때 발생)

- **n**: 새로운 `key` 생성시 발생하는 이벤트 (`A` 에는 포함되지 않는다)

- **A**:  **"g$lshztxed"** 에 대한 별칭이다. (이는 `m` 과 `n` 을 제외한 모든 `parameter` 를 포함한다.)

---

>[!info] Shell
```sh
$ redis-cli config set notify-keyspace-events KEA

$ redis-cli --csv psubscribe '__key*__:*'
Reading messages... (press Ctrl-C to quit)
"psubscribe","__key*__:*",1
```

>[!info] Javascript
```javascript
const redis = require('redis');

const subscriber = redis.createClient(); 

const publisher = redis.createClient(); 

// Keyspace notifications 활성화 
subscriber.config('SET', 'notify-keyspace-events', 'KEA'); 

// 특정 키의 만료 이벤트 구독 
subscriber.subscribe('__keyevent@0__:expired'); 

subscriber.on('message', (channel, message) => { 
	console.log(`키 "${message}"가 만료되었습니다.`); 
	// 여기에 만료 이벤트에 대한 처리 로직을 추가할 수 있습니다. 
}); 

// 테스트를 위한 키 설정 (10초 후 만료) 
publisher.set('testKey', 'someValue', 'EX', 10); 

console.log('10초 후 만료 이벤트를 기다리는 중...');
```

위는 `subscriber.config` 에 `notify-keyspace-events` 로 `KEA` 인자를 설정한다.
이는  `g$lshztxed` 와 `Keyspace`, `Keyevent` 를 활성화 한다고 `redis-server` 에 알려준다

>[!info] `Redis` 는 `default` 로 `notify-keyspace-events` 는 `disabled` 되어 있다.