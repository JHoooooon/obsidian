
>[!info] `개발자를 위한 레디스` 의 내용과 `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

이전의 [[Sorted SET]] 에서 `정렬된 SET` 을 사용할수 있었다.

하지만, 때때로 `Redis` 의 `LIST` 또는 `SET` 을 `copy` 하여 정렬된 `Sorted SET` 을 얻어야 할때가 있다.

`Redis` 는 이러한 상황을 가정하여 편리한 명령어인 `SORT` 를 제공한다.

`SORT` 명령어는 $O(N + M \times log( M ))$ 시간 복잡도를 가진다.
`N` 은 `LIST` 혹은 `SET` 의 요소의 숫자이며, `M` 은 반환하는 요소의 개수이다. 

`SORT` 연산의 시간복잡도를 본다면 많은 양의 데이터를 정렬할때 `Redis`서버의 성능이 저하될수 있음을 알수 있다.

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

>[!info] ALPHA OPTION
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

>[!info] DESC OPTION
```sh
127.0.0.1:6379> SORT "user:123:favorite_restaurant_ids" DESC
1) "455"
2) "365"
3) "333"
4) "200"
5) "104"
```

`LIMIT` 옵션을 추가하여, `offset` 과 받을 요소의 `count` 를 지정할수 있다.

>[!info] LIMIT OPTION
```sh
127.0.0.1:6379> SORT "user:123:favorite_restaurants" ALPHA LIMIT 0 3
1) "Burger King"
2) "Dunkin Donuts"
3) "KFC"
```

만약, 자기 자신의 값으로 요소정렬을 원치 않고, 외부의 `key` 값을 사용하여 요소를 선택하고, 외부의 `key` 값에 매칭된 값으로 `SORT` 하고 싶다면 다음처럼 할수도 있다.

>[!info] BY OPTION

```sh
127.0.0.1:6379> SET "restaurant_rating_200" 4.3 
127.0.0.1:6379> SET "restaurant_rating_365" 4.0 
127.0.0.1:6379> SET "restaurant_rating_104" 4.8 
127.0.0.1:6379> SET "restaurant_rating_455" 4.7 
127.0.0.1:6379> SET "restaurant_rating_333" 4.6 
127.0.0.1:6379> SORT "user:123:favorite_restaurant_ids" BY restaurant_rating_* DESC 
1) "104" 
2) "455" 
3) "333" 
4) "200" 
5) "365"
```

이는 `restaurant_rating_*` 부분의 `200`, `365`, `104`, `455`, `333` 의 값을 사용하여,
`user:123:favorite_restaurant_ids` 에서 `id` 값을 가져온다.

그리고 `DESC` 를 사용하여, `restaurant_rating_*` 의 값들을 기준으로 오름차순으로 정렬한다.

위처럼 외부 `key` 와 `value`  를 사용해서 처리하는건 매우 유용해보인다.
다른 외부 값의 `value` 값으로 치환할수 없을까?

정렬을 위한 옵션에 `GET` 을 사용하면 가능하다 

>[!info]  GET key OPTION
```sh
127.0.0.1:6379> SET "restaurant_name_200" "Ruby Tuesday" 
127.0.0.1:6379> SET "restaurant_name_365" "TGI Friday" 
127.0.0.1:6379> SET "restaurant_name_104" "Applebee's" 
127.0.0.1:6379> SET "restaurant_name_455" "Red Lobster" 
127.0.0.1:6379> SET "restaurant_name_333" "Boiling Crab" 
127.0.0.1:6379> SORT "user:123:favorite_restaurant_ids" BY restaurant_rating_* DESC GET restaurant_name_* 
1) "Applebee's" 
2) "Red Lobster" 
3) "Boiling Crab" 
4) "Ruby Tuesday" 
5) "TGI Friday" 
```

이는 `restaurant_rating_*` 의 모든 `200`, `365`, `104`, `455`, `333` 값을 사용하여, `ids` 를 가져온다

가져온 `ids` 값을 `DESC` 로 정렬후 정렬된 `ids` 값을 `restuarant_name_*` 의 `*` 부분에 차례대로 대입하여, 해당하는 값을 가져온다.

다음은 이렇게 만들어진 새로운 정렬된 `SET` 을 반환과 동시에 `destination key`  로 값을 저장하는 명령어인 `STORE` 옵션이다.

>[!info] STORE destination_key OPTION
```sh
127.0.0.1:6379>SORT "user:123:favorite_restaurant_ids" BY restaurant_rating_* DESC GET restaurant_name_* STORE user:123:favorite_restaurant_names:sort_by_rating 
(integer) 5
```


