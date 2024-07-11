
조건식은 프로그램 실행 중 여러 경로 중 하나를 선택한다.

`cusotmer` 에는 다음의 `active` 열이 있음을 보자

```mysql
DESC customer;
```

```sh
Field      |Type             |Null|Key|
-----------+-----------------+----+---+
customer_id|smallint unsigned|NO  |PRI|
store_id   |tinyint unsigned |NO  |MUL|
first_name |varchar(45)      |NO  |   |
last_name  |varchar(45)      |NO  |MUL|
email      |varchar(50)      |YES |   |
address_id |smallint unsigned|NO  |MUL|
active     |tinyint(1)       |NO  |   |
create_date|datetime         |NO  |   |
last_update|timestamp        |YES |   |
```

조건식을 사용해서  고객 정보를 쿼리할때 활성을 표시하는 $1$ 과 비활성을 표시하는 $0$ 을 저장하는 `customer.active` 열을 포함할수도 있다.

```mysql
SELECT
	first_name,
	last_name,
	CASE
		WHEN active = 1 THEN 'ACTIVE'
		ELSE 'INACTIVE'
	END activity_type
FROM customer c;
```

```sh
first_name|last_name |activity_type|
----------+----------+-------------+
MARY      |SMITH     |ACTIVE       |
PATRICIA  |JOHNSON   |ACTIVE       |
LINDA     |WILLIAMS  |ACTIVE       |
BARBARA   |JONES     |ACTIVE       |
ELIZABETH |BROWN     |ACTIVE       |
JENNIFER  |DAVIS     |ACTIVE       |
MARIA     |MILLER    |ACTIVE       |
SUSAN     |WILSON    |ACTIVE       |
MARGARET  |MOORE     |ACTIVE       |
DOROTHY   |TAYLOR    |ACTIVE       |
LISA      |ANDERSON  |ACTIVE       |
NANCY     |THOMAS    |ACTIVE       |
KAREN     |JACKSON   |ACTIVE       |
BETTY     |WHITE     |ACTIVE       |
HELEN     |HARRIS    |ACTIVE       |
SANDRA    |MARTIN    |INACTIVE     |
DONNA     |THOMPSON  |ACTIVE       |
CAROL     |GARCIA    |ACTIVE       |
...
```

이를 통해 $1$ 과 $0$ 으로 이루어진 값을 문자열 `ACTIVE` 및 `INACTIVE` 로 변환할수 있다.

>[!info] `MySQL` 에는 `if()` 라는 내장함수가 있으며, 다른 프로그래밍 언어에서 제공하는 다른 함수들도 존재한다. <br> 단, `CASE` 표현식은 `if-then-else` 로직을 좀더 쉽게 사용하도록 설계되어 있다<br><br> 그리고 무엇보다 표준이다!!

### 검색된 CASE 표현식

`CASE`  표현식은 `검색된` (`Searched`) `CASE 표현식` 의 예제로 다음과 같다

```mysql
CASE
	WHEN C1 THEN E1
	WHEN C2 THEN E2
	...
	WHEN CN THEN EN
	[ELSE ED]
END
```

`CASE` 는 `WHEN` 절에서 `TRUE`  로 평가되면 해당 표현식을 반환한다.
`ELSE` 는 선택적으로 작성 가능하며, `WHEN` 절이 실패했을때 사용될 값이다.

```mysql
SELECT
	CASE
		WHEN c.name IN ('Children', 'Family', 'Sports', 'Animation')
			THEN 'ALL Ages'
		WHEN c.name = 'Horror'
			THEN 'Adult'
		WHEN c.name IN ('Music', 'Games' )
			THEN 'Teens'
		ELSE 'Other'
	END film_ratings
FROM category c;
```

```sh
film_ratings|
------------+
Other       |
ALL Ages    |
ALL Ages    |
Other       |
Other       |
Other       |
Other       |
ALL Ages    |
Other       |
...
```

카테고리에 따라 영화가 분류되었다.
다음은 서브쿼리를 사용하여 활성 고객에 대한 대여 횟수를 반환한다.

```mysql
SELECT
	c.first_name,
	c.last_name,
	CASE
	WHEN c.active = 0 THEN 0
	ELSE (
		SELECT
		COUNT(*)
		FROM rental r
		WHERE r.customer_id = c.customer_id
	)
	END num_rentals
FROM customer c
```

```sh
first_name|last_name |num_rentals|
----------+----------+-----------+
MARY      |SMITH     |         32|
PATRICIA  |JOHNSON   |         27|
LINDA     |WILLIAMS  |         26|
BARBARA   |JONES     |         22|
ELIZABETH |BROWN     |         38|
JENNIFER  |DAVIS     |         28|
MARIA     |MILLER    |         33|
SUSAN     |WILSON    |         24|
...
SANDRA    |MARTIN    |          0|
...
```

서브쿼리를 사용해서, 결과값을 반환할수 있음을 알게되었다.
이는 `customer_id` 를 그룹화하는 것 보다 더 효율적이다.

### 단순 CASE 표현식

단순 CASE 표현식은 검색된 CASE 표현식과 매우 유사하지만, 덜 유연하다.

```mysql
CASE V0
	WHEN V1 THEN E1
	WHEN V2 THEN E2
	...
	WHEN VN THEN EN
	[ELSE ED]
END
```

이는, `V0` 가 `V1` ~ `VN` 집합의 값이 같은지 확인하고, 같으면 해당 표현식을 반환, 아니면 `ELSE` 를 반환한다.

```mysql
SELECT
	CASE c.name
		WHEN 'Children' THEN 'ALL Ages'
		WHEN 'Family' THEN 'ALL Ages'
		WHEN 'Sports' THEN 'ALL Ages'
		WHEN 'Animation' THEN 'ALL Ages'
		WHEN 'Horror' THEN 'ALL Ages'
		WHEN 'Music' THEN 'ALL Ages'
		WHEN 'Games' THEN 'ALL Ages'
		ELSE 'Other'
	END film_ratings
FROM category c
```

이는 `CASE` 표현식보다 유연성이 떨어질수 있음을 볼 수 있다.

## CASE 표현식의 예

요일과 같이 한정된 값의 집합을 집계해야 할때도 있지만, 결과셋에 값 당 하나의 행 대신 값당 하나의 열이 있는 단일 행을 포함해야 할수도 있다

```mysql
SELECT
	MONTHNAME(r.rental_date),
	COUNT(*) num_rentals
FROM rental r
WHERE return_date BETWEEN '2005-05-01' AND '2005-08-01'
GROUP BY MONTHNAME(rental_date)
```

```sh
MONTHNAME(r.rental_date)|num_rentals|
------------------------+-----------+
May                     |       1156|
June                    |       2311|
July                    |       4187|
```

이를 단일 데이터 행도 반환하도록 지시받았다고 하자.
이 결과를 단일 행으로 변환하려면 세 개의 열을 만들고 각 열에서 해당 월에 해당하는 행만 합산한다.

```mysql
SELECT
SUM(
	CASE
		WHEN MONTHNAME(rental_date) = 'May'
			THEN 1
		ELSE 0
	END
) May_rental,
SUM(
	CASE
		WHEN MONTHNAME(rental_date) = 'June'
			THEN 1
		ELSE 0
	END
) June_rental,
SUM(
	CASE
		WHEN MONTHNAME(rental_date) = 'July'
			THEN 1
		ELSE 0
	END
) July_rental
FROM rental r
WHERE return_date BETWEEN '2005-05-01' AND '2005-08-01'
```

```mysql
May_rental|June_rental|July_rental|
----------+-----------+-----------+
      1156|       2311|       4187|
```

모든 행을 합산하면 각 열은 해당 월에 개설된 계좌수를 반환한다.

>[!info] 다른 DB 에서는 `PIVOT` 으로 이러한 기능을 제공한다.<br><br> 하지만 `MySQL` 에는 `PIVOT` 기능이 없는듯하다.

### 존재 여부 확인

수량과 관계없이 두 엔터티 간에 관계가 있는지 여부를 확인한다.

```mysql
SELECT
	a.first_name,
	a.last_name,	
	CASE
		WHEN EXISTS (
			SELECT 1
			FROM film_actor fa	
				INNER JOIN film f
				ON fa.film_id = f.film_id
			WHERE fa.actior_id = a.actor_id
				AND f.rating = 'G'
		) THEN 'Y'
		ELSE 'N'
	END g_actor,
	CASE
		WHEN EXISTS (
			SELECT 1
			FROM film_actor fa	
				INNER JOIN film f
				ON fa.film_id = f.film_id
			WHERE fa.actior_id = a.actor_id
				AND f.rating = 'NC-17'
		) THEN 'Y'
		ELSE 'N'
	END nc17_actor,
	CASE
		WHEN EXISTS (
			SELECT 1
			FROM film_actor fa	
				INNER JOIN film f
				ON fa.film_id = f.film_id
			WHERE fa.actior_id = a.actor_id
				AND f.rating = 'PG'
		) THEN 'Y'
		ELSE 'N'
	END pg_actor,
FROM actor a
WHERE a.last_name LIKE 'S%' OR a.first_anem LIKE 'S%';
```

```sh
first_name|last_name  |g_actor|nc17_actor|pg_actor|
----------+-----------+-------+----------+--------+
JOE       |SWANK      |Y      |Y         |Y       |
SANDRA    |KILMER     |Y      |Y         |Y       |
CAMERON   |STREEP     |Y      |Y         |Y       |
SANDRA    |PECK       |Y      |Y         |Y       |
SISSY     |SOBIESKI   |Y      |N         |Y       |
NICK      |STALLONE   |Y      |Y         |Y       |
SEAN      |WILLIAMS   |Y      |Y         |Y       |
...
```

다음은 얼마나 많은 행이 있는지 확인하는데도 사용가능하다

```mysql
SELECT
	f.title,
	CASE (
			SELECT COUNT(*)
			FROM inventory i
			WHERE i.film_id = f.film_id
		)
		WHEN 0 THEN 'Out Of stock'
		WHEN 1 THEN 'Scarce'
		WHEN 2 THEN 'Scarce'
		WHEN 3 THEN 'Available'
		WHEN 4 THEN 'Available'
		ELSE 'Common'
	END film_availability
FROM film f
```

```sh
title                      |film_availability|
---------------------------+-----------------+
ACADEMY DINOSAUR           |Common           |
ACE GOLDFINGER             |Available        |
ADAPTATION HOLES           |Available        |
AFFAIR PREJUDICE           |Common           |
AFRICAN EGG                |Available        |
AGENT TRUMAN               |Common           |
AIRPLANE SIERRA            |Common           |
AIRPORT POLLOCK            |Available        |
ALABAMA DEVIL              |Common           |
...
```

이는 `film` 의 `inventory` 개수가 얼마냐에 따라 그 값이 결정된다.
`5` 보다 크면 공통적으로 많이 사용되는 `film` 이다. 

`CASE` 문을 사용할때 `0` 으로 값을 나누지 않도록 분기처리하기도 한다.

>[!warning] $0$ 

```mysql
SELECT 1 / 0; --- NULL
```

```mysql
SELECT
	c.first_name,
	c.last_name
	SUM(p.amount) tot_payment_amt,
	COUNT(p.amount) num_payment,
	SUM(p.amount) /
	CASE
		WHEN COUNT(p.amount) = 0
			THEN 1
		ELSE COUNT(p.amount)
	END avg_payment
FROM customer c
LEFT OUTER JOIN payment p
	ON c.customer_id = p.customer_id
GROUP BY c.first_name , c.last_name;
```

```sh
first_name|last_name |tot_payment_amt|num_payment|avg_payment|
----------+----------+---------------+-----------+-----------+
MARY      |SMITH     |         118.68|         32|   3.708750|
PATRICIA  |JOHNSON   |         128.73|         27|   4.767778|
LINDA     |WILLIAMS  |         135.74|         26|   5.220769|
BARBARA   |JONES     |          81.78|         22|   3.717273|
ELIZABETH |BROWN     |         144.62|         38|   3.805789|
...
```


