
분석함수는 아니지만, 데이터 윈도우 내에서 행 그룹과 함께 작동한다.
`GROUP_CONCAT()` 함수는 열 값 집합을 하나의 구분된 문자열로 피벗하는데 사용되며, 이는 `XML` 또는 `JSON` 문서를 생성하기 위해 결과셋을 변별하는 편리한 방법이다.

다음은 쉼표로 구분된 배우 목록을 생성하는데 사용된다.

```mysql
SELECT
	f.title,
	GROUP_CONCAT(
		a.last_name 
			ORDER BY a.last_name SEPARATOR  ', '
	) actors
FROM actor a
INNER JOIN film_actor fa
ON a.actor_id = fa.actor_id
INNER JOIN film f
ON fa.film_id = f.film_id
GROUP BY f.title
HAVING COUNT(*) = 3;
```

```sh
title                 |actors                  
----------------------+------------------------
ANNIE IDENTITY        |GRANT, KEITEL, MCQUEEN  
ANYTHING SAVANNAH     |MONROE, SWANK, WEST     
ARK RIDGEMONT         |BAILEY, DEGENERES, GOLDB
ARSENIC INDEPENDENCE  |ALLEN, KILMER, REYNOLDS 
AUTUMN CROW           |HUDSON, PITT, TAUTOU    
BEAR GRACELAND        |CRONYN, DAMON, HARRIS   
...
```

