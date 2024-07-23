
`Publish-Subscribe` 는  `Wikipedia` 에 따르면 $1987$ 년에서 부터  긴 역사를 가진 `message` 패턴이다.
`PubSub`  은 간단하게 `evnet` 를 발행할 `publisher` 는 `PubSub` 채널에 `message` 를 보내고, 
이 `PubSub` 채널에 관심있는 각 `subscriber` 로 `event` 를 전달받는 구조이다.

`Kafka` 혹은 `ZeroMQ` 같은 많은 대중적인 `message` 미들웨어들은 이러한 패턴을 활용하여 `message` 전달 시스템을 구축한다.

이는 `Redis` 역시 이러한 `message` 전달 시스템을 제공하는데, 이를 살펴보도록 하자

`Channel` 의 `life cycle` 동안, `SUBSCRIBE`  는 `subscribed` 이전에 `channel` 이 없다면, 자동적으로 `Channel` 을 생성한다.

그리고 `Channel` 에 `subscriber` 가 활성화 되어 있지 않다면,`Channel` 은 그와 동시에 삭제된다.

여기서 중요한 부분이 있는데 `Redis` 의  `PusSub` 은 `message`, `channel`, `relationships` 가 `disk` 에 저장되지 않는다. 이는 휘발성이므로, `server` 가 종료되면 모든 개체는 사라진다.






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

