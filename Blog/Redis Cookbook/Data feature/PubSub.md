
`Publish-Subscribe` 는  `Wikipedia` 에 따르면 $1987$ 년에서 부터  긴 역사를 가진 `message` 패턴이다.
`PubSub`  은 간단하게 `evnet` 를 발행할 `publisher` 는 `PubSub` 채널에 `message` 를 보내고, 
이 `PubSub` 채널에 관심있는 각 `subscriber` 로 `event` 를 전달받는 구조이다.

`Kafka` 혹은 `ZeroMQ` 같은 많은 대중적인 `message` 미들웨어들은 이러한 패턴을 활용하여 `message` 전달 시스템을 구축한다.

이는 `Redis` 역시 이러한 `message` 전달 시스템을 제공하는데, 이를 살펴보도록 하자

---

