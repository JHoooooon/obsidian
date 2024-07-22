
>[!info] `개발자를 위한 레디스` 의 내용과 `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`Redis` 의 `key` 를 삭제할때 `DEL` 또는 `UNLINK` 명령어를 사용한다.

`key` 를 수동으로 삭제하는것 외에도, `keys` 의 `timeout` 을 설정해서 자동적으로 `key` 를 삭제하도록 `Redis` 에 알려줄수 있다.

이를 사용하는것이 `Expiration` 이다.

`key` 의 만료시간은 `UNIX timestamp` 값으로 저장된다

간혹, `Redis 서버` 가 `down` 될지라도, `expiration timestamp` 를 잃지 않으며, `key` 는 여전히 `UNIX timestamp` 시간이 지나면, `server` 가 다시 `up` 될때 만료된다. 

레디스는 만료된 `key` 를 삭제되는 $2$ 가지 방식이 존재한다.

>[!info] 만료되는것과 레디스에서 삭제하는것과는 다른의미이다.<br><br>레디스는 삭제되는데 들어가는 리소스를 줄이기 위해 만료와 동시에 메모리에서 삭제하지 않고, `active` 방식과 `passive` 방식 두가지를 사용한다.

- **passive 방식**: 클라이언트가 키에 접근하고자 할때 키가 만료되었다면 메모리에서 수동적으로 삭제한다.<br>클라이언트 접근시에 삭제된다고 해서 `passive` (`수동적인`) 이라는 뜻을 사용했다고 하는듯 하다. 

- **active 방식**: `TTL` 값이 있는 키 중 `20` 개를 랜덤하게 뽑아낸뒤, 만료된 키를 모두 메모리에서 삭제한다.<br><br>만약 $25\%$  이상의 키가 만료되어 삭제되었다면, 다시 $20$ 개의 키를 랜덤하게 뽑은 뒤 선택하고 프로세스를 반복한다. 이는 기본값으로 $10$ 초마다 실행되며, 설정파일의 `hz` 값으로 설정가능하다.

다음의 방법으로 `key` 의 `timeout` 을 제거할수 있다.

- `PERSIST` 명령어는 `key` 를 지속가능한 `key` 로 만들어준다

- `key` 의 값을 `delete` 하거나 변경한다<br> 이는 `SET`, `GETSET`, `*STORE` 명령어가 포함된다<br><br>`Redis` 의 `LIST`, `SET`, `HASH` 에서부터 변경된 요소는 `timeout` 이 `clear` 되지 않는다.<br>이유는 이 연산은 `key` 의 `value` 객체를 변경하지 않기때문이다. 

>[!warning] 프로그래밍 언어의 객체를 생각하면된다.<br>객체 프로퍼티를 변경한다고 해서, 객체를 참조하는 메모리값이 변경되지 않는다.<br><br> 간단히 말해서 소포의 겉 박스의 내용물을 변경한다고 해서 소포의 겉 박스는 변경되지 않는다. 그저 내용물만 변경될 뿐이다.<br><br>`LIST`, `SET`, `HASH` 모두 객체로써 작동하므로, 내용물을 변경하다고 `timeout` 이 `clear` 되지 않는다<br><br>`EXPIRE` 는 해당 데이터타입 자체에 작동하므로, 객체의 내용물 변경은 해당하지 않는다.

- 만약 `key` 가 존재하거나 연관된 만료키가 없다면, `TTL` 명령에서 $-1$ 을 리턴한다 <br> 그러나 `key` 가 존재하지 않는다면 $-2$  를 리턴한다

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

다음은 `TTL` 을 통해 현재 몇초가 지났는지 확인가능하다.

>[!info] TTL
```sh
127.0.0.1:6379> TTL "closest_restaurant_ids" 
(integer) 253
```

$300$ 초 이후에 `EXIST` 를 사용하여 해당 `LIST` 가 있는지 확인해본다. 

```sh
127.0.0.1:6379> EXISTS "closest_restaurant_ids" 
(integer) 0
```

$0$ 을 반환하여, `LIST` 가 삭제되었음을 알수 있다.

>[!info] `Redis`  는 기본적으로, `empty` 한 `key` 값이 있다면, 삭제된다.<br>`garbage colloector` 에 의해 삭제되는듯하다.


