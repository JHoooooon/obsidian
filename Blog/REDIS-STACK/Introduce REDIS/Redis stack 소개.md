
## multi-model real-time database

>[!info] 이 책은 `Redis Stack for Application Modernization` 의 책 내용을 요약한 것이다.

이 챕터에서는 `Redis-Stack` 에서  `multi-model` 기능을 소개한다.

이는 테이블의 `columns` 와 `rows` 안의 데이터를 구조화 하는, 관계형 패러다임을 사용하는데 익숙한 사람들에게 유용한 방식이다.

다음의 테이블이 있다고 가정하자.

##### city-table
```mysql
CREATE TABLE `city` (
  `ID` int NOT NULL AUTO_INCREMENT,
  `Name` char(35) NOT NULL DEFAULT '',
  `CountryCode` char(3) NOT NULL DEFAULT '',
  `District` char(20) NOT NULL DEFAULT '',
  `Population` int NOT NULL DEFAULT '0',
  PRIMARY KEY (`ID`),
  KEY `CountryCode` (`CountryCode`)
)
```

이를 `field-value` 쌍인 속성을 저장하기 위해 `HASH` 타입을 사용한다.
`HSET` 을 사용하여 해당 값을 저장하는 `command` 이다.

```r
> HSET city:653 Name "Madrid" CountryCode "ESP" District "Madrid" Population 2879052
```

그리고 제대로 젖아되었는지 확인하기 위해 전체 `HASH` 를 검색한다.
`HGETALL` 은 모든 `city:653` 의 `HASH` 값과 키를 반환한다.

```r
HGETALL city:653
1) "Name"
2) "Madrid"
3) "CountryCode"
4) "ESP"
5) "District"
6) "Madrid"
7) "Population"
8) "2879052"
```

그리고 테이블에서 `ID` 가 `653` 인 `Name` 과 `Poluaton` 을 쿼리해보자

```mysql
SELECT Name, Population FROM city WHERE ID=653;
+--------+------------+
| Name   | Population |
+--------+------------+
| Madrid |    2879052 |
+--------+------------+
1 row in set (0.00 sec)
```

`Redis` 에서는 다음의 `HMGET command` 로 값을 가져온다.

>[!info] `HGET` 은 `단일값` 을 가져오지만. `HMGET` 은 `여러값` 을 가져오는데 사용되는 `command` 이다.

```r
127.0.0.1:6379> HMGET city:653 Name Population
1) "Madrid"
2) "2879052"
```

이 예시에서 보면, `Redis` 에서 `city:653` 를, `sql` 에서는 `id = 653` 이라는 `Primary Key` 식별자를 기반으로 데이터를 가져온다. 

만약, 복잡한 데이터를 얻기 위해 데이터 셋의 `search` 쿼리같은 복잡한 쿼리를 해야 할수 있다.
이는 다음처럼 해결할수 있다.

## Secondary key lookup

기본적으로, `Primary key` 로 쿼리를 하는것이 더 효율적이다.

>[!info] `SQL` 은 `Primary key` 생성시 `index` 를 같이 생성하며, `Primary key` 는 `table` 의 `row` 에 바로 접근하는것을 보장한다.

하지만, 무조건 `Primary key` 를 사용하여 쿼리하지는 않는다.
다음은 다른 `Index` 를 생성한 `CountryCode` 컬럼을 사용하여 쿼리한다.

>[!info] [[#city-table]] 에서 테이블 생성시, `CountryCode` 의 `Index` 를 생성한것을 볼수있다.
```mysql
mysql> SELECT Name FROM city WHERE CountryCode = "ESP";
+--------------------------------+
| Name                           |
+--------------------------------+
| Madrid                         |
| Barcelona                      |
| [...]                          |
+--------------------------------+
```

`table` 에 정의된 `CountryCode` 의 `index` 로 효과적으로 데이터를 찾는다.
`Redis` 에서 저장된 `HASH command` 와 비교하기 위해 약간의 설명이 필요하다.

일단, 현재 `table` 의 데이터셋을 `Redis` 의 `HASH` 로 마이그래이션 했다고 가정한다.

`Redis` 는 `SQL` 이 아니므로, `Secondary Index` 기능을 제공하지 않으며, `city:` 네임스페이스의 접두사를 사용하여 `HASH` 를 전부 `scan` 해야 한다.

그때, `Redis` 는 모든 `HASH` 를 찾아보고, 찾기위한 조건이 맞는지 확인하는 과정이 필요하다

>[!info] 다음의 예는 `non-blocking scan` 를 하며, 구성가능한 `batch size`(`3` 개) 로 `city:*` 을 가진 모든 `key name` 을 필터링하도록 명령한다.  
```r
127.0.0.1:6379> SCAN 0 MATCH city:* COUNT 3
1) "512"
2) 1) "city:4019"
   2) "city:9"
   3) "city:103"
```

이는 전체 `SCAN` 을 했으며, 그중 `3` 개의 `city` 값을 가져온다. 

이러한 방식은, 전체 `SCAN` 이므로, 효과적인 `search` 방식은 아니며, 비싼 비용을 지불하는 `search` 방식이다.

현재 원하는 결과는 `CountryCode` 의 값에 맞는 모든 `city` 데이터셋이다.
이를 해결하기 위해서 다음의 효과적인 일반적인 옵션과 `Redis Stack`  기능이 필요하다.

- **Pipelining**: <br>파이프라인은 `commands` 집합이며, 이러한 `command` 집합을 서버로 전송한다.<br>서버는 이에 대한 결과를 한꺼번에 `client` 로 전송하는 방식이다.<br><br>그러므로, `command` 를 매번 여러개 보내는것이 아닌 한번에 서버로 전송하고, 한번에 결과값을 받는 유용한 방식이다.
>[!info] 책에서는 `마켓에서 30번 방문하여 물건을 30번 구매하는것과, 1번만 방문하여 물건을 30개 구매하는것` 의 차이라고 말한다.


- **Using functions**:<br>`Redis` 는 `rua` 라는 `script language` 를 내장하며,<br>이는 `client` 와의 `network` 대기시간과 `client` 의 작업부담을 덜어주는데 도움을 준다.<br><br>`client` 의 명령 `command` 보다 `local server` 에서 더 가까운곳에서 데이터를 검색하므로, 더 효율적이다.<br><br> 다음은 `CountryCode` 를 처리하는 `lua script` 이다.

>[!info]  `SQL` 의 `Stored Procedure` 과 비슷한 개념으로 생각하면 된다.<br><br> 이는 `서버` 내부에서 미리 구현된 함수를 실행하므로, `client` 에서 직접 작성한 `command` 보다 더 빠르고 효율적으로 데이터 검색이 가능하다는 의미로 해석된다.
```lua
#!lua name=mylib
local function city_by_cc(keys, args)
   local match, cursor = {}, "0";
   repeat
      local ret = redis.call("SCAN", cursor, "MATCH", "city:*", "COUNT", 100);
      local cities = ret[2];
        for i = 1, #cities do
         local keyname = cities[i];
         local ccode = redis.call('HMGET',keyname,'Name','CountryCode')
         if ccode[2] == args[1] then
            match[#match + 1] = ccode[1];
         end;
        end;
        cursor = ret[1];
      until cursor == "0";
   return match;
end
redis.register_function('city_by_cc', city_by_cc)
```

이는 다음처럼 `redis` 에서 실행가능하다.

```sh
> cat mylib.lua | redis-cli -x FUNCTION LOAD
```

```r
127.0.0.1:6379> FCALL city_by_cc 0 "ESP"
 1) "A Coru\xf1a (La Coru\xf1a)"
 2) "Almer\xeda"
[...]
59) "Barakaldo"
```

- **Using indexes**:<br>`Data SCAN` 은 안정적인 `real-time`  요구사항에 비효율적이며, 느리다.<br>특히 수백만 혹은 더많은 `key` 가 저장된 `keyspace` 일때 사실로 드러난다.<br><br>`search` 작업의 또 다른 접근법으로 `Redis Core` 의 자료구조는 `Secondary Index` 를 생성한다.<br>`Redis` 집합을 사용하여 이를 수행할수 있는 다양한 옵션이 존재하는데, `Set` 을 사용할경우 다음처럼 가능하다.

>[!info] 다음은 예시를 위해 `city:esp` 라는 `Set` 을 만든다.
```r
SADD city:esp "Sevilla" "Madrid" "Barcelona" "Valencia" "Bilbao" "Las Palmas de Gran Canaria"
```

>[!info] `SISMEMBER` 를 사용하여 특정 `city` 가 `Spain` 에 있는지 확인할수 있다.
```r
127.0.0.1:6379> SISMEMBER city:esp "Madrid"
(integer) 1
```

>[!info] `MATCH` 패턴을 사용하여 `name` 이 포함되었는지 `SCAN` 할수도 있다.
```r
127.0.0.1:6379> SSCAN city:esp 0 MATCH B*
1) "0"
2) 1) "Barcelona"
   2) "Bilbao"
```

>[!info] 만약 `검색 요구사항을 구체화 하기 위해 인구를 고려한 인덱스` 를 설계할수도 있다.<br><br> 다음은 `Sorted Set` 을 사용하고, `score` 로 인구를 설정한다.
```r
127.0.0.1:6379> ZADD city:esp 2879052 "Madrid" 701927 "Sevilla" 1503451 "Barcelona" 739412 "Valencia" 357589 "Bilbao" 354757 "Las Palmas de Gran Canaria"
(integer) 6


127.0.0.1:6379> ZRANGE city:esp 2000000 +inf BYSCORE
1) "Madrid"

127.0.0.1:6379> ZRANK city:esp Madrid
(integer) 5
```

이러한 접근 방식은, `secondary index` 에 대한 비용이 발생하기 십상이다.
이는 보면 ㅇ



- Redis Stack capabilities





