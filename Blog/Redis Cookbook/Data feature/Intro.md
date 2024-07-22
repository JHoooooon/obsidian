
레디스는 여러 유용한 데이터 기능을 함께 제공한다.
이는 다음처럼 이해할수 있다.

- **BItmaps**: `bitmaps` 를 어떻게 다룰수 있는지 보여준다.<br>메모리 공간을 절약하기 위해 문자열 대신 사용한다.

- **Expiration**: 레디스는 `In-Memory data store` 이다.<br>이는 일반적으로 `cache` 하기 위해 사용한다.<br>`Expiration` 기능은 이러한 `cache` 에 만료시간을 주는 기능이다.<br>`Expire Key` 를 사용한다면 어떠한 일이 발생할지 다음은 보여준다.

- **Sorting**: `Sorting` 은 `Redis`의 `LIST`, `SET`, `Sorted SET` 검색할때 정렬하는데 사용한다<br>이는 `SORT` 명령어를 사용하여 처리하는것을 볼수 있다.

- **Pipeline** : `Reids` 에서 `pipeline` 을 어떻게 사용하는지 보여준다.<br>`pipeline` 은 여러 `Redis` 작업의 퍼포먼스를 최적화 시켜주는 최고의 기능이다.

- **PUBSUB**: `Reids` 는 `message 교환 채널` 을 사용할수 있다.<br>`Redis` 의 `Publish/Subscribe` 기능을 위한 명령어와 어플리케이션에서 어떻게 사용하는지를 살펴본다.

- **Wriing/Debugging Lua in Redis**: `Lua` 는 여러 어플리케이션에 내장된 `scripting langauge` 이다. <br>`Lua` 는 `Redis` 에서 여러 작업들을 `bundle` 로 묶고, 원자적으로 실행할수 있도록 보장한다.

