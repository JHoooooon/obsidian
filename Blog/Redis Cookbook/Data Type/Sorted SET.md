
>[!info] `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`Sorted` 는 각 요소에 가중치를 가지고 있으며, 정렬가능함을 의미한다.
항상 정렬이 필요한 경우 이러한 기본적으로 제공해주는 정렬기능을 활용하는것이 좋다

---

투표율을 가진 각 레스토랑을 정렬가능한 `SET` 으로 추가한다.
`ZADD` 이 요소를 추가하는 명령이다

>[!info] ZADD
```sh
127.0.0.1:6379> ZADD ranking:restaurants 100 "Olive Garden" 23 "PF Chang's" 34 "Outback Steakhouse" 45 "Red Lobster" 88 "Longhorn Steakhouse"
 (integer) 5
```

`ZADD` 명령은 다양한 옵션을 제공한다.

```sh
ZADD key [NX | XX] [GT | LT] [CH] [INCR] score member [score member
  ...]
```

- **XX** : 요소가 이미 존재할때만 스코어를 업데이트한다

- **NX**: 요소가 존재하지 않을때만 삽입 가능하며, 기존의 요소의 스코어가 존재한다면 삽입하지 않는다.

- **LT**: 스코어가 기존 요소의 스코어 보다 작을때에만 업데이트한다.<br>기존 요소가 존재하지 않는다면 새로운 값을 삽입한다.

- **GT**: 스코어가 기존 요소의 스코어 보다 클때에만 업데이트한다.<br>기존 요소가 존재하지 않는다면 새로운 값을 삽입한다.

- **CH**: 변경된 요소 수 반환한다

- **INCR**: 기존 전수에 새 점수를 더한다

>[!note] 보통 `CH` 와 `GT` 및 `INCR` 를 같이 사용한다.<br> 기본적은 `ZADD` 는 새롭게 추가된 요소의 개수를 반환하며, 업데이트 개수를 반환하지 않는다<br><br> 그러므로 `GT` 나 `LT` 를 사용해 업데이트 되더라도 반환값은 `0` 일 것이다.<br><br>이러한 부분을 개선하기 위해 `CH` 값을 같이 사용해서 명령하는것이 좋다.

```sh
> ZADD myset 10 "a" 20 "b" 30 "c"
(integer) 3

> ZADD myset GT 15 "a" 25 "b" 35 "c"
(integer) 0  // 새로 추가된 멤버가 없으므로 0 반환

> ZADD myset GT CH 15 "a" 25 "b" 35 "c"
(integer) 2  // "b"와 "c"의 점수가 업데이트되어 2 반환
```

`ranking` 을 검색하기 위해 `ZRANGE` 명령을 실행한다

>[!warning] 책에서는 `ZREVRANGE` 를 사용하는데, `ZREVRANGE` 는 `6.2.0` 버전 이후로 `deprecated` 되었다.<br><br>`ZRANGE` 에서 `REV` 를 사용하면 똑같은 `command` 이므로, 다음처럼 한다


>[!info] ZRANGE
```sh
127.0.0.1:6379> ZRANGE ranking:restaurants 0 -1 REV WITHSCORES
 1) "Olive Garden"
 2) "100"
 3) "Longhorn Steakhouse"
 4) "88"
 5) "Red Lobster"
 6) "45"
 7) "Outback Steakhouse"
 8) "34"
 9) "PF Chang's"
10) "23"
```

`ZRANGE` 는 다음과 같은 `OPTION` 을 가진다.

```sh
ZRANGE key start stop [BYSCORE | BYLEX] [REV] [LIMIT offset count]
  [WITHSCORES]
```

- **BYSCORE**: `BYSCORE` 는 `SCORE` 에서  `<start>` 와 `<stop>` 사이에 포함되는 범위의 요소를 리턴한다.<br><br>특이한 점은 `range` 값에 `(` 문자 있으면 `score` 가 값을 포함하지 않으며, 없으면 포함한다.<br> <br>`ZRANGE zset (1 5 BYSCORE` : 1 < socre <= 5 <br>`ZRANGE zset (1 (5 BYSCORE` : 1 < socre < 5 <br>`ZRANGE zset (1 +inf BYSCORE`: 1 < score < +inf

- **BYREX**:  `BYLEX` 옵션을 사용하면 사전식 순서를 이용해 특정 요소를 조회할수 있다.<br><br>`ZRANGE mySortedSet (b (f BYLEX`: 'b' < score < 'f'

- **REV**: `REV` 인자는 순서를 반대로 한다.<br>그래서 요소는 최고점수에서 최저 점수로 나열된다.<br>만약 점수가 동점이라면 역 사전순으로 정렬된다.

- **LIMIT**: `LIMIT` 은 `SQL 의 LIMIT offset` 과 같다.<br>매칭된 요소들의 개수를 제한한다.<br><br>음수 `offset count`  는 `offset` 의 모든 요소를 반환한다.<br>`offset` 이 큰 경우 반환할 요소에 도달하기 전에 `offset` 요소에 대한 정렬된 세트를 순회하므로,<br>이로 인해 최대 `O(N)` 시간 복잡도가 추가될수 있다.

- **WITHSCORE** : 선택적 인수인 `WITHSCORE` 인수는 반환된 요소 점수로 명령의 응답을 추가적으로 넣어 반환한다.<br><br>이는 `value1, ..., valueN` 으로 반환되는 대신 `value1, score1, ..., valueN, scoreN` 을 가진 `list` 를 반환한다.

다음은 `score` 값을 증가시킨다면 `ZINCRBY` 명령을 사용한다 

>[!info] ZINCRBY
```sh
127.0.0.1:6379> ZINCRBY ranking:restaurants 1 "Red Lobster"  
"46"
```

`ZREVRANK` 명령을 통해 `RANKING` 의 순위를 알수 있고, `ZSCORE` 는 해당 요소의 `score` 를 알수 있다

>[!info] ZREVRANK, ZSCORE
```sh
127.0.0.1:6379> ZREVRANK ranking:restaurants "Olive Garden" 
(integer) 0 
 
127.0.0.1:6379> ZSCORE ranking:restaurants  "Olive Garden" 
"100"
```

>[!info] ZREVRANK 는 높은 점수에서 낮은 점수순으로 정렬된 `member` 의 `rank` 값을 리턴한다.

`ZUNIONSTORE` 명령은 두 순위의 조합이 필요한 경우 사용할수 있는 명령어이다.

```sh
127.0.0.1:6379> ZADD ranking2:restaurants 50 "Olive Garden" 33 "PF Chang's" 55 "Outback Steakhouse" 190 "Kung Pao House" 
(integer) 4 
 
127.0.0.1:6379> ZUNIONSTORE totalranking 2 ranking:restaurants ranking2:restaurants WEIGHTS 1 2 
(integer) 6 
 
 
127.0.0.1:6379> ZREVRANGE totalranking 0 -1 WITHSCORES 
 1) "Kung Pao House" 
 2) "380" 
 3) "Olive Garden" 
 4) "200" 
 5) "Outback Steakhouse" 
 6) "144" 
 7) "PF Chang's" 
 8) "89" 
 9) "Longhorn Steakhouse" 
10) "88" 
11) "Red Lobster" 
12) "46" 
127.0.0.1:6379>
```

```sh
ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]
```

- **destination**: 결과를 저장할 새로운 Sorted Set 의 키 이름

- **numkeys**: 합칠 `Sorted Set` 의 개수

- **key**: 합칠 `Sorted Set` 의 키 이름들

- **WEIGHTS (선택적)**: 각 `Sorted Set` 의 점수를 곱할 가중치

- **AGGREGATE (선택적)**: 중복 맴버의 점수 계산 방식 (SUM | MIN | MAX) - default: SUM

추가적으로 `Redis` 는 `Sorted Set` 객체를 저장하기 위해 내부적으로 `2` 개의 `enconding` 방식을 사용한다.

- `ziplist`: `Sorted Set` 의 길이가 `zset-max-ziplist-entries` (`default`: `128`) 보자 작고, 모든 요소의 사이즈가 구성에서 `zset-max-ziplist-value`(`default`: $64byte$) 보다 작다면, 작은 공간을 저장하기 위해 `ziplist` `encoding` 방식을 사용한다. 

- `skiplist`: 구성별 `ziplist` `encoidng` 을 사용할수 없다면 `default` `encoding` 방식으로 `skiplist` 를 사용한다.



