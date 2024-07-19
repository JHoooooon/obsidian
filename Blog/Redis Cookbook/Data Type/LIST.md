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
`BLPOP` 과 `BRPOP` 으로, 이 두 명령은 `LIST` 데이터 구조에서 사용하는 `blocking` 연산이다.

`blocking` 하는 연산이 왜 필요한가 했는데 다음의 `3` 가지 이유가 있다.

- **블로킹 연산**: `LIST` 가 비어있을때 즉시 반환하지 않고 대기
- **타입아웃 설정**: `대기 시간` 을 지정
- **여러 리스트 모니터링**: 한번에 여러 리스트를 모니터링

여기에서 `blocking` 연산에서 `LIST` 가 비어있을때 즉시 반환하지 않고 대기 하는 것이 중요하다.
앞에서 이미 말했듯, `LIST` 는 비어있는 `key` 가 있다면 알아서 `remove` 처리한다.

그렇기에, `BLPOP` 혹은 `BRPOP` 같은 경우 `LIST` 가 비어있지 않다면 일반적인 `LPOP` 혹은 `RPOP` 처럼 행동하지만, `마지막 원소` 가 `POP` 되는 순간 지정한 시간 만큼 대기한다.

이 대기하는 순간 `LIST` 에 새로운 값을 추가하여 `LIST` 가 비어있지 않도록 보장하기 위한 방식으로 사용가능하며, 다음처럼 `zero-timeout` 을 지정할수 있다.

>[!info] 대기시간으로 `0` 을 지정할수 있는데 이를 `zero-timeout` 이라 한다.

`0` 은 영원히 기다리는것을 의미하며, 이 기능은 작업자가 새 작업을 할당할때까지 기다리는 작업에 유용하다.

다음을 보자

>[!info] BLPOP
```sh
worker-1> BRPOP job_queue 0 
worker-2> BRPOP job_queue 0
```

 `redis-cli` 에 `2` 개의 `worker` 를 생성하여 각 `worker` 당 `BRPOP` 명령을 내리며 `zero-timeout` 을 설정한다. 

그럼 `2` 개의 `worker` 는 지속적인 대기 상태가 된다.
대기 상태에서 `dispatcher` 가 `job_queue` 에 `LPUSH`  한다.

```sh
dispatcher> LPUSH job_queue job1
```

그러면 대기중인 `worker-1` 에서 이 `job1` 을 받아 처리하고 대기가 풀린다.

```sh
worker-1> BRPOP job_queue 0 
1) "job_queue" 
2) "job1" 

worker-1>
```

이번에는 `job2`, `job3` 를 `LPUSH` 한다

```sh
dispatcher> LPUSH job_queue job2 job3
```

그럼 대기중인 `worker-2` 에서 `job2` 를 받아 처리하고 대기가 풀린다.

```sh
worker-2> BRPOP job_queue 0 
1) "job_queue" 
2) "job2" 

worker-2>
```

`job_queue` `List` 에 남아있는 원소를 다음처럼 확인하면 `job3` 만 남아있다.

```sh
dispatcher> LRANGE job_queue 0 -1 
1) "job3"
```

---

### QUICKLIST 자료구조

`Redis` 는 `quicklist` `encoding` 을 사용하여 `list` 객체를 저장한다.
`quicklist` 는 `ziplist` 와 `linkedlist` 의 혼합형 자료구조이다.

이는 여러개의 `ziplist` 를 `이중 연결 리스트` (`doubly linked list`) 로 연결한 형태이며,  각 노드는 `ziplist` 를 포함한다.

작은 리스트는 단일 `ziplist` 로 저장되며, 리스트가 커지면 여러 `ziplist` 로 분할되어 `quicklist` 구조로 저장되는 방식이다.

이러한 `quicklist` 는 `list` 객체 메모리 저장 공간을 조정하는 `2` 설정 옵션이 있다.

- `list-max-ziplist-size`: `list` 의 내부 표현을 제어하는 설정이다.<br>이 옵션은 리스트가 언제 압축 리스트 인코딩을 사용할지 결정한다.<br><br>이 옵션은 `Redis` 의 메모리 사용을 최적화하는데 중요한 역할을 한다<br><br>**양수값**:  `list` 엔트리의 최대 개수 지정<br>**음수값**: `byte`  단위로 최대 크기를 지정<br><br>**-1**: `4KB`<br>**-2**: `8KB`<br>**-3**: `16KB`<br>**-4**: `32KB`<br>**-5**: `64KB`<br>

>[!info] 리스트가 `5` 개 이하의 엔트리를 가질때 `ziplist` `encoding` 을 사용한다
```sh
list-max-ziplist-size 5
```

>[!info] 리스트가 `4KB` 이하일때, `ziplist` `encoding` 을 사용한다
```sh
list-max-ziplist-size -1
```


- `list-compress-depth`: `list` 압축 정책이다.<br>리스트의 양 끝에서 부터 지정된 개수의 노드를 제외하고 나머지를 압축한다.<br>앞축된 부분은 특별한 방식으로 `encoding` 되어 메모리사용을 줄인다.

---

