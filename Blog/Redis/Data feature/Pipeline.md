
>[!info] `개발자를 위한 레디스` 의 내용과 `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`RESP` (`Redis Serialization Protocol`) 는 `Redis` 클라이언트가 `Redis` 서버와 통신할때 사용하는 프로토콜이다.

 `client` 와 `server` 간 사이의 소통은 일반적으로 다음처럼 진행된다
 
1. `client` 는 `command` 를 `server` 로 보낸다
2. 서버는 `command` 를 받고 `command` 를 `execution queue` 로 넣는다.<br>(`Redis` 는 `single-threaded execution model` 이다)
3. `command` 를 실행한다.
4. `server` 는 실행 결과를 리턴하고 `client` 로 보낸다.

이러한 전체 프로세스(`요청과 응답사이의 왕복시간`)  에 걸리는 시간을 `round-trip time`(`RTT`) 로 명명한다.

현재 $2$ 번째와 $3$ 번째 프로세스를 본다면, 이는 `Redis Server`  에서 작동되는것을 알수 있고,
$1$ 번째와 $4$ 번째 프로세스는 `server` 와 `client` 사이의 전체 `네트워크 대기시간`(`network latency`) 과 연관되어 있음을 알수 있다.

만약, 여러 명령들을 실행한다고 하면, 
`server` 에서 실행되는 시간은 매우 짧은 것에 비해 `network`  전송시간은 길수밖에 없다

이러한 과정을 `Redis` 의 `pipeline`  을 통해 더 나은 방식으로 처리할수 있다.

`pipeline` 의 기본 아이디어는 각기 다른 `command` 실행하기 위해 대기하지 않고,  여러 `command` 를 하나로 묶어(`bundle`) 한번에 여러 `command` 를 보내는 것이다.

이는 `server` 에게 모든 `command` 를 실행한후 그 결과를 반환하도록 요청한다.

$1$ 번째 와 $4$ 번째에서 여러 `command` 를 보내는 방식에서 오직 한번만  보내므로 전체 실행 시간을 극단적으로 줄어든다.

>[!note] 요청과 응답 사이의 왕복 시간(`RTT`) 는 성능에 큰 영향을 미친다.<br><br>왕복시간이 `250m` 인 경우 `redis`서버가 초당 $10$ 만개 처리 가능하다고 가정할때, 네트워크 통신 소요시간으로 인해 초당 $4$ 만개 요청만 처리 가능하게 될수도 있다.<br><br>이는 **네트워크 통신 소요시간을 줄이면 성능을 크게 향상시킬수 있음을 말한다.**

---

`pipeline` 을 사용하기 위해 모든 명령어를 담은 `pipeline.txt` 를 생성한다.

>[!info] pipeline.txt
```sh
~$ cat << EOF > pipeline.txt 
heredoc> set mykey myvalue                  
heredoc> sadd myset value1 value2           
heredoc> get mykey
heredoc> scard myset                        
heredoc> EOF

~$ cat pipeline.txt
set mykey myvalue
sadd myset value1 value2
get mykey
scard myset
```

그리고 `redis-cli --pipe` 를 통해 모든 명령어를 받아 실행한다

>[!info] --pipe
```sh
~$ cat pipeline.txt | bin/redis-cli --pipe 
All data transferred. Waiting for the last reply... 
Last reply received from server. 
errors: 0, replies: 4
```

이 예시를 보면 매우 간단하게 `pipeline`  을 보낼수 있음을 볼수 있다.
이는 각 `command` 사이에 줄바꿈을 이용해서 실행할 여러개의 `command`  를 한번에 보내는 방식이다.

>[!info] `pipeline.txt` 의 `raw` 버전 
```sh
$ cat pipeline.txt
*3\r\n$3\r\nSET\r\n$5\r\nmykey\r\n$7\r\nmyvalue\r\n*4\r\n$4\r\nSADD\r\n$5\r\nmyset\r\n$6\r\nvalue1\r\n$6\r\nvalue2\r\n*2\r\n$3\r\nGET\r\n$5\r\nmykey\r\n*2\r\n$5\r\nSCARD\r\n$5\r\nmyset\r\n

$ echo -e "$(cat datapipe.txt)" | bin/redis-cli --pipe 
All data transferred. Waiting for the last reply... 
Last reply received from server. 
errors: 0, replies: 4
```

다음은 `NODEJS`  의 `redisio` 의 `pipeline` 에시이다.

```js
const pipeline = redis.pipeline();
pipeline.set("foo", "bar");
pipeline.del("cc");
pipeline.exec((err, results) => {
  // `err` is always null, and `results` is an array of responses
  // corresponding to the sequence of queued commands.
  // Each response follows the format `[err, result]`.
});

// You can even chain the commands:
redis
  .pipeline()
  .set("foo", "bar")
  .del("cc")
  .exec((err, results) => {});

// `exec` also returns a Promise:
const promise = redis.pipeline().set("foo", "bar").get("foo").exec();
promise.then((result) => {
  // result === [[null, 'OK'], [null, 'bar']]
});
```

이를 통해 `promise` 로 `pipeline` 에 메서드체인된 명령의 결과를 받아 처리됨을 볼 수 있다.
기본적으로 `pipeline` 을 사용한 `command` 와 그렇지 않은 `commend` 는 차이가 크다.

>[!info] `command` 가 크면 클수록, 그 차이가 확연하게 차이날수 있다.<br><br>`개발자를 위한 레디스`  에서는 $100$ 만개의 키를 가져오는데 일반 `SCAN`  으로 가져온후, 타입과 메모리 사용량을 각각 `command` 로 보냈는데 $4$ 시간이 걸렸다.<br><br> 반면, `pipeline` 으로 타입과 메모리 사용량의 `command` 를 한꺼번에 $1200$ 개를 보내는데 $3$ 분으로 줄었다.. 

>[!warning] 주의할점은 한번에 너무 많은 쿼리를 `pipeline` 으로 보내면 네트워크 대역폭 한계로 속도가 저하될수 있다.<br>이는 레디스 클라이언트 쿼리 버퍼 제한에 걸릴수 있기도 하다.<br><br>**이부분을 처리하기 위해 `batch` 형태로 서버로 보내는 것을 추천한다.**

>[!warning] 주의할 점은, `pipeline` 의 `command` 단위로 원자성을 보장한다는 것이다.<br><br>이는 다른 클라이언트가 같은 `pipeline` 을 보낸다고 가정할때, 서버에 먼저 접근한 `command` 를 실행하고 있더라도, 이후 접근한 클라이언트의 연결을 차단하지 않으며, 각각 `command` 는 교차로 수행된다.






