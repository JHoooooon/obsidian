
---

>[!info] 쿼리에서 반환된 결과 셋의 행은 특정 순서로 나열되지 않는다<br><br>`order by`  절을 사용하여 정렬하도록 지시해야 한다.

>[!info] `order by`   절은 원시 열 데이터 또는 열 데이터를 기반으로 표현식을 사용하여 결과셋을 정렬하는 매커니즘 이다.


```mysql
select 
	c.first_name,
	c.last_name,
	time(r.rental_date) rental_time
from customer c
	inner join rental r
	on c.customer_id = r.customer_id
where date(r.rental_date) = '2005-06-14'
```

이 예제를 실행해보면 다음처럼 정렬되지 않는 형식으로 출력된다.

```sh
first_name|last_name|rental_time|
----------+---------+-----------+
CATHERINE |CAMPBELL |   23:17:03|
JOYCE     |EDWARDS  |   23:16:26|
AMBER     |DIXON    |   23:42:56|
JEANETTE  |GREENE   |   23:54:46|
MINNIE    |ROMERO   |   23:00:34|
GWENDOLYN |MAY      |   23:16:27|
SONIA     |GREGORY  |   23:50:11|
MIRIAM    |MCKINNEY |   23:07:08|
CHARLES   |KOWALSKI |   23:54:34|
...
```

이때, `first_name` 기준으로 정렬되도록 하려면 `order by`  절을 사용한다.

```mysql
select 
	c.first_name,
	c.last_name,
	time(r.rental_date) rental_time
from customer c
	inner join rental r
	on c.customer_id = r.customer_id
where date(r.rental_date) = '2005-06-14'
order by c.first_name;
```

```sh
first_name|last_name|rental_time|
----------+---------+-----------+
AMBER     |DIXON    |   23:42:56|
CATHERINE |CAMPBELL |   23:17:03|
CHARLES   |KOWALSKI |   23:54:34|
DANIEL    |CABRAL   |   23:09:38|
ELMER     |NOE      |   22:55:13|
GWENDOLYN |MAY      |   23:16:27|
HERMAN    |DEVORE   |   23:35:09|
JEANETTE  |GREENE   |   23:54:46|
JEFFERY   |PINSON   |   22:53:33|
...
```

`A-Z` 순으로 정렬된것을 볼수 있다,
만약, `first_name`  이 같고 `last_name`  이 다르다면, `last_name`  역시 정렬할 필요가 있다.

이럴때 다음처럼 정렬한다.

```mysql
select 
	c.first_name,
	c.last_name,
	time(r.rental_date) rental_time
from customer c
	inner join rental r
	on c.customer_id = r.customer_id
where date(r.rental_date) = '2005-06-14'
order by c.first_name, c.last_name;
```

위의 예시는 `first_name`  을 먼저 정렬하고,  정렬된 `first_name` 을 기준으로 `last_name` 을 정렬하는 예이다.


### 내림차순 및 오름차순

정렬시 `asc` 및 `desc` 를 사용하여 `오름자순`, `내림차순`으로 정렬가능하다
기본값은 `asc`(`오름차순`) 이다.

```mysql
select c.first_name, c.last_name, time(r.rental_date) rental_time
from customer c
	inner join rental r
	on c.customer_id = r.customer_id
where date(r.rental_date) = '2005-06-14'
order by time(r.rental_date) desc;
```

이는 `time(r.rental_date)` 을 기준으로 `내림차순`  정렬되서 출력된다.

### 순서를 통한 정렬

`order by` 사용시 `select`
