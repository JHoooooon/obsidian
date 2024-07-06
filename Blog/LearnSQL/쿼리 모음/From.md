
---

## From 절

>[!info] `From` 절은 쿼리에 사용디는 테이블을 명시할 뿐만 아니라, 테이블을 서로 연결하는 수단도 함께 정의한다.

### 테이블 유형

`Table`  이라는 용어는 데이터베이스에 저장된 일련의 관련 행 집합을 떠올린다.

이 책에서는 데이터가 어떻게 저장되는지에 관한 개념 대신, 관련 행들의 집합에 집중하여 테이블 용어를 더 일반적인 방식으로 사용한다고 말한다.

- **영구테이블**(`permanent table`): `create talbe` 문으로 생성

- **파생테이블**(`derived table`): 하위 쿼리에서 반환하고 메모리에 보관된 행

- **임시테이블**(`temporary table`): 메모리에 저장된 휘발성 데이터 

- **가상테이블**(`virtual table`): `create view` 문으로 생성

각 테이블 유형은 `from`  절에 포함될 수 있다
다음은 각 유형에 대해서 살펴본다

### 파생 테이블

`Subquery`  는 다른 쿼리에 포함된 쿼리이다.
`Subquery`  는 괄호로 묶여 있으며 `select` 문의 여러 부분에서 찾을 수 있다.

`from`  절에서 생성된 `Subquery` 는 다른 테이블과 상호작용할수 있는 파생 테이블을 생성하는 역할을 한다.

```mysql
select concat(
		cust.last_name, 
		', ', 
		cust.first_name
	) full_name
from (
	select 
		first_name, 
		last_name, 
		email
	from customer
	where first_name = 'JESSIE'
) cust
```

위는 `from`  절에 `subquery` 를 생성하여, `derived table`  을 만든 예시이다.
이는 쿼리된 결과를, `cust` 라는 `alias` 로 참조하여, `select` 문의 `concat` 에서 이름을 합치고 결과를 테이블로 반환한다.

>[!info] `cust` 의 데이터는 쿼리 기간 동안 메모리에 보관된 후 삭제된다. 

### 임시테이블

구현방식이 약간 다르지만, 관계형 데이터베이스는 휘발성의 임시 테이블 생성이 가능하다
임시테이블은 삽입된 데이터는 어느 시점에 사라진다.

>[!info] 보통 트랜잭션이 끝날때 또는 데이터베이스 세션이 닫힐때까지 삭제된다.

```mysql
-- temporary table 생성
create temporary table actors_j
	(
		actor_id smallint(5) primary key,
		first_name varchar(45),
		last_name varchar(45)
	);

-- actor 의 반환테이블을 actors_j 에 삽입
-- actor 의 반환테이블은 J로 시작하는 last_name 의 모음이다.
insert into actors_j 
	select actor_id, first_name, last_name
	from actor
	where last_name LIKE 'J%';

select * from actors_j;
```

```sh
actor_id|first_name|last_name|
--------+----------+---------+
       8|MATTHEW   |JOHANSSON|
      43|KIRK      |JOVOVICH |
      64|RAY       |JOHANSSON|
      82|WOODY     |JOLIE    |
     119|WARREN    |JACKMAN  |
     131|JANE      |JACKMAN  |
     146|ALBERT    |JOHANSSON|
```

`actor_j` 는 `last_name` 이 `J` 로 시작하는 모든 데이터를 반환한다.

>[!info] 이는 일시적인 메모리로 저장되며, 세션 종료시 사라진다.<br><br>대부분의 데이터베이스 서버는 세션이 종료될때 임시 테이블을 삭제한다.<br> 하지만 오라클은 세션에서 임스테이블 정의를 사용할 수 있도록 유지할 수 있다고 한다.

### 가상테이블 (view)

`view`  는 데이터 딕셔너리에 저장된 쿼리이다.
마치 테이블처럼 동작하지만 `viwe` 는 저장된 데이터가 없으며 읽기 전용이다.

그렇기에 `Virutal Table`  이라고도 부른다
`view`  에 대한 쿼리를 실행하면 쿼리가 뷰 정의와 합쳐져 실행할 최종 쿼리를 만든다

```mysql
create view cust_vw AS
	select customer_id, first_name, last_name, active
	from customer;
```

앞의 예시는 `cust_vw` 를 만든다.
이는 `AS` 절 및의 `select` 문을 실행하고, 이에 대한 결과를 반환한다.

>[!info] 뷰를 작성하더라도 데이터가 추가 생성되거나 저장되지는 않는다.<br> 서버는 이후 사용할 때 `select` 문 대신 `view` 가 존재하므로 다음과 같이 뷰를 쿼리한다.

```mysql
select first_name, last_name
from cust_vw
where active = 0;
```

사용자로 부터 열을 숨기고 복잡한 데이터베이스 설계를 단순화하는 등 다양한 이유로 `view` 가 만들어진다.

### 테이블 연결

`ANSI` 에서는 다음처럼 `JOIN` 문을 사용하여, 처리한다.

```mysql
select 
	cust.firstn_name,
	cust.last_name,
	time(rental.rental_date) rental_time
from customer cust
	inner join rental
	on cust.customer_id = rental.customer_id
where
	date(rental.rental_date) = '2005-06-14';
```

>[!info] `JOIN` 문은 별도의 챕터에서 다룬다

### 테이블 별칭 정의

`select` 에서 사용했듯이 `table` 에도 `alias` 지정이 가능하다.
이는 여러 `table` 을 `join`  하거나 사용해야할때, `select`, `where`, `group by`, `have` , `order by`  절에서 해당 `table` 을 참조하는데 쓰인다.

```mysql
select 
	c.first_name fname, 
	c.last_name lname, 
	time(r.rental_date) rental_time
from customer c
	inner join rental r
		on c.customer_id = r.customer_id
where date(r.rental_date) = '2005-06-14';
```

여기서 `customer` 테이블은 `c` , `rental` 테이블은 `r` 로써 참조된다.
`AS` 키워드로 가독성있게 사용해도 된다.

### Where 절

테이블에서 모든 행을 검색하는대신 관심 없는 행을 필터링할때 사용하는 구절이다.

>[!info] `where` 절은 결과셋에 출력되기를 원하지 않는 행을 필터링하는 매처니즘이다.

```mysql
select title from film where rating = 'G' and rental_duration >= 7;
```

위의 구절은 `film`  테이블에서 `rating`  이 `G` 이고, `rental_duration` 이 `7` 보다 크거나 같은 영화를 필터링한다.

>[!info] 이렇게 `where` 절에 필터링 하기 위한 조건을 `필터조건`(`filter conditions`) 이라한다.

다음은 `rating` 이 `G` 이거나, `retal_duration` 이 `7` 과 같거나 큰 필터조건이다.

```mysql
select title, rating, rental_duration
from film
where rating = 'G' or rental_duration >= 7;
```

다음은 `rating` 이 `G` 이고 `rental_duration`이 `7` 보다 같거나 크거나,
`rating`  이 `PG-13`  이고 `rental_duration` 이  `4` 보다 작은 필터 조건이다. 

```mysql
select title, rating, rental_duration
from film
where 
	(rating = 'G' and rental_duration >= 7) or 
	(rating = 'PG-13' and rental_duration < 4);
```

위의 필터조건은 그룹을 지어주어야 한다
