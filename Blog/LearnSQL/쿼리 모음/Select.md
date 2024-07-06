>[!info] [[쿼리 역학]] 의 소주제이다.
---

### Select 절

`select` 절은 `모든 열 중에 쿼리 결과에 포함할 열을 결정하는 역할` 을 한다
이는 다음과 같은 항목이 `select` 절에 포함될수 있다.

---
- 숫자 또는 문자여 같은 리터럴
- transaction.amount * -1 과 같은 표현식
- ROUND(transaction.amount, 2) 와 같은 내장 함수 호출
- 사용자 정의 함수 호출
---
다음은 테이블열, 리터럴, 표현식 및 내장 함수 호출을 사용하는 예제이다

```mysql
select 
	'COMMON' language_usage,
	language_id * 3.1415927 lang_pi_value,
	upper(name) language_name
from language;
	
```

```sh
language_usage|lang_pi_value|language_name|
--------------+-------------+-------------+
COMMON        |    3.1415927|ENGLISH      |
COMMON        |    6.2831854|ITALIAN      |
COMMON        |    9.4247781|JAPANESE     |
COMMON        |   12.5663708|MANDARIN     |
COMMON        |   15.7079635|FRENCH       |
COMMON        |   18.8495562|GERMAN       |
```
``
이러한 특성을 활용한다면 `from` 절 없이 출력도 가능하다

```mysql
select
	version(),
	user(),
	database();
```

```sh
version()|user()                  |database()|
---------+------------------------+----------+
8.0.30   |avnadmin@121.167.242.152|sakila    |
```

모든 열을 출력하고 싶다면 `*`  을 사용하면 테이블의 모든 열이 출력된다

```mysql
select * from language;
```

```sh
language_id|name    |last_update        |
-----------+--------+-------------------+
          1|English |2006-02-15 05:02:19|
          2|Italian |2006-02-15 05:02:19|
          3|Japanese|2006-02-15 05:02:19|
          4|Mandarin|2006-02-15 05:02:19|
          5|French  |2006-02-15 05:02:19|
          6|German  |2006-02-15 05:02:19|
```

이는 다음과 같다

```mysql
select
	language_id ,
	name ,
	last_update
from `language` l;
```

```sh
language_id|name    |last_update        |
-----------+--------+-------------------+
          1|English |2006-02-15 05:02:19|
          2|Italian |2006-02-15 05:02:19|
          3|Japanese|2006-02-15 05:02:19|
          4|Mandarin|2006-02-15 05:02:19|
          5|French  |2006-02-15 05:02:19|
          6|German  |2006-02-15 05:02:19|
```
``
### column alias

열에 별칭(`alias`) 를 추가할수 있다
이는 고유한 레이블을 지정한다고 보면 된다.

```mysql
select
	language_id ,
	'COMMON' language_usage ,
	upper(name) language_name
from `language` l
```

```sh
language_id|language_usage|language_name|
-----------+--------------+------------+
          1|COMMON        |ENGLISH     |
          2|COMMON        |ITALIAN     |
          3|COMMON        |JAPANESE    |
          4|COMMON        |MANDARIN    |
          5|COMMON        |FRENCH      |
          6|COMMON        |GERMAN      |
```

위에서 `alias` 는 `language_usage`  와 `language_name` 이다.
사실 `alias`  는 `as` 키워드를 사용해서 처리하는것이 더 두드러져 보이지만, 편의상 생략하는 경우도 많다

>[!note] `as` 키워드는 선택의 문제이다.<br>회사 `convention` 에 따르자.

### 중복 제거

쿼리가 중복된 데이터 행을 반환할 수 있다.
다음은 영화에 출연한 모든 배우의 `ID` 를 검색하는 쿼리이다.

```mysql
select actor_id from film_actor order by actor_id;
```

```sh
actor_id|
--------+
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       1|
       2|
       2|
       2|
       2|
       2|
	 ...|
	 200|
```

한명의 배우가 여러 영화를 찍었으니 `actor_id`  가 여러개 나오는건 당연하다.
이러한 경우 중복된 결과를 없애주는 방법이 필요하다

`distinct` 키워드를 사용하면 쉽게 해결할수 있다

>[!note] `distinct` 키워드는 하나의 고유한 집합을 나타낸다.

```mysql
select distinct acttor_id from film_actor order by actor_id;
```

```sh
actor_id|
--------+
       1|
       2|
       3|
       4|
       5|
       6|
       7|
       8|
       9|
      10|
      11|
      12|
      13|
      14|
      15|
      16|
      17|
      18|
      19|
      20|
      21|
      22|
      23|
      24|
	 ...|
	 200|
```

중복된 결과 없이 하나의 `actor_id` 목록이 나온다.

>[!note] 중복값이 없는것이 확실할때는 `all` 키워드를 지정해도 된다.<br>`all` 키워드는 기본값이므로 굳이 명시적으로 지정안해도 상관없다.

>[!warning] `distinct` 결과를 생성할때 데이터를 정렬해야 하므로 결과셋의 용량이 클때는 시간이 오래걸릴수 있다. 중복이 없는지 확인하려고 `distinct` 를 사용하는 함정에 빠지는 대신, 데이터를 이해하고 중복 여부를 파악하는것이 중요하다.




