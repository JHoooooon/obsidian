
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

이는 `table` 에 정의된 `CountryCode` 의 `index` 로 효과적으로 데이터를 찾는다.
이를 `Redis` 에서 저장된 `HASH command` 와 비교하기 위해 약간의 설명이 필요하다.

우리가 이러한 ``
