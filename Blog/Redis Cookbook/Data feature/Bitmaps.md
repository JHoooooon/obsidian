
>[!info] `개발자를 위한 레디스` 의 내용과 `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`Bitmap` 은 `bits` 로 이루어진 배열이다.
`Redis`  `bitmap` 은 새로운 데이터 타입이 아니다.

이는 실제 데이터 타입의 유형은  `string` 타입이다. 

`Bitmap` 은 특정 상황에 대한 정보를 `boolean` 값으로 메모리 공간에 저장한다.

이를 이용하여  `User` 가 어플리케이션에서 제공하는 기능을 제공하고 있는지 `Bitmap` 을 사용하여 `flag` 로 저장할수 있다.

>[!info] `Boolean` 은 `True` 아니면 `False` 값만 가지므로, 프로그래밍 에서는 `0` 과 `1` 로 사용된다.<br>문자열보다 `bit` 인 `0` 이나 `1` 로 표현하는게, 메모리상으로는 당연 이득이다.<br><br>단, 레디스는 숫자값 역시 문자열로 치환하여 처리한다.<br> 그러므로 일반적인 프로그래밍 언어에서 숫자값으로 처리한다고 생각하면 안된다.

`BITMAP` 구조는 다음처럼 생각해 볼 수 있다. 

![[bitmap structuer.png]]

위는 여러 `user_id` 를 가진 `Redis` 집합을 사용하는것을 볼 수 있다.
`bitmap` 안의 모든 유저는 `bit` 를 사용했는지 안했는지 상관없이 하나의 `bit` 를 필요로 한다.

`user_id` 가 $20억$ 의 유저를 지원한다고 가정하자.
그럼 `user_id` 는 $00억$ 유저의 `bit` 가 필요하게 되며, 대략적으로 `250MB`의 메모리를 사용할 것이다.

$2,000,000,000 \div 8bit = 250,000,000 byte$

만약 `SET` 자료구조 안에 비슷한 카운팅 기능을 구현하는 데이터 구조를 만들었다고 하자.
그럼, 이는 `user_id` 를 저장하므로, `id` 는 `8byte` 로 이루어질 것이다.

그리고, `1.6억` 명의 유저가 매우 자주 접속하는 유저를 찾는다고 가정해보자.
그럼, 이를 다음처럼 계산할수 있다.

$160,000,000 \times 64 byte = 1,280,000,000 byte$

`1.28 GB` 로 계산되는것을 알수 있다. 

오히려 $20억$ 보다도 더 적은 $1.6$ 억명의 유저를 저장하는데, 대략 $1000$ 배 정도의 메모리가 더 많이 사용되는것을 볼 수 있다.

하지만 다음과 같은 상황에서는 `BIT` 가 유용하지 않을수 있다.
만약, $20억$ 명의 유저의 $1\%$ 의  유저만 기능을 사용한다고 가정하자.

$2,000,000,000 \times 0.01 = 2,000,000$

이는 총 $200만$ 명의 유저만 사용한다.

`SET` 자료구조를 사용하면 다음과 같다

$2,000,000 \times 64byte = 128,000,000 byte$

이는 존재하는 $20억$ 명중 $1\%$ 의 유저만 `SET` 자료구조에 저장되므로 총 $128 MB$ 만 필요하다.
하지만, `BIT` 는, 여전히 $20억$ 명의 모든 `user_Id` 를 가지고 있으므로, $250MB$ 를 사용한다.

>[!info] 앞에서 이미 말했지만, `BIT` 는 모든 `user_id` 를 `bit` 형식의 배열로 저장한다.<br>`bit` 의 값이 있든 없든, `flag` 로써 `0` 아니면, `1` 로 저장되므로, 모든 `user_id` 가 존재해야 처리 가능하다<br><br> 반면 `SET` 자료구조는 `user_id` 를 삽입하므로, 모든 `user_id` 의 존재가 필요하지 않다. 

>[!warning] `bitmap` 안에 희소 `bit` 가 세팅되어 때때로 `Redis server` 가  `block` 될수 있다.<br>이는 `bit` 가 매우큰 `offset` 을 가지던가, 존재하는 `bitmap` 의 크기가 매우 작을때 발생한다<br><br>`Redis` 는 즉각적으로 메모리에 할당하므로 `bitmap` 의 크기를 확장되기 때문에 발생할수 있는 문제라고 말한다.

---

`BIT` 값을 저장하기 위해 다음처럼 `SETBIT` 를 사용할수 있다.

>[!note] 다음은 `ID` 가 $100$ 인 유저에 `BIT` `flag` 를 저장한다.

>[!info] SETBIT
```sh
127.0.0.1:6379> SETBIT "users_tried_reservation" 100 1
(integer) 0
```

위의 상황은, `user` 가 예약을 시도했는지를 표시하는 `bit` 이다.

다음은 `GETBIT` 명령어를 사용하여, `bitmap` 으로 부터 지정한 값으로 `bit` 값을 찾는다.
하지만 `users_tried_online_orders` 에 $400$ 값을 저장하지 않았으므로, `0` 값을 반환한다.

>[!info] GETBIT
```sh
127.0.0.1:6379> GETBIT "users_tried_online_orders" 400
(integer) 0
```

다음은, `BITCOUNT` 명령어를 사용하여 `bit` 값이 몇개있는지 확인한다.

>[!info] BITCOUNT
```sh
127.0.0.1:6379> BITCOUNT "users_tried_reservation"
(integer) 1
```

`bitwise` 연산을  처리하기 위한 명령어는 `BITOP` 이다.
`bitwise` 는 다음의 연산을 제공한다.

- AND
- OR
- XOR
- NOT

이로인해 나온 결과는 목적지 `key` 에 저장된다.
다음의 예는, `tried_both_reservation_and_online_orders` 에 `tried_reservation` 과 `tried_online_orders`  에 같이 속하는 `id` 들을 저장하는 명령이다. 

>[!info] BITOP
```sh
127.0.0.1:6379> BITOP AND "users_tried_both_reservation_and_online_orders" "users_tried_reservation" "users_tried_online_orders" 
(integer) 13

127.0.0.1:6379> BITCOUNT "users_tried_both_reservation_and_online_orders" 
(integer) 0
```

`BITCOUNT` 를 연산해보면, 둘다에 속하는 값이 없음을 알수 있다.

