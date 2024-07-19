>[!info] `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`Redis` 에서 `value` 에 대한 `key` 를 연결하고 싶을때 사용가능한 `command` 는 `SET` 이다. 
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

>[!info] `SET` 실행시 `EXIST` `command` 를 사용할수 있다.<br>하지만 편의상 `SETNX` 및 `SETXX` 방식의 `command` 를 같이 지원한다. 
```sh
127.0.0.1:6379> SET "Extream Pizza" "300 Broadway, New York, NY" NX
(nil)

127.0.0.1:6379> GET "Extream Pizza"
"300 Broadway, Washington, DC 20009"
```

