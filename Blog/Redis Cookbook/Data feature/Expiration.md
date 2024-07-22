
`Redis` 의 `key` 를 삭제할때 `DEL` 또는 `UNLINK` 명령어를 사용한다.

`key` 를 수동으로 삭제하는것 외에도, `keys` 의 `timeout` 을 설정해서 자동적으로 `key` 를 삭제하도록 `Redis` 에 알려줄수 있다.

이를 사용하는것이 `Expiration` 이다.

`key` 의 만료시간은 `UNIX timestamp` 값으로 저장된다

간혹, `Redis 서버` 가 `down` 될지라도, `expiration timestamp` 를 잃지 않으며, `key` 는 여전히 `UNIX timestamp` 시간이 지나면, `server` 가 다시 `up` 될때 만료된다. 

`expired` `key` 이며 , `cliet` 가  이 `key` 에 접근을 시도하면, `Redis` 는 `memory` 에서 즉작적으로 해당하는 `key` 를 삭제한다.

이러한 방식으로 `Redis` 에서 `key` 를 삭제하는 방식을 `expiring passively` 라고 한다.
만약 `expired` `key` 이고 절대 접근하지 않는다면, `Redis` 는 확률적 알고리즘을 사용하여

---

다음은 , `Redis` 배열에 총 $5$ 명의 `user` 를 삽입한다.

이때, `expired` 를 지정하여 `timeout` 되도록 처리하면, 해당 시간 만큼 `user` 가 유지되고 이후에는 사라진다.

다음은 `closest_restaurant_ids` 의 유저가 $300$ 초까지 유지되고 사라지도록 명령한다.

>[!info] EXPIRE
```sh
127.0.0.1:6379> LPUSH "closest_restaurant_ids"
109 200 233 543 222
(integer) 5

127.0.0.1:6379> EXPIRE "closest_restaurant_ids" 300 
(integer) 1
```

$300$ 초 이후에 `EXIST` 를 사용하여 해당 `LIST` 가 있는지 확인해본다. 

```sh
127.0.0.1:6379> EXISTS "closest_restaurant_ids" 
(integer) 0
```

$0$ 을 반환하여, `LIST` 가 삭제되었음을 알수 있다.

>[!info] `Redis`  는 기본적으로, `empty` 한 `key` 값이 있다면, 삭제된다.<br>`garbage colloector` 에 의해 삭제되는듯하다.


