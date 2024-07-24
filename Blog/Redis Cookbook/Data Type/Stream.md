`Stream` 은 `Redis` 에서 새롭게 만들어진 자료 구조이다.
이는 `Pus/Sub` 처럼 `fire and forget` 방식이 아닌, `list` 에 저장했다가 `consumer` 의 수신여부를 구분지을수 있고, 모든 `counsumer` 에게 메시지를 전달할수 있다.

이러한 `stream` 은 대규모 메시징 데이터를 빠르게 처리할수 있도록 설계되었다.
`stream` 은 몇가지 특징을 갖는데 다음과 같다

- `stream` 은 `data structure` 이다.

- `stream` 은 `append-only list` 처럼 행동한다.<br>(`append-only list` 는 모든 내용을 `list` 로 기록함을 말한다.)

- `stream` 의 모든 항목은 `HASH` 이다.

- `stream` 의 모든 항목은 `unique ID` 를 갖는다.<br>모든 `unique ID` 는 `timestamp`(`miliseconds`) 와 `0` 부터 시작하는 `sequence-number` 로 이루어져있다.<br><br>예)<br>`id: 15778368000000-0`<br>`id: 15778368000000-1`<br>`id: 15774040000000-2`<br>
- `stream` 은 `ID` 기반으로한 `range queries` 를 지원한다.<br>`unique ID` 는 `timestamp` 로 이루어져있다고 했다. 이는 시간을 의미하므로, 시간순으로 범위 쿼리가 가능하다는 이야기다<br><br>`XRANGE message 1577836800000-0 1577844000000-0`<br><br>이는 `1577836800000-0` 에서 `1577844000000-0` 까지의 모든 `entires` `ID` 를 쿼리한다.

- `stream` 은 `Consumer groups` 이다.<br>**데이터를 여러 소비자에게 전달하는것을 `팬아웃`(`fan-out`) 이라 한다.**<br><br>`Redis` 에서 `stream`  은 여러 소비자가 `XREAD` 를 사용하면, `fan-out` 가능하다.<br><br>반면`counsumer` 들의 집합을 `consumer group` 이라 한다.<br><br>`XGROUP CREATE Email EmailServiceGroup $` <br>`XREADGROUP GROUP EmailServiceGroup emailService1 COUNT 1 STREAMS Email >`<br><br>`consumer group` 에 속한  `cousumer`들은 그룹내의 다른 `consumer` 가 읽지 않은 데이터만 읽는다.  <br><br>이는 독립적이지만, 동일한 동작을 수행하는 서버에서 데이터를 분산처리하는데 좋은 방식이다.

---

## Stream  DEL 및 TTL 설정

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

`XADD` 는 `stream` `key` 에 새로운 항목을 추가한다.

>[!info] [XADD](https://redis.io/docs/latest/commands/xadd/)
```sh
XADD key [NOMKSTREAM] [<MAXLEN | MINID> [= | ~] threshold
  [LIMIT count]] <* | id> field value [field value ...]
```

만약 이미 만들어진 `stream` `key` 가 없다면, `stream` `value` 와 함께 `key` 는 생성된다.

>[!info] [[HASH]] 에서 `key` 가 없을때, `redis-server` 가 알아서 생성해주는것과 같은 원리다. <br>`NOMKSTREAM` `option` 을 사용하면 `stream` `key` 생성을 `disabled` 할수도 있다고 한다.

`stream`  에 저장되는 각 요소는 `field-value pair` 로 구성되며, 이는 `user`  가 지정한 것과 동일한 순서로 저장된다.

>[!info] [[#XRANGE]] 나 [[#XREAD]] 가 `stream` 에서 읽을때, [[#XADD]] 에 추가된 동일한 순서로 `field-value` 를 반환하도록 보장한다.

>[!info] [[#XADD]] 는 오직 `stream`  `data` 를 추가하는 `command` 이며, 삭제시 `XDEL` 혹은 `XTRIM` 으로 `stream` `data` 를 삭제할수있다.

>[!warning] `XDEL` 혹은 `XTRIM` 은 `stream` `data` 를 삭제하는것이지, `stream` 삭제를 하지 않는다. `stream` 자체를 삭제하길 원한다면 [[#Stream DEL 및 TTL 설정]] 로 처리해야 한다.

- **key**: `stream` 의 `key`<br>`stream` 은 `Redis 5.0` 이후 생성된 자료구조로, 다른 자료구조들처럼 `key` 값과 매핑된다.

- **NOMKSTREAM**: 만약, `stream` `key` 가 아직 존재하지 않은 `stream`  이라면, `Redis` 는 [[HASH]] 처럼 `value` 생성시 `key`  를 자동 생성한다.<br><br>이러한 동작을 `disabled` 하고 싶다면, `NOMKSTREAM` 옵션을 사용한다.

- **<MAXLEN | MINID> [= | ~] threshold [LIMIT count]]** : [[#XADD]] 의 이 구문은 [[#XTRIM]] `command` 와 동일한 의미로써 사용된다.<br><br>👉 **MAXLEN**: `stream` 의 `length` 가 지정한 `threshold` 를 초과하면 요소를 제거한다.<br>`threshold` 는 `positive integer` 이다.<br><br>👉 **MINID**: `stream` 의 `length` 가 지정한 `threshold` 보다 작은 `ID` 라면 요소를 제거한다.<br>`threshold` 는 `stream ID` 이다.<br><br>👉 **`~`(Nearly exect trimming)**: `trimming` 은 `Redis` 서버의 메모리 관리 기능중 하나로, 이는 `stream` 의 크기를 제한하고, 오래된 항목을 제거하는데 사용된다.<br><br>이러한 `trimming` 은 $2$ 가지의 방식으로 나뉜다.<br>`exact` 과 `Nearly exact` 이다.<br>`Nearly exact` 라고 하는데 이 용어는 정확하지만, 완벽하지 않는것을 의미한다.<br><br>이는 `stream` 의 크기를 특정 제한 내로 유지하는데 사용된다.<br>이는 정확한 항목을 유지하려고 시도하지만, 항상 정확하지는 않다.<br>하지만, `exact` 보다는 일찍 `trimming` 이 중지된다.<br>(이는 정확하게 자르는데 더 많은 리소스를 낭비하는 동작을 하기에 이러는가 싶다.)<br><br>이로 인해 `trimming` 이 훨씬 더 효율적이 되지만, 정확하게 딱 맞는 숫자의 `threshold` 값을 유지하지는 않는다.  <br><br> 정리하자면 다음과 같다.<br> <br>:LiDot: `=`(`exect trimming`) 는 정확하지만, 대규모의 `stream` 에는 성능비용이 발생할수 있다.<br>:LiDot: `~`(`Nearly exect trimming`) 은 약간의 오차는 허용하지만, 더 효율적으로 `trimming` 가능하다.<br><br>Default 는 `Exect trimming` 이다.<br><br>👉 **LIMIT** : 이는 `trimming` 과 연관이 있는 명령이다.<br><br> 🔥 `LIMIT` 은 `~`(`nealry exact trimming`) 에서 더 의미가 있으며, `=`(`exact trimming`) 에서 모든 초과 항목을 제거해야 하므로, `LIMIT` 의 효과가 제한적이다.<br><br>기본적으로 `trimming` 을 사용하면, `stream` 의 개수를 `trimming` 에 명시한 크기 만큼 제한한다.<br> <br> 만약, `trimming` 의 개수를 초과한다면, 초과한 만큼의 오래된 `data` 를 알아서 삭제한다.<br> 만약 대규모의 `stream` 이 발생하여 많은 수의 `data` 가 작성되고, `trimming` 유지 수를 초과했다고 하자.<br><br>이러한 경우 초과된 개수 만큼 오래된 데이터를 제거해야 한다.<br>이는 당연 초과된 개수가 많을수록 성능에 악영향을 미칠수 있다.<br>실제로 `LIMIT` 을 지정하지 않으면, 초과된 개수만큼 한번에 제거한다.<br><br>`LIMIT` 은 `trimming` 으로 인한 데이터 제거시, 저정한 `LIMIT` 값 만큼의 항목만 삭제하고 나머지는 다음 `XADD` 작업시 처리하도록 한다.

- **<* | id >(Specifing a Stream ID as an argument)**: `Stream` 은 주어진 항목마다 `ID` 식별자를 갖는다.<br><br>[[#XADD]]  는 `ID` 인자가 `*` 문자로 지정되어 있다면, 자동적으로 `unique ID` 를 생성한다.<br>하지만 자주 사용은 하지 않지만, 사용자 지정 `ID` 를 생성할수도 있다.<br>(보통 `SQL` 의 `ID` 와 일치시키려는 경우 많이 사용한다.)<br><br>`ID` 는 `1526919030474-55` 처럼 $2$ 개의 `-` 로 구분된 숫자로 이루어져 있다.<br>이는 두 구분된 숫자는 $64bit$ 만큼 지정할수 있다.(거의 무한하다..)<br><br>자동적으로 생성되는 `unique ID` 는 `<timestamp>-<squance number>` 형식으로 저장된다.<br>이러한 자동 생성방식을 `*` 문자를 사용하여 지정할수 있는데 이는 다음처럼 역시 가능하다.<br><br>`> XADD mystream 1526919030474-55 message "Hello,"`<br>`"1526919030474-55"`<br><br>`> XADD mystream 1526919030474-* message " World!"`<br>`"1526919030474-56"`<br><br>`*` 는 항상 자동 증분되도록 보장한다.<br><br>만약 `XADD` 에 명시적 `ID` 를 지정하는 경우에는 최소 유효 `ID` 는 `0-1` 이며, 사용자는 현재 `stream`  내부의 다른 `ID` 보다 큰 `ID` 를 지정해야 만 한다. 

















 




