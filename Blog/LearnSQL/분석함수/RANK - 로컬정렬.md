
분석기능을 위해 결괏세을 데이터 윈도우로 분할하는 것과 함께 정렬 순서를 지정할 수도 있다.

예를 들어 판매량이 가장 높은 달에 값 `1` 이 주어지는 각 월의 순위 번호를 정의하려면, 순위에 사용할 열을 지정해야 한다.

```mysql
SELECT
	QUARTER(payment_date) quarter,
	MONTHNAME(payment_date) payment_mn,
	SUM(amount) monthly_sales,
	RANK() OVER (
		ORDER BY SUM(amount) DESC
	) sales_rank
FROM payment p
WHERE 
	YEAR(payment_date) = '2005'
GROUP BY
	QUARTER(payment_date),
	MONTHNAME(payment_date)
ORDER BY 1, MONTHNAEM(payment_date);

```

```sh
quarter|payment_mn|monthly_sales|sales_rank|
-------+----------+-------------+----------+
      2|June      |      9629.89|         3|
      2|May       |      4823.44|         4|
      3|August    |     24070.14|         2|
      3|July      |     28368.91|         1|
```

>[!warning] 다중 order by 절
>위의 예시에서 `order by` 절이 2번 사용된다. 하나는 쿼리 끝에서 결과셋을 정렬하는것이고, 하나는 `RANK` 에서 사용된다.
>
>하나 이상의 `ORDER BY`  절과 함께 분석 함수를 사용했더라도 결과를 원하는 방식으로 정렬하고자 한다면 쿼리 끝에 `ORDER BY` 절이 필요하다는점을 알아야 한다.

경우에 따라 `PARTITION BY` 와 함께 사용가능하다.

```mysql
SELECT
	QUARTER(payment_date) quarter,
	MONTHNAME(payment_date) payment_mn,
	SUM(amount) monthly_sales,
	RANK() OVER (
		PARTITION BY QUARTER(payment_date)
		ORDER BY SUM(amount) DESC
	) sales_rank
FROM payment p
WHERE YEAR(payment_date) = '2005'
GROUP BY
	QUARTER(payment_date),
	MONTHNAME(payment_date)
ORDER BY 1, MONTHNAME(payment_date);
```

```sh
quarter|payment_mn|monthly_sales|sales_rank|
-------+----------+-------------+----------+
      2|June      |      9629.89|         1|
      2|May       |      4823.44|         2|
      3|August    |     24070.14|         2|
      3|July      |     28368.91|         1|
```

이는 분기별로 순위를 매기는것을 볼수 있다.







