
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

#### 파생 테이블

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
`view`  에 대한 쿼리를 실행하면 쿼리가 뷰 정의와 합쳐져 실행할 쵲







