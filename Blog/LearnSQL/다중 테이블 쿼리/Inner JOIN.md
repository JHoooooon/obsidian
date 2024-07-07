
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





