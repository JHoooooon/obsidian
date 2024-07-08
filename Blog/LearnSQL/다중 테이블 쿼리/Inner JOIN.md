
---

이전 쿼리를 수정하려면, 두 테이블이 어떤 관계인지 부터 확인해야 한다.
`cusotmer.addres_id` 와 `address.address_id` 는 `foreign key` 로써 종속관계에 있다.

```mysql
select 
	c.first_name,
	c.last_name,
	a.address
from
	customer c
		inner join address a 
		on c.address_id = a.address_id;
```

`599` 개의 `row` 가 생성되었다.
앞의 [[JOIN 이란?]] 에서 `361,197` 개의 `row` 와는 상반된다.

`inner join` 은 `customer` 테이블과 `address` 테이블을 비교하여, `address_id` 로 참조되지 않는 `row` 는 제외된다

즉, `customer` 테이블이 참조하는 `foreign key` 가 존재하는 `row` 만 가져온다 

만약, `customer` 테이블에는 존재하지 않지만, `address` 테이블에는 존재하는 `address_id` 는 결과셋에서 제외한다.

>[!info] 만약, `존재하지 않는 address_id` 를 포함해서 결과셋을 얻고 싶다면, `outer join` 및, `cross join` 을 통해서 가능하다.

다음은 위의 코드와 같다

```mysql
select 
	c.first_name,
	c.last_name,
	a.address
from
	customer c
	join address a 
	on c.address_id = a.address_id;
```

>[!info] `inner` 의 명시적 정의 가 없다면, `default` 로 `inner join`  으로 처리된다.

다음은 `on` 하위 절 대신, 같은 `field` 명이라면 `using` 절을 사용해도 된다.

```mysql
select 
	c.first_name,
	c.last_name,
	a.address
from
	customer c
	join address a 
	using(address_id);
```

>[!warning] 이는 `using` 을 사용할수 있는 간단한 표기법이므로, 혼돈을 피하려면 `on` 하위절을 사용하라한다.

### ANSI 조인 문법

책에서 테이블 조인에 사용된 표기법은 `ANSI SQL` 표준의 `SQL92` 버전에 준하여 소개한다.
`SQL92` 스펙이 출시되기 전부터 사용되었던 조인 문법은 다음과 같다

```mysql
SELECT
	c.first_name,
	c.last_name,
	a.address
FROM customer c, address a
WHERE c.address_id = a.address_id ;
```

이는 모든 `SQL` 서버들은 위의 방식으로 `JOIN` 가능하다
이는 `on` 절이 아닌 `where` 절을 사용하고, `join`  은 `from` 절로 여러 테이블을 참조한다.

`ANSI` 조인 문법은 다음과 같은 이점이 있다고 한다.

1. **조인 조건과 필터 조건은 이해하기 쉽게 두개의 각각 절로 구분된다**
2. **각 테이블 쌍에 대한 조인 조건이 `on` 절에 포함되어 있으므로, 조인 조건의 일부가 실수로 누락될 가능성이 낮다**
3. **표준화 되어 있어 데이터베이스 서버 이식이 가능하다.**

>[!info] 웬만하면 `on` 절을 사용하여 `join` 하도록 추천하고 있다.<br><br>반면 `from` `where` 절을 통한 `join` 을 사용하면, `JOIN` 문인지 아닌지 구분이 바로 되지는 않는다<br><br>하지만, 표준 이전에 있던 문법으로 알아두어야 한다.<br><br>`SQL 쿡북` 에서도 `form` `where` 절을 사용해서 처리하는 방법을 같이 보여주었던 것으로 기억한다.

### 세 개 이상 테이블 조인

```mysql
SELECT
	c.first_name,
	c.last_name ,
	ct.city 
FROM customer c
	INNER JOIN address a
	ON c.address_id = a.address_id
	INNER JOIN city ct
	ON a.city_id = ct.city_id;

SELECT
	c.first_name,
	c.last_name ,
	ct.city
FROM city ct
	INNER JOIN address a
	ON a.city_id = ct.city_id
	INNER JOIN customer c
	ON c.address_id = a.address_id;

SELECT
	c.first_name,
	c.last_name ,
	ct.city
FROM address a
	INNER JOIN city ct
	ON a.city_id = ct.city_id
	INNER JOIN customer c
	ON c.address_id = a.address_id;
```

이는 동일한 결과를 반환한다.

`SQL` 은 비절차적 언어이다.

데이터베이스는 수집된 통계를 이용해서 서버는 셋중 하나의 테이블을 시작점으로 선택한 다음 나머지 테이블을 조인할 순서를 결정한다. 

>[!info] 선택된 테이블을 `드라이빙 테이블`(`driveing table`) 이라한다.<br><br>이렇게 조인할 순서를 알아서 선택하기에 테이블 나열하는 순서는 중요하지 않다.

>[!note] 테이블을 원하는 순서로 조인하려면, 테이블을 원하는 순서로 배치한 다음 `MYSQL` 에서는 `straight_join` 키워드를 사용하거나, `SQL Server` 에서는 `force order` 옵션을, `oracle` 에서는 `leading` 힌트를 사용한다.

```mysql
SELECT STRAIGHT_JOIN 
	c.first_name, 
	c.last_name,
	ct.city
FROM city ct
	INNER JOIN address a
	ON a.city_id = ct.city_id
	INNER JOIN customer c
	ON c.address_id = a.address_id;
```

### 서브쿼리 사용

다음은 `customer` 테이블의 `address` 및 `city` 테이블에 대한 서브쿼리를 이용해 조인하는 예이다.

```mysql
SELECT
	c.first_name,
	c.last_name,
	addr.city
FROM customer c
INNER JOIN (
	SELECT
		a.address_id,
		a.address,
		ct.city
	FROM address a
		INNER JOIN city ct
		ON a.city_id = ct.city_id
	WHERE a.district = 'California'
) addr
ON c.address_id = addr.address_id;
```

`California` 에 거주하는 `customer` 의 `이름`, `성`, `주소 및 도시` 를 반환한다.

>[!note] 서브쿼리를 사용하지 않고 단순히 세 개의 테이블을 조인해서 작성할 수도 있지만, 하나 이상의 서브쿼리를 사용하는 편이 성능 및 가독성 측면에서 유리할 수 있다.

>[!info] 서브 쿼리에 대해서는 추후 설명한다고 한다.<br><br>서브 쿼리에 대한 동작방식을 이해해야 성능관련 부분에 대한 이해도가 쌓일수 있을것 같다.

### 테이블 재사용

여러 테이블을 조인할 경우 같은 테이블을 두번 이상 조인해야 할수도 있다

다음은 `film_actor` 테이블에 있는 영화와 관련있다.
두면의 특정 배우가 출연한 영화 제목을 찾으려면 다음과 같이 쿼리를 작성해야 한다

```mysql
SELECT
	f.title
FROM film f
	INNER JOIN film_actor fa
	ON f.film_id = fa.film_id
	INNER JOIN actor a
	ON fa.actor_id = a.actor_id
WHERE
	(a.first_name = 'CATE' AND a.last_name = 'MCQUEEN')
	OR (a.first_name = 'CUBA' AND a.last_name = 'BIRCH');
```

위는 `CATE McQueenN` 또는 `CUBA Birch` 가 출현한 영화를 모두 반환한다.
만약, 두사람 모두 출연한 영화를 출력한다고 하면, 이를 구분해서 쿼리 해야한다.

```mysql
SELECT
	f.title
FROM film f
	INNER JOIN film_actor fa1
	ON f.film_id = fa1.film_id
	INNER JOIN actor a1
	ON fa1.actor_id = a1.actor_id
	INNER JOIN film_actor fa2
	ON f.film_id = fa2.film_id
	INNER JOIN actor a2
	ON fa2.actor_id = a2.actor_id
WHERE
	(a1.first_name = 'CATE' AND a1.last_name = 'MCQUEEN')
	AND (a2.first_name = 'CUBA' AND a2.last_name = 'BIRCH');
```

```sh
title           |
----------------+
BLOOD ARGONAUTS |
TOWERS HURRICANE|
```

### 셀프 조인

쿼리에 동일한 테이블을 한 번 이상 포함할수 있을 뿐만 아니라 테이블을 자기자신과 조인할수 있다
자기자신과 조인한다고 해서 `Self JOIN` 이라한다.

>[!note] 처음에는 이상해 보일수 있다.<br>`자기 참조 외래 키`(`self-referencing foreign key`) 가 포함되기 때문이다.

>[!warning] 책에서 예시로 `prequel_film_id` 를 추가한것이다,<br> 실제 데이터에는 없는 타입이다.<br> `prequel_film_id` 는 `film_id` 로 자신을 참조하는 `forien key` 로써 사용된다.
```mysql
> desc film;

Field               |
--------------------+
film_id             |
title               |
description         |
release_year        |
language_id         |
original_language_id|
rental_duration     |
rental_rate         |
length              |
replacement_cost    |
rating              |
special_features    |
last_update         |
prequel_film_id     |
```

다음은 `film` 테이블에 해당 영화의 전편(`frequel`) 을 나타내는 `prequel_film_id` 열이 포함되어 있느지 확인하는 쿼리이다.

```mysql
SELECT
	f.title,
	f_prnt.title prequel
FROM film f
	INNER JOIN film f_prnt
	ON f_prnt.file_id = f.prequel_film_id
WHERE f.prequel_film_id IS NOT NULL;
```

이는 자기자신을 참조한다.

`prequel_film_id` 외래 키를 사용해서 `film` 테이블을 셀프 조인하는데, 어떤 테이블이 어떤 용도로 사용되는 명확히 하는 위해 테이블 별칭 `f` 와 `f_prnt` 를 정의한다.

>[!note] 궁금한게, `where` 절을 넣어야 하나?
>`inner join` 에서 이미 `null` 값이면,  제외되서 `where`  절 사용을 안해도 될것 같은데...<br>일단 넘기도록 하자..

