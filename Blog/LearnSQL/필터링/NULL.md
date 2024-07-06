
---

>[!info] `NULL` 은 값이 없는것이다.

`NULL`  은 다양한 경우가 있어 정의가 약간 불분명하다

- **해당사항 없음**<br>예를들어 `ATM` 에서 발생한 거래내역의 직원 ID 열

- **아직 알려지지 않은 값**<br>예를 들어 고객 테이블에 행이 생성될때 연방 ID 를 알수 없는 경우

- **정의되지 않는 값**<br>예를 들어 데이터베이스에 아직 추가되지 않은 제품의 계좌가 생성된 경우

`NULL` 로 작업할때 다음의 사항을 기억해야 한다

- **`NULL` 일수는 있지만, `NULL` 과 같을수 없다**
- **두개의 `NULL`  은 서로 같지 않다**

`NULL` 인지 확인하려면 `IS NULL` 을 사용해야 한다

```mysql
select rantal_id, customer_id
from rental
where return_date is null;
```

반면, `equl` 연산자를 사용하면 출력되지 않는다

```mysql
select rantal_id, customer_id
from rental
where return_date = null;
```

> [!warning] 데이터베이스는 위의 상황에서 `ERROR` 가 나지 않는다.<br>`null` 을 확인하는 조건을 구성할때 주의해야 할 사항이다.

다음은 `NULL` 이 아닌 데이터를 출력한다

```mysql
select rental_id, customer_id, return_date
from rental
where return_date is not null;
```

다음은 `2005년 5월 에서 8월` 사이에 반납되지 않은 모든 대여 정보를 찾는다.

```mysql
select rental_id, customer_id, return_date
from rental
where return_date IS NULL or return_date not between '2005-05-01' and '2005-09-01';
```

여기서 주의할점은 `return_date is null` 을 포함해야 한다는것이다.

`is null`  인 값도 `5월 에서 8월` 사이에 반납되지 않은 대여정보이기 때문이다.

이러한 함정을 잘 알고 처리할줄 알아야 한다.

