
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
first_name|last_name|
----------+---------+
JENNIFER  |DAVIS    |
```

이 쿼리에서 `actor` 집합의 `EXCEPT` 를 연산하면 `actor` 집합에만 있는 데이터셋을 가져와야 한다.
즉, `JENNIFER DAVIS` 을 제외한 나머지 `actor` 집합을 가져와야 한다.

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
JUDY      |DEAN     |
JODIE     |DEGENERES|
JULIANNE  |DENCH    |
```

위처럼 출력되며, `MySQL` 에서는 다음처럼 쿼리할 수 있다.

```mysql
SELECT
	a.first_name,
	a.last_name
FROM actor a
INNER JOIN customer c
	ON (a.first_name LIKE 'J%' AND a.last_name LIKE 'D%')
	AND (c.first_name LIKE 'J%' AND c.last_name LIKE 'D%')
WHERE c.first_name <> a.first_name AND c.last_name <> a.last_name;
```

```sh
first_name|last_name|
----------+---------+
JUDY      |DEAN     |
JODIE     |DEGENERES|
JULIANNE  |DENCH    |
```

`inner join` 이후에, `customer` 의 이름을 제외하면, `actor` 에 있는 이름을 출력할수 있다.

>[!info] `DB2` 에서는 `expect all` 을 지원한다고 말한다.<br> 이는 `ANSI` 표준에서 지원하지만, `MySQL` 에서는 지원하지 않는다.<br><br>이처럼 이루어진 `expect all` 연산은 `A` 와 `B` 연산시, 중복되지 않는 `A` 집합의 데이터셋과,  `A` 와 `B` 의 중복되는 `B` 의 데이터셋을 포함한다.  

## 복합 쿼리의 결과 정렬

복합 쿼리의 결과를 정렬하고 싶다면 `order by` 절을 쿼리 마지막에 추가한다.
`order by` 절에서 필드 명을 참조할때, 첫번째 쿼리의 이름을 선택해야 한다

```mysql
SELECT
	a.first_name fname,
	a.last_name lname
FROM actor a
WHERE a.first_name LIKE 'J%' AND a.last_name LIKE 'D%'
UNION ALL
SELECT
	c.first_name,
	c.last_name
FROM customer c
WHERE c.first_name LIKE 'J%' AND c.last_name LIKE 'D%'
ORDER BY lname, fname; -- <- 첫번째 쿼리의 fname 과 lname 을 참조
```

```sh
fname   |lname    |
--------+---------+
JENNIFER|DAVIS    |
JENNIFER|DAVIS    |
JUDY    |DEAN     |
JODIE   |DEGENERES|
JULIANNE|DENCH    |
```

>[!warning] 두번째 쿼리의 `field` 명을 `order by` 절에서 사용하면, `Error` 가 발생한다.<br><br>가독성있게 하고 싶다면, 첫번째 쿼리와 두번째 쿼리의 `field` 명의 `alias` 를 같은 명칭으로 하는것이 좋다.

## 집합 연산의 순서

**복합 쿼리가 서로 다른 집합 연산자를 사용하는 두개 이상의 쿼리를 포함할 경우 원하는 결과**를 얻으려면 **복합 쿼리문에 쿼리를 배치할 순서를 고려**해야 한다.

```mysql
SELECT
	a.first_name,
	a.last_name
FROM actor a
WHERE a.first_name LIKE 'J%' AND a.last_name LIKE 'D%'
UNION ALL
SELECT
	a.first_name,
	a.last_name
FROM actor a
WHERE a.first_name LIKE 'M%' AND a.last_name LIKE 'T%'
UNION
SELECT 
	c.first_name, 
	c.last_name
FROM customer c
WHERE c.first_name LIKE 'J%' AND c.last_name LIKE 'D%';
```

```sh
first_name|last_name|
----------+---------+
JENNIFER  |DAVIS    |
JUDY      |DEAN     |
JODIE     |DEGENERES|
JULIANNE  |DENCH    |
MARY      |TANDY    |
MENA      |TEMPLE   |
```

다음은 `union` 과 `union all` 연산자의 위치를 변경해본다.

```mysql
SELECT
	a.first_name,
	a.last_name
FROM actor a
WHERE a.first_name LIKE 'J%' AND a.last_name LIKE 'D%'
UNION
SELECT
	a.first_name,
	a.last_name
FROM actor a
WHERE a.first_name LIKE 'M%' AND a.last_name LIKE 'T%'
UNION ALL
SELECT 
	c.first_name, 
	c.last_name
FROM customer c
WHERE c.first_name LIKE 'J%' AND c.last_name LIKE 'D%';
```

```sh
first_name|last_name|
----------+---------+
JENNIFER  |DAVIS    |
JUDY      |DEAN     |
JODIE     |DEGENERES|
JULIANNE  |DENCH    |
MARY      |TANDY    |
MENA      |TEMPLE   |
JENNIFER  |DAVIS    |
```

이는 집합 연산자의 위치에 따라 복합 쿼리를 처리하는 방식에 분명한 차이가 있다.
이는 위에서 아래로 시작되는데, 약간의 예외사항이 있을수 있다.

1. `ANSI SQL` 사항에는 `intersect` 연산자가 다른 집합 연산자보다 우선 순위를 가진다.
2. 여러 쿼리를 괄호로 묶어 쿼리가 결합되는 순서를 지정할 수 있다.

>[!warning] `MySQL` 에서는 복합 쿼리시 괄호를 허용하지 않는다. 하지만 다른 데이터베이스에서는 복합 쿼리를 괄호로 묶어서 처리 순서를 재정의가능하다.

```postgresql
SELECT
	a.first_name,
	a.last_name
FROM actor a
WHERE a.first_name LIKE 'J%' AND a.last_name LIKE 'D%'
UNION
SELECT
	a.first_name,
	a.last_name
FROM actor a
WHERE a.first_name LIKE 'M%' AND a.last_name LIKE 'T%'
UNION ALL
(
SELECT 
	c.first_name, 
	c.last_name
FROM customer c
WHERE c.first_name LIKE 'J%' AND c.last_name LIKE 'D%'
);
```

이는 괄호처리한 `UNION ALL` 먼저 실행한다.






