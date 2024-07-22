
>[!info] `개발자를 위한 레디스` 의 내용과 `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

`RESP` (`Redis Serialization Protocol`) 는 `Redis` 클라이언트가 `Redis` 서버와 통신할때 사용하는 프로토콜이다.

 `client` 와 `server` 간 사이의 소통은 일반적으로 다음처럼 진행된다
 
1. `client` 는 `command` 를 `server` 로 보낸다
2. 서버는 `command` 를 받고 `command` 를 `execution queue` 로 넣는다.<br>(`Redis` 는 `single-threaded execution model` 이다)
3. `command` 를 실행한다.
4. `server` 는 실행 결과를 리턴하고 `client` 로 보낸다.

이러한 전체 프로세스의 시간을 `round-trip time`(`RTT`) 로 명명한다.

현재 $2$ 번째와 $3$ 번째 프로세스를 본다면, 이는 `Redis Server`  에서 작동되는것을 알수 있고,
$1$ 번째와 $4$ 번째 프로세스는 `server` 와 `client` 사이의 전체 `네트워크 대기시간`(`network latency`) 과 연관되어 있음을 알수 있다.

만약, 여러 명령들을 실행한다고 하면, 
`server` 에서 실행되는 시간은 매우 짧은 것에 비해 `network`  전송시간은 길수밖에 없다

이러한 과정을 `Redis` 의 `pipeline`  을 통해 더 나은 방식으로 처리할수 있다.

`pipeline` 의 기본 아이디어는 각기 다른 `command` 실행하기 위해 대기하지 않고,  여러 `command` 를 하나로 묶어(`bundle`) 한번에 여러 `command` 를 보내는 것이다.

이는 `server` 에게 모든 `command` 를 실행한후 그 결과를 반환하도록 요청한다.

$1$ 번째 와 $4$ 번째에서 여러 `command` 를 보내는 방식에서 오직 한번만  보내므로 전체 실행 시간을 극단적으로 줄어든다.

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





