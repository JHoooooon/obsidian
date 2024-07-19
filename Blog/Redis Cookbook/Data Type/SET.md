
`SET` 은 정렬되지 않은 유일한 객체들에 대한 컬랙션 타입이다   

이는 종종 중복되는 값을 삭제하거나, 수학연산 (`union`, `intersection`, `difference` 같은 집합연산) 을 하는데 사용된다.

`Redis`  에서 `SET` 데이터 타입에 설정가능한 최대 요소의 개수는 $2^{23} - 1$  이다.
`SET` 에서 모든 요소를 반환하는 명령어인 `SMEMBERS` 를 사용할수 있지만, 그 양이 굉장히 많다면, `Redis` 서버는 멈출수 있다. 

>[!info] [[HASH]] 에서 처럼 저장된 값을 한번에 불러오면 `Redis Server` 가 `block` 될수 있다.

이를 해결하기 위해 `SET` 전용 `SSCAN`  을 사용한다.  
>[!info] `SSCAN` 은 [[HASH]] 의 `HSCAN` 과 상당히 유사하다.<br>`SCAN` 사용 관련해서 살펴본다면 [[HASH]] 의 `HSCAN` 부분을 보면 좋을듯 하다.

`Redis` 는 `union` (`SUNION`, `SUNIONSTORE`), `intersection` (`SINTER`, `SINTERSTORE`), `difference` (`DI)

``

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

