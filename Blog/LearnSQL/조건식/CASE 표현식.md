
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


