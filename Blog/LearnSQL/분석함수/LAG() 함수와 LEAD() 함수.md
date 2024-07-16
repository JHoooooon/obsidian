
데이터 윈도우에 대한 합계 및 평균 계산과 더불어 또 다른 일반적인 보고 작업으로는 한 행의 값을 다른 행과 비교하는 경우이다.

- **LAG(ref_field, row_num)**: 이전 행에서 열값을 검색
- **LEAD(ref_field, row_num)**: 이후 행에서 열값을 검색

```mysql
SELECT
	YEARWEEK(payment_date) payment_week,
	SUM(amount) week_tot,
	LAG(SUM(amount), 1)
		OVER (ORDER BY YEARWEEK(payment_date)) prev_wk_tot,
	LEAD(SUM(amount), 1)
		OVER (ORDER BY YEARWEEK(payment_date)) next_wk_tot
FROM payment p
GROUP BY YEARWEEK(payment_date)
ORDER BY 1;
```

```sh
payment_week|week_tot|prev_wk_tot|next_wk_tot|
------------+--------+-----------+-----------+
      200521| 2846.19|           |    1977.25|
      200522| 1977.25|    2846.19|    5603.43|
      200524| 5603.43|    1977.25|    4026.46|
      200525| 4026.46|    5603.43|    8490.83|
      200527| 8490.83|    4026.46|    5982.64|
      200528| 5982.64|    8490.83|   11027.23|
      200530|11027.23|    5982.64|    8412.07|
      200531| 8412.07|   11027.23|   10619.11|
      200533|10619.11|    8412.07|    7907.17|
      200534| 7907.17|   10619.11|     514.18|
      200607|  514.18|    7907.17|           |
```

`prev_wk_tot` 을 보면 현재 행에서 이전 행의 `week_tot` 을 참조 하며,
`next_wk_tot` 에서는 현재 생에서 이후 행의 `week_tot` 을 참조한다.

두번째 매개변수는 기본값이 `1` 이다.
다음은 이전 주와 백분율 차이를 생성한다.

```mysql
SELECT
	YEARWEEK(payment_date) payment_week,
	SUM(amount) week_tot,
	ROUND(
		(SUM(amount) - LAG(SUM(amount), 1)
			OVER (ORDER BY YEARWEEK(payment_date))) /
		LAG(SUM(amount), 1)
			OVER (ORDER BY YEARWEEK(payment_date)) *
		100
	, 1) pct_diff
FROM payment p
GROUP BY YEARWEEK(payment_date)
ORDER BY 1;
```

```sh
payment_week|week_tot|pct_diff|
------------+--------+--------+
      200521| 2846.19|        |
      200522| 1977.25|   -30.5|
      200524| 5603.43|   183.4|
      200525| 4026.46|   -28.1|
      200527| 8490.83|   110.9|
      200528| 5982.64|   -29.5|
      200530|11027.23|    84.3|
      200531| 8412.07|   -23.7|
      200533|10619.11|    26.2|
      200534| 7907.17|   -25.5|
      200607|  514.18|   -93.5|
```





