
---

>[!info] 데이터에서 결과를 검색하기 전에 데이터베이스 서버가 데이터를 정제하는 흐름을 찾아볼 수 있다.<br><br>이러한 메커니즘 중 하나는 데이터를 열 값 별로 그룹화는 `group by` 절이다

예를 들어 40편 이상의 영화를 대여한 모든 고객을 찾는다고 가정해보자

`rental` 테이블은 `16.044` 개의 행을 모두 살펴보는 대신 **서버가 고객별로 모든 대여 내역을 그룹화**하고 각 **고객의 대여 횟수를 계산한 다음 대여 횟수가 `40` 이상인 고객만 반환하도록 지시**하는 쿼리를 작성할 수 있다.

```mysql
select c.first_name, c.last_name, count(*)
from customer c
	inner join rental r
	on c.customer_id = r.customer_id
group by c.first_name, c.last_name
having count(*) >= 40;
```

`group by c.first_name, c.last_name` 구문은 **`c.first_name` 과 `c.last_name` 을 가진 고객을 그룹화** 하라는 이야기이다.

`having count(*) >= 40;` 구문은 **`c.first_name`, `c.last_name` 으로 그룹화된 고객의 개수가 `40` 과 같거나 큰 조건 필터**를 가진다

>[!warning] `having` 절은 `group by`  를 통해 그룹화된 테이블에서 조건필터를 사용해서 필터링할때 사용하는 절이다.<br><br>`where`  절은 그룹화된 테이블에서 조건 필터를 사용할수 없다.

>[!info]  이후 `group by` 절과 `having`  절은 챕터를 할애해서 자세히 설명한다고 말한다.

