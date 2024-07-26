
>[!info] `개발자를 위한 레디스` 의 내용과 `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`transaction`  은 `RDB`  에서 그룹지어진 수행된 작업을 원자적으로 실행한다.
이는 그룹돤위로 실행을 완료하고 전체적으로 성공 혹은 실패하는 방식을 의미한다.

---

다음은 초당판매를 하는 상황이다.
이를 구현은 다음처럼 이루어진다.

```python
//Initiate the count of coupon codes: 
SET("counts:seckilling",5);   
 
//Start decreasing the counter: 
WATCH("counts:seckilling");   
count = GET("counts:seckilling"); 
MULTI();  
if count > 0 then 
           DECR("counts:seckilling",1); 
           EXEC();   
           if ret != null then 
                   print "Succeed!" 
              else 
                   print "Failed!" 
else 
  DISCARD();   
print "Seckilling Over!" 
```

1. `WATCH` `command` 는 `EXEC` `command` 이전에 `key` 가 변경되는지를 나타내는 `flag`를 설정한다.  그러면, 모든 `transaction `을 폐기하고, 카운터의 값을 다시 얻는다.

2. `MULTI` 명령어로 트랙잭션을 시작한다. 만약 카운터 값이 유효하지 않다면 `DISCARD` 명령어로 트랜잭션을 즉시 중단한다. 유효하다면 카운터를 감소시키는 작업을 진행한다.

3. 그 다음 트랜잭션 실행을 시도한다. 이전에 사용한 `WATCH` 명령어로 인해 `Redis `는 `counts:seckilling` 값이 변경되었는지 확인한다. 값이 변경되었다면 트랜잭션은 중단되며, 이는 초당 판매 실패로 간주된다.

이러한 초당 판매에 위한 과잉판매를 해결하기 위해 처리를 하는 내용이며, 카운터 값을 조회할때 발생하는 `race condition` 을 처리해준다.

---

`RDBMS` 와 `Redis` 간의 `Transaction` 의 다른점을 주목할 가치가 있다.
중요한 차이점은, `Redis` 의 `Transaction `은 `rollback` 기능이 없다는 것이다.

일반적으로 `Redis` 의 `transaction` 은 다양한 처리 매커니즘에 의해 2가지 타입의 에러가 발생할수 있다.

1. 첫번째는 `syntax Error` 이다.<br>이 경우 `command` 가 `queuing` 되는 동안 발견된 에러이기 때문에, 전체 `transaction`은 빠르게 실패되며, 이 `transaction` 의 `commands` 도 처리되지 않는다.

>[!info] Syntax Error
```sh
127.0.0.1:6379> MULTI 
OK 
127.0.0.1:6379> SET FOO BAR 
QUEUED 
127.0.0.1:6379> GOT FOO 
(error) ERR unknown command 'GOT' 
127.0.0.1:6379> INCR MAS 
QUEUED 
127.0.0.1:6379> EXEC 
(error) EXECABORT Transaction discarded because of previous errors. 
```

2. 두번째는 모든 `commands` 는 `queued` 되었지만, 실행 중간에 발생한 에러이다.<br>이후 명령은 `rollback` 없이 계속 실행된다.

>[!info] Error occurs in the middle of execute
```sh
127.0.0.1:6379> MULTI 
OK 
127.0.0.1:6379> SET foo bar 
QUEUED 
127.0.0.1:6379> INCR foo 
QUEUED 
127.0.0.1:6379> SET foo mas 
QUEUED 
127.0.0.1:6379> GET foo 
QUEUED 
127.0.0.1:6379> EXEC 
1) OK 
2) (error) ERR value is not an integer or out of range 
3) OK 
4) "mas"
```



