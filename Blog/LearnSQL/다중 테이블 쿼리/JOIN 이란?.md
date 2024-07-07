
---

>[!info] 단일 테이블에서의 쿼리도 적지 않지만, 대부분의 쿼리는 `2` 개, `3` 개 또는 그 이상의 테이블이 필요하다

```mysql
desc customer;
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

```mysql
desc address;
```

```sh
Field      |Type             |Null|Key|
-----------+-----------------+----+---+
address_id |smallint unsigned|NO  |PRI|
address    |varchar(50)      |NO  |   |
address2   |varchar(50)      |YES |   |
district   |varchar(20)      |NO  |   |
city_id    |smallint unsigned|NO  |MUL|
postal_code|varchar(10)      |YES |   |
phone      |varchar(20)      |NO  |   |
location   |geometry         |NO  |MUL|
last_update|timestamp        |NO  |   |
```

주소와 함께 각 고객의 성과 이름을 검색한다.

이는 `customer` 의 `address_id` 와 `address` 의 `address_id` 를 기준으로해서, `2` 테이블을 합치고, 원하는 `field` 를 `select` 하면 된다.

이렇게 `customer` 의 `address_id` 를 `address` 의 `Forgien Key` 라고 하며, 이러한 `Forgien Key` 를 사용하여 연결수단으로 사용한다. 

이러한 연결수단을 사용하여 테이블을 합치는 방법을 `JOIN` 이라 한다.

## 데카르트곱

`JOIN` 하는 방법은 여러가지가 있는데, 가장 쉬운 것은 다음처럼 `from` 절에서 `JOIN` 하는방법이다.

```mysql

SELECT
	c.first_name,
	c.last_name,
	a.address
FROM
	customer c JOIN address a;
```

```sh
first_name|last_name   |address          |
----------+------------+-----------------+
AUSTIN    |CINTRON     |47 MySakila Drive|
WADE      |DELVALLE    |47 MySakila Drive|
FREDDIE   |DUGGAN      |47 MySakila Drive|
ENRIQUE   |FORSYTHE    |47 MySakila Drive|
TERRENCE  |GUNDERSON   |47 MySakila Drive|
EDUARDO   |HIATT       |47 MySakila Drive|
RENE      |MCALISTER   |47 MySakila Drive|
TERRANCE  |ROUSH       |47 MySakila Drive|
KENT      |ARSENAULT   |47 MySakila Drive|
...
```

총 검색된 `row` 수는 `361,197` 이다.
이는 `address` 의 총 `row` `603` 과 `customer` 의 총 `row` `599` 을 곱한 값이다.

`603 * 599 = 361,197`

이는 `query` 가 어떻게 `JOIN` 되는지 알수 있는 중요한 단서이다.

> [!info] 두 테이블을 조인할때, 모든 `순열`  을 생성해야 한다. <br><br> 두 쌍의 모든 경우의 수를 알아야 하므로, `데카르트곱` (`cartesian product`) 을 사용한다.

이를 사용하여 테이블 쌍의 모든 경우의 순열을 생성하는 원리이다.
이러한 유형의 `JOIN` 을 `교차조인`(`cross join`) 이라 한다.





