`Stream` 은 `Redis` 에서 새롭게 만들어진 자료 구조이다.
이는 `Pus/Sub` 처럼 `fire and forget` 방식이 아닌, `list` 에 저장했다가 `consumer` 의 수신여부를 구분지을수 있고, 모든 `counsumer` 에게 메시지를 전달할수 있다.

이러한 `stream` 은 대규모 메시징 데이터를 빠르게 처리할수 있도록 설계되었다.
`stream` 은 몇가지 특징을 갖는데 다음과 같다

- `stream` 은 `data structure` 이다.

- `stream` 은 `append-only list` 처럼 행동한다.<br>(`append-only list` 는 모든 내용을 `list` 로 기록함을 말한다.)

- `stream` 의 모든 항목은 `HASH` 이다.

- `stream` 의 모든 항목은 `unique ID` 를 갖는다.<br>모든 `unique ID` 는 `timestamp`(`miliseconds`) 와 `0` 부터 시작하는 `sequence-number` 로 이루어져있다.<br><br>예)<br>`id: 15778368000000-0`<br>`id: 15778368000000-1`<br>`id: 15774040000000-2`<br>
- `stream` 은 `ID` 기반으로한 `range queries` 를 지원한다.<br>`unique ID` 는 `timestamp` 로 이루어져있다고 했다. 이는 시간을 의미하므로, 시간순으로 범위 쿼리가 가능하다는 이야기다<br><br>`XRANGE message 1577836800000-0 1577844000000-0`<br><br>이는 `1577836800000-0` 에서 `1577844000000-0` 까지의 모든 `entires` `ID` 를 쿼리한다.

- `stream` 은 `Consumer groups` 이다.<br>**데이터를 여러 소비자에게 전달하는것을 `팬아웃`(`fan-out`) 이라 한다.**<br><br>`Redis` 에서 `stream`  은 여러 소비자가 `XREAD` 를 사용하면, `fan-out` 가능하다.<br><br>반면`counsumer` 들의 집합을 `consumer group` 이라 한다.<br><br>`XGROUP CREATE Email EmailServiceGroup $` <br>`XREADGROUP GROUP EmailServiceGroup emailService1 COUNT 1 STREAMS Email >`<br><br>`consumer group` 에 속한  `cousumer`들은 그룹내의 한 `consumer` 가 읽지 않은 데이터만 읽는다.  

---

## `Stream`  `DEL` 및 `TTL` 설정

`DEL` 을 통해 `events` `stream` 을 삭제한다.

>[!info] `events` 라는 `Stream` 이 존재시 삭제
```sh
127.0.0.1:6379> DEL events
```

다음은 `EXPIRE` 을 사용하여 `86400` 초를 만료기간으로 설정한다.

>[!info] `messages` 라는 `Stream`  에 만료기간 설정
```sh
127.0.0.1:6379> EXPIRE messages 86400
```


## XADD

`XADD` 는 `stream` 에 새로운 항목을 추가한다.

>[!info] [XADD](https://redis.io/docs/latest/commands/xadd/)
```sh
XADD key [NOMKSTREAM] [<MAXLEN | MINID> [= | ~] threshold
  [LIMIT count]] <* | id> field value [field value ...]
```



