>[!info] `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`Redis` 에서 `value` 에 대한 `key` 를 연결하고 싶을때 사용가능한 `command` 는 `SET` 이다. 
다음은 사용가능한 [SET Option](https://redis.io/docs/latest/commands/set/) 이다.

- **EX seconds**: 초단위 만기 시간을 설정한다. (양의 정수)

- **PX millisencods**: 밀리초 단위 만기 시간을 설정한다. (양의 정수)

- **EXAT timestampe-seconds**: 초 단위 `Unix time` 만기 시간을 설정한다. (양의 정수)

- **PXAT timestampe-milliseconds**: 밀리초 단위 `Unix time` 만기 시간을 설정한다. (양의 정수)

- **NX**:  존재하지 않은 `key` 라면, `key` 값을 `set` 한다. 

- **XX**:   존재하는 `key` 라면, `key` 값을 `set` 한다. 

- **KEEPTTL**: `key` 에 대한 `Time To Live` 로, 설정한 시간동안 `key` 를 유지한다.

- **GET**: `key` 에 이전 `string` 혹은 `key` 가 존재하지 않다면 `nil` 을 반환한다.<br>`key` 에 저장된 값이 `string` 이 아닌경우 오류가 반환되고 `SET` 이 중단된다. 

>[!warning] `GET option` 에서 `string` 값이 아닌경우 오류가 발생한다고 말했다.<br>이는 `Redis` 특성상 `Number` 값역시 문자열로 저장되는 특성이 있으므로, 일반적인 `scalar` 값은 `string` 이라고 생각한다<br><br> `string` 값이 아닌 경우는 `List`, `Set`, `Sorted Set`, `Hash`, `Bitmap`, `HyperLogLog`, `Stream` 등등의 다른 데이터 타입을 사용할수 없다는 이야기이다.<br><br>`SET` 은 기본적으로 문자열을 저장하는데 사용된다.

`String` 은 `Redis` 자체에서 어떻게 `encoding` 되어야 하는지 알 필요가 있다.
`Redis` 는 `3` 가지의 `encoding` 으로 `string` 객체를 사용하며, `string` 값마다 자동적으로 `encoding` 형식을 결정한다.

다음은 이러한 `Redis` 의 `encoding` 값들이다.

- **int**: 64-bit 부호있는 정수를 문자열로 표현
- **embstr**: 44 Byte 보자 작거나 같은 길이값을 가진 문자열
- **raw**: 44 Byte 보다 큰 길이를 가진 문자열

이를 확이하기 위해서는 다음처럼 `OBJECT` `command` 를 사용할수 있다.

```sh
127.0.0.1:6379> SET myKey 12345 
OK 
127.0.0.1:6379> OBJECT ENCODING myKey 
"int" 
127.0.0.1:6379> SET myKey "a string" 
OK 
127.0.0.1:6379> OBJECT ENCODING myKey 
"embstr" 
127.0.0.1:6379> SET myKey "a long string whose length is more than 39 bytes" 
OK 
127.0.0.1:6379> OBJECT ENCODING myKey 
"raw"
```

--- 

다음은 `name` `key` 와 `address` 값을 생성한다.

>[!info] SET
```sh
127.0.0.1:6379> SET "Extreme Pizza" "300 Broadway, New York, NY" 
OK
```

`GET` 을 통해 `Extreame Pizza` 의 `key` 값으로 `value` 를 가져온다

>[!info] GET
```sh
127.0.0.1:6379> GET "Extreme Pizza" 
"300 Broadway, New York, NY" 
```

>[!warning] 만약 `key` 값이 존재하지 않는다면 `nil` 을 반환한다
```sh
127.0.0.1:6379> GET "Yummy Pizza" 
(nil) 
```

`STRLEN` `command` 는 `value` `string` 의 `length` 값을 반환한다.

>[!info] STRLEN
```sh
127.0.0.1:6379> STRLEN "Extreme Pizza"
 (integer) 26
```

>[!warning] 만약 `key` 값이 존재하지 않는다면 `0` 을 반환한다
```sh
127.0.0.1:6379> STRLEN "This Pizza is Non-existent"
(integer) 0
```

`value` 값에 `string value` 을 추가한다 

>[!info] APPEND
```sh
127.0.0.1:6379> APPEND "Extreme Pizza" " 10011" 
(integer) 32 
127.0.0.1:6379> GET "Extreme Pizza" 
"300 Broadway, New York, NY 10011"
```

`SETRANGE` 는 `string value` 의 한부분을 덮어씌운다.

>[!info] SETRANGE
```sh
127.0.0.1:6379> SETRANGE "Extreme Pizza" 14 "Washington, DC 20009" 
(integer) 34 
127.0.0.1:6379> GET "Extreme Pizza" 
"300 Broadway, Washington, DC 20009" 
```

`SET` 을 사용할시 `key` 가 존재할때, `key` 가 존재하지 않을때 두가지 부분을 생각할 필요가 있다.
만약 `SET` 사용시 해당하는 `key` 가 존재한다면, 어떻게 처리할지 명시해주어야 한다.

`XX` 와 `NX` 라는 접두어가 존재하는 이유이다
`XX` 는 `Exist` 일때, `Value` 값을 쓰고, `NX` 는 `Not Exist` 일때 `Value` 값을 쓴다.

```sh
127.0.0.1:6379> SET "Extream Pizza" "300 Broadway, New York, NY" NX
(nil)

127.0.0.1:6379> GET "Extream Pizza"
"300 Broadway, Washington, DC 20009"

127.0.0.1:6379> SET "Extream Pizza" "300 Broadway, New York, NY" XX
OK

127.0.0.1:6379> GET "Extream Pizza"
"300 Broadway, New York, NY"
```

>[!info] `SET` 실행시 `EXIST` `command` 를 사용할수 있다.<br>하지만 편의상 `SETNX` 및 `SETXX` 방식의 `command` 를 같이 지원한다. 
```sh
127.0.0.1:6379> SETNX "Lobster Palace" "437 Main St, Chicago, IL" 
(integer) 1 
127.0.0.1:6379> SETNX "Extreme Pizza" "100 Broadway, New York, NY" 
(integer) 0
```

`MSET` 과 `MGET`  `command` 는 `GET` 및 `SET` 에서 여러  `key` 들을 한번에 설정 및 가져올수 있다.
`MSET` 은 전체 작업은 원자적으로 처리되며, 여러번 `SET` 및 `GET` 할 필요없이 한번에 `command` 를 입력할수 있는 장점을 가진다.

>[!info] 여러 명령어를 매번 입력하면, `client` 및 `server` 가 요청을 많이 보낼수록 좋은 방식은 아니다.<br> 가능하다면 한번의 `command` 로 요청을 보내고 한번에 응답을 받는것이 좋다.

>[!info] `MSET` 과 `MGET`
```sh
127.0.0.1:6379> MSET "Sakura Sushi" "123 Ellis St, Chicago, IL" "Green Curry Thai" "456 American Way, Seattle, WA" 
OK 

127.0.0.1:6379> MGET "Sakura Sushi" "Green Curry Thai" "nonexistent" 
1) "123 Ellis St, Chicago, IL" 
2) "456 American Way, Seattle, WA" 
3) (nil)
```

