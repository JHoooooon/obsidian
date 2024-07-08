
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

```mysql
SELECT 
	'CUST' type,
	c.first_name, 
	c.last_name
FROM customer c
UNION ALL
SELECT 
	'ACTR' type,
	a.first_name,
	a.last_name
FROM actor a;
```

이 쿼리는 `customer` 와 `actor` 의 정보를 합친 테이블을 쿼리한다
[[#집합연산의 규칙]] 에서와 같이


