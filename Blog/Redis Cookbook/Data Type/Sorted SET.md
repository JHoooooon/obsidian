
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

