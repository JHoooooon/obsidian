
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





