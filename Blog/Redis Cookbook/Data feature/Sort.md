
이전의 [[Sorted SET]] 에서 `정렬된 SET` 을 사용할수 있었다.

하지만, 때때로 `Redis` 의 `LIST` 또는 `SET` 을 `copy` 하여 정렬된 `Sorted SET` 을 얻어야 할때가 있다.

`Redis` 는 이러한 상황을 가정하여 편리한 명령어인 `SORT` 를 제공한다.

---

다음은 `SORT` `key `로 오름차순으로 정렬하는 예시이다

>[!info] SORT
```sh
127.0.0.1:6379> SADD "user:123:favorite_restaurant_ids" 200 365 104 455 333
(integer) 5
127.0.0.1:6379> SORT "user:123:favorite_restaurant_ids"
1) "104"
2) "200"
3) "333"
4) "365"
5) "455"
```

만약, `non-numeric`요소이고, 그러한 요소를 사전식순으로  정렬하고 싶다면,
`ALPHA`  옵션을 추가할수 있다.

>[!info] ALPHA option
```sh
127.0.0.1:6379> SADD "user:123:favorite_restaurants" "Dunkin Donuts" "Subway" "KFC" "Burger King" "Wendy's" 
(integer) 5 

127.0.0.1:6379> SORT "user:123:favorite_restaurants" ALPHA 
1) "Burger King" 
2) "Dunkin Donuts" 
3) "KFC" 
4) "Subway" 
5) "Wendy's"
```

`DESC` 옵션을 추가하여, `SORT` 명령에서 내림차순으로 정렬할수 있다.
기본값으로 `ASC` 이다.

>[!info] DESC option
```sh
127.0.0.1:6379> SORT "user:123:favorite_restaurant_ids" DESC
1) "455"
2) "365"
3) "333"
4) "200"
5) "104"
```

`LIMIT` 옵션을 추가하여, `offset` 과 받을 요소의 `count` 를 지정할수 있다.

>[!info] LIMIT option
```sh
127.0.0.1:6379> SORT "user:123:favorite_restaurants" ALPHA LIMIT 0 3
1) "Burger King"
2) "Dunkin Donuts"
3) "KFC"
```

만약, 자기 자신의 값에 의해 `SORT` 요소 를 원치 않고, 외부의 `key` 값을 사용하여 `SORT` 하고 싶다면 다음처럼 할수도 있다.

>[!info] BY option