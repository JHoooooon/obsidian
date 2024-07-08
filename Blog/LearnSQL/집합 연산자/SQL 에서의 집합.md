
---

## 집합연산의 규칙

집합연산을 수행할때 다음의 규칙을 적용해야 한다

1. **두 데이터셋 모두 같은 수의 열을 가져야한다**
2. **두 데이터셋 각 열의 자료형은 서로 동일해야 한다**

```mysql
SELECT 1 num, 'abc' str
UNION
SELECT 9 num, 'xyz' str;
```

```sh
num|str|
---+---+
  1|abc|
  9|xyz|
```

단일행으로 구성된 데이터셋을 생성한다.
이 경우 집합 연선자 `union` 은 데이터베이스 서버에서 두 집합의 모든 행을 결합한다.

>[!info] 이 쿼리는 여러 개의 독립적인 쿼리로 구성되므로 `복합쿼리`(`compound query`) 라고 한다
여러 집합연산이 필요한 경우 복합 쿼리에는 두개 이상의 쿼리를 포함할수 있다

## union 연산자

>[!note] `Union` 및 `union all` 연산자는 여러 데이터 집합을 결합할 수 있다.<br><br>`union` 은 결합된 집합을 정렬하고 중복을 제거하는 반면, `union all` 은 그렇지 않다는 것이다.

`union all` 을 사용하면 데이터셋의 행수는 결합되는 집합의 행수의 총합과 같다. 

다음은 `union all` 이기에 모든 결과셋을 반환한다

```mysql
SELECT
	'actr' type,
	a.first_name,
	a.last_name
FROM actor a
UNION ALL
SELECT
	'actr' type,
	a.first_name,
	a.last_name
FROM actor a
```

위의 쿼리는 `798` 개의 데이터셋을 반환한다.

반면 아래는 중복된 열을 제거하고, 데이터셋을 반환한다
아래 쿼리는 `199` 개의 데이터셋을 반환한다

```mysql
SELECT
	'actr' type,
	a.first_name,
	a.last_name
FROM actor a
UNION
SELECT
	'actr' type,
	a.first_name,
	a.last_name
FROM actor a
```

이 쿼리들은 `actor` 와 `actor` 의 정보를 합친 테이블을 쿼리한다
[[#집합연산의 규칙]] 에서와 같이 `같은 수의 열` 과 `각 열의 같은 타입` 을 가지기에, `union` 처리가 가능하다 

책에서 다음과 같은 쿼리를 보여주며 `sakila` 데이터베이스에서의 예시를 보여준다
다음은, `customer` 와 `actor` 의  `first_name`, `last_name` 의 `union` 이며 
두 쿼리 모두 이니셜이 `JD` 인 사람을 쿼리한다.

`union all` 을 사용하면 중복되는 이름이 나오는것을 볼수 있다.

```mysql
SELECT
	first_name,
	last_name
FROM customer c
WHERE c.first_name LIKE 'J%' AND c.last_name LIKE 'D%'
UNION ALL
SELECT
	first_name,
	last_name
FROM actor c
WHERE c.first_name LIKE 'J%' AND c.last_name LIKE 'D%'
```

```sh
first_name|last_name|
----------+---------+
JENNIFER  |DAVIS    |
JENNIFER  |DAVIS    |
JUDY      |DEAN     |
JODIE     |DEGENERES|
JULIANNE  |DENCH    |
```

다음은 `union` 으로 출력한 데이터셋이다.

```mysql
SELECT
	first_name,
	last_name
FROM customer c
WHERE c.first_name LIKE 'J%' AND c.last_name LIKE 'D%'
UNION ALL
SELECT
	first_name,
	last_name
FROM actor c
WHERE c.first_name LIKE 'J%' AND c.last_name LIKE 'D%'
```

```sh
first_name|last_name|
----------+---------+
JENNIFER  |DAVIS    |
JUDY      |DEAN     |
JODIE     |DEGENERES|
JULIANNE  |DENCH    |
```

고유한 값을 출력하는것을 볼 수 있다.

## Intersect 연산자

>[!info] `ANSI SQL` 사양에서는 `intersect` 연산을 포함한다.<br><br> 하지만 `MySQL 8` 에서는 `intersect` 연산자 지원을 하지 않는다.<br>다음은 `postgersql` 에서의 `interscet` 이다.<br><br>`MySQL 8.03` 이상부터는 지원한다고 말한다. 

```postgresql
SELECT 
	c.first_name,
	c.last_name
FROM customer c
WHERE c.first_name LIKE 'J%' AND c.last_name LIKE 'D%'
INTERSECT
SELECT
	first_name,
	last_name
FROM actor c
WHERE c.first_name LIKE 'J%' AND c.last_name LIKE 'D%'

```

```sh
first_name|last_name|
----------+---------+
JENNIFER  |DAVIS    |
```

쿼리시, 두 집합의 교집합을 반환하는것을 볼 수 있다.

>[!info] 만약 집합이 하나도 겹치지 않으면 두 집합의 교차연산은 빈 집합이다.

>[!info] `ANSI SQL` 사양은 중복을 제거하지 않는 `intersect all` 연산자를 제공한다.<br>하지만 이는 `DB2` 에서만 제공한다고 말한다.

다음은 `MySQL` 의 `intersect`  를 `join` 으로 구현한 내용이다.

```mysql
SELECT DISTINCT 
	c.first_name,
	c.last_name
FROM customer c
INNER JOIN actor a
	ON (c.first_name LIKE 'J%' AND c.last_name LIKE 'D%') 
	AND (a.first_name LIKE 'J%' AND a.last_name LIKE 'D%')
```

```mysql
SELECT
	c.first_name,
	c.last_name
FROM customer c
WHERE
	c.first_name IN (
	SELECT
		a.first_name
			FROM actor a
		WHERE a.first_name LIKE 'J%'
	) AND
	c.last_name IN (
		SELECT
			a.last_name
		FROM actor a
		WHERE a.last_name LIKE 'D%'
	);
```

이는 이전의 `Intersect` 와 같다.

```sh
first_name|last_name|
----------+---------+
JENNIFER  |DAVIS    |
```
## execpt 연산자

>[!info] `ANSI SQL` 에서 `except` 연산을 수행하는 연산자는 포함된다.<br>하지만 `MySQL 8.0` 은 `except` 연산자를 지원하지 않는다. <br><br>다음은 `Postgersql` 에서 사용되는 쿼리이다. 

다음은 `actor` 에서 `first_name` 과 `last_name` 을 출력한 결과이다.

```mysql
SELECT
	a.first_name,
	a.last_name
FROM actor a
WHERE a.first_name LIKE 'J%' AND a.last_name LIKE 'D%'
```

```sh
first_name|last_name|
----------+---------+
JENNIFER  |DAVIS    |
JUDY      |DEAN     |
JODIE     |DEGENERES|
JULIANNE  |DENCH    |
```

다음은 `customer` 에서 `first_name` 과 `last_name` 을 출력한 결과이다.

```mysql
SELECT
	c.first_name,
	c.last_name
FROM customer c
WHERE c.first_name LIKE 'J%' AND c.last_name LIKE 'D%'
```

```sh

```

```postgresql
SELECT
	a.frist_name,
	a.last_name
FROM actor a
WHERE a.first_name LIKE '%J' AND a.last_name LIKE '%J'
EXCEPT
SELECT
	c.frist_name,
	c.last_name
FROM customer c
WHERE c.first_name LIKE '%J' AND c.last_name LIKE '%J';
```

```sh
first_name|last_name|
----------+---------+
JENNIFER  |DAVIS    |
```





