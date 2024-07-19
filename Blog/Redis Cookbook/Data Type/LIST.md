>[!info] `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`LIST` 데이터 타입은 어플리케이션 개발에서 매우 유용하게 사용가능한 타입이다.
`LIST` 는 `Objects` 를 나열할수도 있으며, `stack` 또는 `queue` 로써 사용할수도 있다.

`LIST` 는 자료구조에서 말하는 `Doubly Linked List`(`이중 연결 리스트`) 와 비슷하다고 말한다.

---

다음은 `favorit_restaurants` `List` 에 두개의 `value` 값을 리스트의 왼쪽 끝에 `PUSH` 한다. 

>[!info] LPUSH
```sh
127.0.0.1:6379> LPUSH favorite_restaurants "PF Chang's" "Olive Garden"
(integer) 2
```

다음은 모든 `restaurants` 의 이름을 `LIST` 에서 가져온다

>[!info] LRANGE
```sh
127.0.0.1:6379> LRANGE favorite_restaurants 0 -1
1) "Olive Garden"
2) "PF Chang's"
```

이는 `0` 번째 부터 `-1` (`-1` 은 맨 끝을 뜻한다) 까지 `LIST` 의 값을 나열한다.
다음은, 오른쪽 끝에 `value` 값을 `PUSH` 한다 

>[!info] RPUSH
```
127.0.0.1:6379> RPUSH favorite_restaurants "Outback Steakhouse" "Red Lobster"
(integer) 4
127.0.0.1:6379> LRANGE favorite_restaurants 0 -1
1) "Olive Garden"
2) "PF Chang's"
3) "Outback Steakhouse"
4) "Red Lobster"
```

다음은 `LINDEX` 를 사용하여, 해당하는 `INDEX` 의 `value` 를 찾는다

>[!info] LINDEX
```sh
127.0.0.1:6379> LINDEX favorite_restaurants 3
"Outback Steakhouse"
```

앞에서 말했지만, `LIST` 는 `Dobuly Linked List` 자료구조와 비슷한 패턴을 가진다.
이는 새로운 원소를 다음의 `3` 가 `command` 를 사용하여 넣을수 있음을 알수 있다.

- **LPUSH**: 원소를 왼쪽 끝으로 `LIST` 앞에 추가한다
- **RPUSH**: 원소를 오른쪽 끝으로 `LIST` 뒤에 추가한다
- **LINSERT**: `LIST` 내의 `중점이 되는 원소` 앞 혹은 뒤로 `새로운 원소` 를 추가한다

`LPUSH`, `RPUSH`, `LINSERT` 는 추가된 이후 추가한 원소의 개수 만큼의 숫자를 리턴한다. 
보면 알겠지만, `LIST` 를 초기화하는 작업은 필요치 않다.

그저 `KEY` 값을 `Empty List` 에 삽입하면 자동적으로, `LIST` 가 생성된다.
또한 `KEY` 를 가진 `LIST` 가 비어있다면 `Redis` 가 알아서 삭제 처리 한다.
그러므로, 명시적으로 `delete` 하지 않는다.

>[!note] 만약, `KEY` 가 존재하는 `LIST` 에만 요소를 넣고 싶다면,<br>`LPUSHX`, `RPUSHX` `command` 를 사용할수 있다. 

`LIST` 에서 값을 제거하고 싶다면 `LPOP` 혹은 `RPOP` `command` 를 사용한다.

>[!info] LPOP, RPOP
```sh
127.0.0.1:6379> LPOP favorite_restaurants
"Olive Garden"
127.0.0.1:6379> RPOP favorite_restaurants
"Red Lobster"
127.0.0.1:6379> LPOP non_existent
(nil)
```

`LTRIM` 은 여러 요소를 한꺼번에 제거하는데 사용된다.
`start`, `end` 값을 지정하여 해당 범위에 속하는 요소만 남기고 삭제한다.
다음은 `1` 부터 `-1` 까지의 요소만 남기고 전부 삭제한다.

>[!info] LTRIM
```sh
127.0.0.1:6379> LRANGE favorite_restaurants 0 -1
1) "PF Chang's"
2) "Indian Tandoor"
3) "Outback Steakhouse"
127.0.0.1:6379> LTRIM favorite_restaurants 1 -1
OK
127.0.0.1:6379> LRANGE favorite_restaurants 0 -1
1) "Indian Tandoor"
2) "Outback Steakhouse"
```

`LSET` 명령은 지정한 `LIST` 안에 요소의 값을 설정하는데 사용된다.

>[!info] LSET
```sh
127.0.0.1:6379> LSET favorite_restaurants 1 "Longhorn Steakhouse"
OK
127.0.0.1:6379> LRANGE favorite_restaurants 0 -1
1) "Indian Tandoor"
2) "Longhorn Steakhouse"
```

`LPOP` 과 `RPOP` 명령을 `blockking` 하는 명령역시 존재한다.
`BLPOP` 과 `BRPOP` 으로, 이 두 명령은 `LIST` 데이터 구조에서 사용하는 `blocking` 