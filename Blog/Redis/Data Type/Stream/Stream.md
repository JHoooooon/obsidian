
>[!info] `개발자를 위한 레디스` 의 내용과 `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`Stream` 은 `Redis` 에서 새롭게 만들어진 자료 구조이다.
이는 `Pus/Sub` 처럼 `fire and forget` 방식이 아닌, `list` 에 저장했다가 `consumer` 의 수신여부를 구분지을수 있고, 모든 `counsumer` 에게 메시지를 전달할수 있다.

>[!info] 이 `Stream`  에 대해서는, `micro service` 관련 해서도, 지금 현재 만들 프로젝트에서도 유용하게 사용될 개념으로, `DOCS` 의 내용도 같이 정리한다.
## Stream 의 특징

`stream` 은 대규모 메시징 데이터를 빠르게 처리할수 있도록 설계되었다.
`stream` 은 몇가지 특징을 갖는데 다음과 같다

- `stream` 은 `data structure` 이다.

- `stream` 은 `append-only list` 처럼 행동한다.<br>(`append-only list` 는 모든 내용을 `list` 로 기록함을 말한다.)

- `stream` 의 모든 항목은 `HASH` 이다.

- `stream` 의 모든 항목은 `unique ID` 를 갖는다.<br>모든 `unique ID` 는 `timestamp`(`miliseconds`) 와 `0` 부터 시작하는 `sequence-number` 로 이루어져있다.<br><br>예)<br>`id: 15778368000000-0`<br>`id: 15778368000000-1`<br>`id: 15774040000000-2`<br>
- `stream` 은 `ID` 기반으로한 `range queries` 를 지원한다.<br>`unique ID` 는 `timestamp` 로 이루어져있다고 했다. 이는 시간을 의미하므로, 시간순으로 범위 쿼리가 가능하다는 이야기다<br><br>`XRANGE message 1577836800000-0 1577844000000-0`<br><br>이는 `1577836800000-0` 에서 `1577844000000-0` 까지의 모든 `entires` `ID` 를 쿼리한다.

- `stream` 은 `Consumer groups` 이다.<br>**데이터를 여러 소비자에게 전달하는것을 `팬아웃`(`fan-out`) 이라 한다.**<br><br>`Redis` 에서 `stream`  은 여러 소비자가 `XREAD` 를 사용하면, `fan-out` 가능하다.<br><br>`counsumer` 들의 집합을 `consumer group` 이라 한다.<br><br>`XGROUP CREATE Email EmailServiceGroup $` <br>`XREADGROUP GROUP EmailServiceGroup emailService1 COUNT 1 STREAMS Email >`<br><br>`consumer group` 에 속한  `cousumer`들은 그룹내의 다른 `consumer` 가 읽지 않은 데이터만 읽는다.  <br><br>이는 독립적이지만, 동일한 동작을 수행하는 서버에서 데이터를 분산처리하는데 좋은 방식이다.

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

## XDEL

>[!info] [XDEL](https://redis.io/docs/latest/commands/xdel/)
```sh
XDEL key id [id ...]
```

이는 `stream` 의 `entries` 를 삭제하는 명령이다.
그리고, 반환값은 삭제된 `entries` 의 수이다.

일반적으로, [[#XDEL]] 을 사용하는 방식은 `stream` 의 항목을 실제 제거하는것이 아닌, 삭제될것을 표시하는 것이다.

`macro-node` 의 모든 항목이 삭제됨으로 표시된 경우에만, 전체 `node` 는 파괴되고, `memory` 에 회수된다.

만약, `stream` 으로 부터  $50\%$ 보다 더 많은  `entries` 를 추가한다고 가정하자.
그럼 대량의 `entries` 를 제거해야 한다.

이때 `stream` 의 단편화로 인해, `memory` 사용랑은 증가할 것이다.
그렇지만 `stream` 은 증가되지 않으며, 기존의 퍼포먼스를 유지한다. 

>[!info]  내가 이해하기로는 `macro-node` 의 모든 항목이 삭제됨으로 표시될때만 삭제하기 때문에, 대량의 `entries` 를 추가했다고 해서 처리되지 않는다고 하는듯하다.

## XREAD

>[!info] [XREAD](https://redis.io/docs/latest/commands/xread/)
```sh
XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] id
  [id ...]
```

여러 혹은 하나의 `streams` `data` 를 읽는다.
호출자에 의해 마지막으로 전달받은 `ID` 보다 큰 `ID` 에 대한 항목만 반환받는다. 

- **[COUNT count]**: `COUNT` 옵션은 `stream` 당 반환 요소개수를 반환한다.

- **[BLOCK miliseconds]**: `BLOCK` 은 `producer` 가 `data` 를 [[#XADD]] 할때까지 `miliseconds` 동안 대기한다.<br><br>`BLOCK` 은 고정된 `polling` 을 방지하거나, 내부적으로 `BLOCK` 할수 있도록 할 목적으로, 이 명령어는 지정된 `string ID` 와 `ID` 에 따라 반환할 데이터가 없을 경우 `BLOCKING` 될수 있으며, 요청된 `key` 중 하나가 데이터를 받으면 자동으로 `BLOCKING` 이 해제된다.<br><br>다음용어를 정리한다.<br><br>👉 **동기적형태**: 명령어가 실행되면 즉시 결과를 반환한다.<br>👉 **폴링 방지**: 새 데이터를 주기적으로 확인하는 대신, 데이터가 있을때까지 기다린다.<br>👉 **블로킹**: 데이터가 없을때 명령어 실행을 일시 중지한다.<br>👉 **자동 언 블로킹**: 새 데이터가 추가되면 명령어가 자동으로 재개된다.

- **STREAMS key [key ...] id [id...]**: `STREAM` 옵션은 필수이며 반드시 마지막 옵션이어야 한다.<br>이는 가변 길이의 인수를 처리하는 옵션이기 때문이다.<br><br>그래서 `key` 의 목록으로 시작하고, 이후에 연관된 `ID` 들의 목록을 받으며, `stream` 에 주어진 `ID` 보다 큰 값을 받도록 호출한다<br><br>👉 **Incomplate IDS**: 불완전한 `ID` 사용역시 유효한다.<br>이는 `ID` 의 `sequnace` 부분이 누락된 경우 항상 $0$ 으로 해석되기 때문이다.<br><br>`> XREAD COUNT 2 STREAMS mystream writers 0 0`<br><br>이는 다음과 같다<br><br> `> XREAD COUNT 2 STREAMS mystream writers 0-0 0-0`<br><br>👉 **The special `$` ID**: `Blocking` 할때, 때때로 `XADD` 를 통해 `stream` 의 항목이 추가될때만 수신하고 싶을때가 있다.<br><br>예를들어, 이미 추가된 항목의 `history` 에는 관심이 없는 상황이 있을수 있다.<br>이는 `stream` 의 최 상단 요소 `ID` 만을 `check` 하고, 해당 `ID` 만을 [[#XREAD]] 명령을 내려, 최 상단의 항목만을 가져올때의 경우이다.<br><br>이는 다른 `command` 를 호출하여 처리해야 하므로, 깔끔한 방법이라 할수 없다.<br>대신, `Redis` 에서는 이를 처리할수 있는 특별한 `$` `ID` 를 사용할수 있다.<br><br>`$` `ID` 는 스트림의 가장 최신 항목 다음을 가리키는 특수한 `ID` 이다.<br> `XREAD` 에서 처음 호출하는 `$` `ID` 를 이해하는것은 매우 중요하다.<br><br>1. **첫번째 `XREAD` 호출**: 처음 호출시 `$` 를 `ID` 로 사용하면 가장 최신의 데이터부터 읽기 시작한다.<br>2. **이후 `XREAD` 호출**: 첫번째 호출 이후에 `$` 를 계속 사용하면 안된다. 대신, 마지막으로 받은 항목의 `ID` 를 사용해야 한다.<br><br>`$` 는 항상 현재시점의 최신항목 다음을 가리키기 때문에, 이전 호출과 현재 호출사이에 추가된 모든 항목을 놓칠수 있기 때문이다.<br><br>`> XREAD BLOCK 5000 COUNT 100 STREAMS mystream $`<br><br>`> XREAD BLOCK 5000 COUNT 100 STREAMS mystream last_recived_id`<br><br>👉 **The special `+` ID**: `XREVRANGE` 명령을 사용하여 쉽게 하나의 `stream` 안의 마지막 항목을 읽을수 있다.<br><br>`> XREVRANGE stream + - COUNT 1`<br><br>그러나 이러한 접근은 각 `stream`  에 대해 별도의 명령을 실행해야 하기 때문에 더 많은 `stream` 을 추가하면 이 접근 방식이 느려진다.<br><br>대신, `Redis 7.4` 이후부터 특별한 `ID` 로써  `+` 을 사용할수 있다.<br>이는 `stream`  안에 존재하는 마지막 요소를 요청한다.<br><br>`> XREAD STREAM streamA streamB streamC streamD + + + +` <br><br>특별한 `ID` 를 사용할때, `COUNT` 옵션은 무시되며, 오직 마지막 항목만 반환될수 있다.

>[!info] XREAD STREAM
```sh
> XREAD COUNT 2 STREAMS mystream writers 1526999352406-0 1526985685298-0
1) 1) "mystream"
   2) 1) 1) 1526999626221-0
         2) 1) "duration"
            2) "911"
            3) "event-id"
            4) "7"
            5) "user-id"
            6) "9488232"
2) 1) "writers"
   2) 1) 1) 1526985691746-0
         2) 1) "name"
            2) "Toni"
            3) "surname"
            4) "Morrison"
      2) 1) 1526985712947-0
         2) 1) "name"
            2) "Agatha"
            3) "surname"
            4) "Christie"
```

## XRANGE 

`XRANGE` 는 `ID` 의 범위와 맞는 항목들을 반환한다.
이러한 `Range` 는 큰`ID`  와 작은 `ID` 로 정의된다.

두 지정된 `ID`  중 하나와 정확하게 맞는 항목 이거나 `2` 지정된 `ID` 사이의 항목들을 가진다
`XRANGE` 명령어는 다양한 용도가 있다.

- **지정된 시간 범위안의 `items` 를 반환**<br>이는 `stream ID` 가 시간과 관련있기 때문에 가능하다.<br>[[#Stream 의 특징]] 에서 `ID` 가 자동적으로 어떻게 생성되는지 알수있다. 

- **점진적으로 `stream` 반복** <br>매 반복마다 `item` 들을 분할해서 리턴한다.<br>그러나 일반적인 `SCAN` 계열의 함수들보다 훨씬 강력하다.

- **`stream` 에서 단일 항목을 가져오고, 쿼리 간격의 시작과 끝으로 가져올 항목의 `ID` 를 두번 제공한다.**<br><br>번역체라서 이해한 내용을 다시 풀어쓴다..  `SCAN` 종류의 함수를 실행시, 반복하면서, 값을 가져오는데 이때 쿼리 시작의 `ID` 에서 부터 모든 요소를 순환하여 끝까지 가게된다.<br><br>이때 단일항목의 `ID` 만 지정했으니, 끝에 있는 항목은 쿼리 시작의 `ID` 의 바로 전 `ID` 일 것이다.<br><br>이말은, 단일 항목을 `fatch` 할때, 시작과 끝은 구분하는 `ID`  는 현재 제공된 `ID` 라는 말이다.

다음은, `XRANGE` 의 명령어이다.

>[!info] [XRANGE](https://redis.io/docs/latest/commands/xrange/)
```sh
XRANGE key start end [COUNT count]
```

- **key** : `stream` 의 `key`
- **start** : `stream` 의 `key` 에서 찾을 범위의 시작
- **end** : `stream` 의 `key` 에서 찾을 범위의 마지막
- **[COUNT count]**: `XRANGE` 의 반복에서, 한번에 몇개의 항목을 가져올지 지정하는 옵션

### - and + special IDs

`XRANGE` 사용시 특별한 `ID` 들을 제공한다.

- **+**: 현재 `stream` 의 가장 큰 `ID`
- **-**: 현재 `stream` 의 가장 작은 `ID`

이를 통해 다음처럼 쿼리 가능하다.

```sh
> XRANGE somestream - +
1) 1) 1526985054069-0
   2) 1) "duration"
      2) "72"
      3) "event-id"
      4) "9"
      5) "user-id"
      6) "839248"
2) 1) 1526985069902-0
   2) 1) "duration"
      2) "415"
      3) "event-id"
      4) "2"
      5) "user-id"
      6) "772213"
... other entries here ...
```

### Incomplete IDs

[[#Stream 의 특징]] 에서 `ID` 는 `timestamp-sequnce number` 식으로 자동 생성될수있다고 말했다.

이때, `ID` 의 형식이 아닌, 불완전한 형식의 `ID`  를 사용해도 쿼리 가능하다.
이는 `timestamp` 자체가 `miliseconds` 로 이루어진 시간 값이기 때문이다.

```sh
> XRANGE somestream 1526985054069 1526985055069
```

이는 지정된 밀리초 사이의 모든 항목을 반환하기 위해, `0` 부터 시작 시퀀스를, `-18446744073709551615` 종료 시퀀스를 자동생성한다. 

만약 같은 밀리초를 두번 반복하면, 시퀀스 번호 범위가 `0` 에서 최대 값이기 때문에 해당 밀리초내의 모든 항목을 얻을수 있음을 의미한다

```sh
> XRANGE somestream 1526985054069 1526985054069 # 같은 밀리초
```

>[!info] `1526985054069-0` 과 `1526985054069-18446744073709551615` 이니 두개의 `milisecond` 가 반복되며, 해당 밀리초의 시작과 끝인 시퀀스가 생성된다.

### Exclusive ranges

`Inclusive range` 는 지정한 `ID` 를 범위에 포함한다. 
이는 다음과 같다

>[!info] Inclusive range
```sh
XRANGE mystream 1526985054069-0 1526985055069-0
```

`Exclusive range` 는 지정한 `ID` 를 범위에 포함하지 않는다. 
이를 표현하기 위한 특수기호인 `(` 를 사용한다.

>[!info] Exclusive range
```sh
XRANGE mystream (1526985054069-0 (1526985055069-0
```

이 명령은 ID가 1526985054069-0보다 크고 1526985055069-0보다 작은 항목만 반환한다. 즉, 시작과 끝 ID의 항목은 포함되지 않는다.
### Iterating a stream

`stream` 의 반복하기 위해, 각 항목당 $2$ 개의 요소를 원한다고 가정하자.
이를 위해 다음처럼 `fetching` 할수 있다.

```sh
> XRANGE writers - + COUNT 2
1) 1) 1526985676425-0
   2) 1) "name"
      2) "Virginia"
      3) "surname"
      4) "Woolf"
2) 1) 1526985685298-0
   2) 1) "name"
      2) "Jane"
      3) "surname"
      4) "Austen"
```

`-` 로 부터 다시 반복을 실행하는대신, `start` 범위를 리턴된 마지막 항목을 사용하여 범위지정 가능하다.

```sh
> XRANGE writers (1526985685298-0 + COUNT 2
1) 1) 1526985691746-0
   2) 1) "name"
      2) "Toni"
      3) "surname"
      4) "Morrison"
2) 1) 1526985712947-0
   2) 1) "name"
      2) "Agatha"
      3) "surname"
      4) "Christie"
```

이는 [[#Exclusive ranges]] 를 사용하여, 이전의 명령에서의 마지막 `ID` 값인 `1526985685298-0` 을 포함하지 않고, 그 다음의 $2$ 개의 항목을 리턴한다.

이러한 방식을 사용하여 `iteration` 가능하다.
이는 효과적으로 `stream`  의 항목을 순회할수 있다. 

>[!info] [[#Exclusive ranges]] 는 `Reids 6.2` 이후 추가된 `spec` 이다.<br>이전 버전에서 이처럼 사용하려면 다음처럼 사용할수 있다고 한다.

```sh
> XRANGE writers 1526985685298-1 + COUNT 2
1) 1) 1526985691746-0
   2) 1) "name"
      2) "Toni"
...
```

이는 `1526985685298-0` 이 이전 `XRANGE` 의 마지막 값이므로, 이를 제외한 값인 `1526985685298-1` 부터 항목을 찾아 처리하는 것이다.

### Fetching single itmes

[[#XGET]] 명령을 본다면, [[#XRANGE]] 가 효과적으로 단일 항목을 가져올수있기 때문에 실망할수 있다고 말한다.

>[!note] 뭐 굳이 실망할것 까지야...

이는 다음처럼 $2$ 번 같은 `ID` 를 지정해주면 된다.

```sh
> XRANGE mystream 1526984818136-0 1526984818136-0
1) 1) 1526984818136-0
   2) 1) "duration"
      2) "1532"
      3) "event-id"
      4) "5"
      5) "user-id"
      6) "7782813"
```

## XTRIM

[[#XTRIM]] 은 항목의 나열에서 필요하다면, 제거하는 명령어이다.

>[!info] 이는 가장 오래된 `ID` (`lower ID`) 를 제거한다<br><br>`lower ID` 는 `timestamp` 상 더 낮은 시간값을 가지므로, 더 오래된 `ID` 라 볼수 있다.

>[!info] [XTRIM](https://redis.io/docs/latest/commands/xtrim/)
```sh
XTRIM key <MAXLEN | MINID> [= | ~] threshold [LIMIT count]
```


`Trimming` 은 `stream` 에서 다음중 하나를 완료하는 전략이다.

- **MAXLEN**: `stream`  의 항목 개수가 `threshold` 만큼 다다르면 항목을 삭제한다.<br>`threshold` 는 양수이다.

```sh
XTRIM mystream MAXLEN 1000
```

- **MINID**: `threshold` 가 `ID` 일때, `threshold` 보다 작은 `ID`  를 제거한다.

```sh
XTRIM mystream MINID 649085820
```

### Nearly exact trimming

`Exact trimming` 은 해당 `threshold` 에 해당 조건에 맞게 정확하게 `trimming` 한다.
이는 `default` 값이며, 직접 명시한다면 `=` 을 사용한다.

하지만, `Redis` 는 조금더 효율적으로 `trimming` 하기 위한 `Nearly exact trimming` 방식을 사용할수 있도록 제공한다.

이는 옵셔널한 값으로 `~` 를 사용하여 명시할수있다.

```sh
XTRIM mystream MAXLEN ~ 1000
```

이것이 의미하는 바는, `stream` 의 최소 `thresold` 길이로 `trimming` 하는 명령어이다. 그러나, 약간 더 많을수 있다.

이 경우 `Redis` 는 `trimming` 을 일찍 종료한다.
이는 퍼포먼스상 이득을 얻는다고 말한다.

>[!info] 예를들어, 자료구조에서 전체 `macro node` 를 삭제할수 없을때, 퍼포먼스상 이득을 얻을수 있다고 한다.<br><br>[[#XTRIM]] 에서 `Redis`  는 전체 `매크로 노드` 단위로 삭제를 수행한다.

>[!info] `매크로 노드` 는 `Redis` `stream` 에서 여러개의 `stream` `entries` 를 그룹화하여 저장하는 단위이다.<br><br>각 `매크로 노드` 는 많은 수의 `stream` `entries` 를 포함한다.

이러한 구조는 조금더 효과적으로 `trimming` 할수 있으며, `threshold` 보다 약간의 추가된 `entries` 가 있겠지만, 일반적으로 만족할만하다.

```sh
XTRIM key <MAXLEN | MINID> [= | ~] threshold [LIMIT count]
```

이 명령어를 보면 `LIMIT` 절이 옵셔널하게 붙는다
`LIMIT` 을 같이 사용하면 `thresold` 에 의해 삭제될 `entries` 의 개수를 지정할 수 있다.

명시적으로 선언하지 않으면, 기본 값은 $100 * macro node 안의 항목 수$ 이며, `0` 을 지정하면 `LIMIT` 은 `disabled` 된다.

