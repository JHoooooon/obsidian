
`SET` 은 정렬되지 않은 유일한 객체들에 대한 컬렉션 타입이다   

이는 종종 중복되는 값을 삭제하거나, 수학연산 (`union`, `intersection`, `difference` 같은 집합연산) 을 하는데 사용된다.

`Redis`  에서 `SET` 데이터 타입에 설정가능한 최대 요소의 개수는 $2^{23} - 1$  이다.
`SET` 에서 모든 요소를 반환하는 명령어인 `SMEMBERS` 를 사용할수 있지만, 그 양이 굉장히 많다면, `Redis` 서버는 멈출수 있다. 

>[!info] [[HASH]] 에서 처럼 저장된 값을 한번에 불러오면 `Redis Server` 가 `block` 될수 있다.

이를 해결하기 위해 `SET` 전용 `SSCAN`  을 사용한다.  

>[!info] `SSCAN` 은 [[HASH]] 의 `HSCAN` 과 상당히 유사하다.<br>`SCAN` 사용 관련해서 살펴본다면 [[HASH]] 의 `HSCAN` 부분을 보면 좋을듯 하다.

`Redis` 는 `union` (`SUNION`, `SUNIONSTORE`), `intersection` (`SINTER`, `SINTERSTORE`), `difference` (`SDIFF`, `SDIFFSTORE`) 작업을 위한 명령 그룹을 제공한다.

위의 `STORE` 접미사가 없는 명령어 (`SUNION`, `SINTER`, `SDIFF`) 는 단순히 해당 작업에 대한 `SET` 의 결과 값을 반환한다. 

`STORE` 접미사가 있는 명령어 (`SUNIONSTORE`, `SINTERSTORE`, `SDIFFSTORE`) 명령은 목적지 `key` 안으로, 연산된 결과를 저장한다.

```sh
127.0.0.1:6379> SMEMBERS "Original Buffalo Wings" 
1) "affordable" 
2) "great taste" 

127.0.0.1:6379> SADD "Big Bear Wings" "affordable" "spacious" "great music" 
(integer) 3 

127.0.0.1:6379> SINTER "Original Buffalo Wings" "Big Bear Wings" 
1) "affordable" 
 
127.0.0.1:6379> SINTERSTORE "common_tags" "Original Buffalo Wings" "Big Bear Wings" 
(integer) 1 

127.0.0.1:6379> SMEMBERS "common_tags" 
1) "affordable"
```

`Redis` 는 `SET` 객체 저장을할때 내부적으로, `2` 가지 `encoding` 을 수행한다

- `intset`: 모든 요소가 `integer` 이고 구성에서 `set-max-intset-entries` (`default`: $512$) 보다 작은 `SET` 의 경우 `encoding` 된다<br><br>`intset`  은 작은 `HASH` 에 대한 공간을 절약하는데 사용된다. 

- `hashtable`: 구성별로 `inset` `encoding`  을 사용할수 없는 경우 `default` `encoding` 으로 사용된다.

---

`tags` 를 `SET` 데이터타입에 저장한다. 

>[!info] SADD
```sh
127.0.0.1:6379> SADD "Original Buffalo Wings" "affordable" "spicy" "busy" "great taste"(integer) 4
```

`SET` 데이터 타입에 요소가 있는지 확인하기 위해 `SISMEMBER` 를 사용한다
값이 없다면 `0` 있다면, `1` 을 반환한다.

>[!info] SISMEMBER
```sh
127.0.0.1:6379> SISMEMBER "Original Buffalo Wings" "busy"
(integer) 1

127.0.0.1:6379> SISMEMBER "Original Buffalo Wings" "costly"
(integer) 0
```

`SET` 데이터 타입에서 요소를 삭제하려면 `SREM` 을 사용한다
삭제되었다면, 삭제된 개수를 반환하며, 없다면 `0` 을 반환한다

>[!info] SREM
```sh
127.0.0.1:6379> SREM "Original Buffalo Wings" "busy" "spicy"
(integer) 2

127.0.0.1:6379> SISMEMBER "Original Buffalo Wings" "busy"
(integer) 0

127.0.0.1:6379> SISMEMBER "Original Buffalo Wings" "spicy"
(integer) 0
```

`SET` 데이터 타입에 저장된 요소의 `member` 의 개수를 알기 위해서 `SCARD` 를 사용한다

>[!info] `SET CARDINALITY` 를 반환하기에 `SCARD` 이다.<br> `CARDINALITY` 는 `element` 의 개수를 말한다.

>[!info] SCARD
```sh
127.0.0.1:6379> SCARD "Original Buffalo Wings" 
(integer) 2 
```

