>[!info] `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`HASH` 데이터 타입은 `field` 와 `value` 사이의 관계 매핑을 표현한 타입이다
`programing language` 에서 `dictionaries` 혹은 `maps` 과 같다.

`Redis` 는 `HSET`  과 `HMSET` 과 함께 자동적으로 `HASH` 를 생성한다.
`Redis` 는 비어있는 `Hash` 가 있다면, 자동적으로 삭제 처리한다.

>[!info] [[LIST]] 에서 자동적으로 `List` 를 생성하는것과 비슷하게 동작한다.

`HASH` 는 최대 $2^{23} - 1$  `field` 의 개수를 삽입할수 있다.
이렇듯 `HASH` 안의 `field` 의 수가 많은경우, `HGETALL` `command` 를 실행하면,  `Redis` 서버가 차단될수 있다.

>[!info] 너무 많은 값을 전부다 출력한다면, `server` 에 과부하가 될수 있음을 말하는듯하다.

이 경우 `HSCAN` 은 모든 `field`  를 점진적으로 검색할수 있다.
`HSCAN` 은 `Redis` `SCAN` 명령중 하나이다.

>[!info] `SCAN`, `HSCAN`, `SSCAN`, `ZSCAN` 이 존재한다

이러한 `SCAN` 명령은 점진적으로 반복해서 요소를 찾으므로, `server` 가 `block` 되지 않도록 한다.

이 명령은 `cursor-based interator` (`cursor` 기반으로된 `interator`) 로 매번 `SCAN` 을 매번 호출하여 `cursor` 를 지정해주어야 한다.

>[!info] `cursor` 의 시작지점은 `0` 이다.

`cursor` 가 끝이나면, `Redis` 는 `새로운 cursor` 와 함께 요소 `list` 를 반환하며, `새로운 cursor` 는 다음 반복에서 사용가능하다. 

`HSCAN`  은 다음의 포멧을 사용하여 호출한다

- `HSCAN` `cursor` 는 `[MATCH pattern] [COUNT number]` 옵션을 사용한다

- `MATCH` 옵션은 `glob-style` 패턴을 사용하여 `field` 를 매치할수 있다.

- `COUNT` 옵션은 각 반복한다, 얼마나 많은 요소를 반환할지 주는 `hint` 값이다.<br>`Redis` 는 `COUNT` 에 매치된 요소의 반환 개수를  보장하지 않는다. 단지 `hint` 를 줄 뿐이다.<br><br> `default` 는 `10` 으로 설정된다.

다음은 `HSCAN` 명령을 사용하여, `garden`  을 포함하는 `field` 를 반환하는 예시이다.
`0` 은 `cursor` 의 시작지점부터 반복하라는 명령이다

```sh
127.0.0.1:6379> HSCAN restaurant_ratings 0 MATCH *garden* 
1) "309" 
2) 1) "panda garden" 
   2) "3.9" 
   3) "chang's garden" 
   4) "4.5" 
   5) "rice garden" 
   6) "4.8"
```

`309` 는 새로운 `cursor` 를 말하며, 새롭게 스캔을 시작하기 위해 사용한다.

```sh
127.0.0.1:6379> HSCAN restaurant_ratings 309 MATCH *garden* 
1) "0" 
2) 1) "szechuwan garden" 
   2) "4.9" 
   3) "garden wok restaurant" 
   4) "4.7" 
   5) "win garden" 
   6) "4.0" 
   7) "east garden restaurant" 
   8) "4.6"
```

`0` 을 다시 반환하는데, 이는 `cursor` 가 전체를 `scan` 완료 하였다는 것을 의미한다.

시작지점을 가진 새로운 `cursor` 를 다시 반환한것이므로 다시 처음부터 스캔하고 싶다면 `0` 을 사용하여 `HSCAN` 하면 된다.

---

`Redis` 는 내부적으로 `HASH` 객체를 `2` 개의 `encoding` 을 사용한다

- `ziplist`: `list-max-ziplist-entries`(`default`: $512$) 보다 작고 목록에 있는 모든 요소의 크기가 `list-max-ziplist-value`(`default`: $64 byte$) 보다 작은 `HASH` 의 경우, `ziplist` 는 `HASH` 를 공간을 절약하는데 사용된다.

- `hashtable`: `ziplist` `enconding`  을 구성별로 사용할수 없는 경우 사용되는 `default encoding` 이다.

---

다음은 `HMSET` 을 사용하여, `HASH` 에 여러 `fields` 를 생성한다.

>[!info] HMSET 
```sh
127.0.0.1:6379> HMSET "Kyoto Ramen" "address" "801 Mission St, San Jose, CA" "phone" "555-123-6543" "rating" "5.0" 
OK
```

이는 `Kyoto Ramen` 이라는 `HASH` 를 생성하고, `address` 와 `phone`,`rating` `fields` 를 생성한다.

`HGET` 은 `property` 의 `key` 값으로 `value` 값을 가져온다.

>[!info] HGET
```sh
127.0.0.1:6379> HGET "Kyoto Ramen" "rating" 
"5.0" 
```

만약 `HASH` 안에 `field` 가 있는지 확인하려면, `HEXISTS` 를 사용한다.
`1`  은 존재하는 `field` 이며, `0` 은 존재하지 않는 `field` 이다.

>[!info] HEXISTS
```sh
127.0.0.1:6379> HEXISTS "Kyoto Ramen" "phone" 
(integer) 1 
127.0.0.1:6379> HEXISTS "Kyoto Ramen" "hours" 
(integer) 0
```

다음은, 모든 `field` 를 반환하는 명령어이다

>[!info] HGETALL
```sh
127.0.0.1:6379> HGETALL "Kyoto Ramen" 
1) "address" 
2) "801 Mission St, San Jose, CA" 
3) "phone" 
4) "555-123-6543" 
5) "rating" 
6) "5.0" 
```

`HGETALL` 은 해당 `key` 값을 가진 `HASH` 의 모든 `field` 와 `value` 를 반환한다.
만약, `field` 의 `value` 값을 변경하거나, 추가한다면, `HSET` 을 사용한다.

>[!info] HSET
```sh
127.0.0.1:6379> HSET "Kyoto Ramen" "rating" "4.9" 
(integer) 0 
127.0.0.1:6379> HSET "Kyoto Ramen" "status" "open" 
(integer) 1 
 
127.0.0.1:6379> HMGET "Kyoto Ramen" "rating" "status" 
1) "4.9" 
2) "open" 
```

`field` 를 삭제하려면 `HDEL` 을 사용한다.

>[!info] HDEL
```sh
127.0.0.1:6379> HDEL "Kyoto Ramen" "address" "phone" 
(integer) 2 
127.0.0.1:6379> HGETALL "Kyoto Ramen" 
1) "rating" 
2) "4.9"
```

`HSET` 및 `HMSET` 은 기존에 있는 `field` 가 존재한다면 새로운 값으로 덮어씌운다
`HSETNX` 를 사용하면, `field` 가 존재하지 않을때만 `value` 값을 할당한다.

>[!info] HSETNX
```sh
127.0.0.1:6379> HMSET "Kyoto Ramen" "address" "801 Mission St, San Jose, CA" "phone" "555-123-6543" "rating" "5.0" 
OK
127.0.0.1:6379> HSETNX "Kyoto Ramen" "phone" "555-555-0001" 
(integer) 0 
127.0.0.1:6379> HGET "Kyoto Ramen" "phone" 
"555-123-6543"
```

만약 `HGET` 혹은 `HMGET` 에서 해당 `field` 가 존재하지 않는다면 `nil` 값을 반환한다.

>[!info] NIL
```sh
127.0.0.1:6379> HMGET "Kyoto Ramen" "rating" "hours" 
1) "4.9" 
2) (nil) 
 
127.0.0.1:6379> HGET "Little Sheep Mongolian" "address" 
(nil)
```

