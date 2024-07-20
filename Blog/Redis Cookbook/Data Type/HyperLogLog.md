>[!info] `개발자를 위한 레디스` 의 내용과 `PacktPub Redis Cookbook` 의 내용을 정리한 것이다.

>[!note] `hyperloglog` 는 집합의 원소 개수인 카디널리티를 추정할수 있는 자료구조이다.<br>대량 데이터에서 중복되지 않는 고유한 값을 집계할때 유용하게 사용할 수 있는 데이터 구조이다

`SET` 과 같은 데이터 구조에서는 중복을 피하기 위해 저장된 데이터를 모두 기억하고 있다.
이는 저장되는 데이터가 많아질 수록 그 만큼 많은 메모리를 사용해야만 한다.

현재 필요한건, `데이터 자체를 모두 기억하는것이 아닌 저장된 데이터의 원소개수만 필요`하다는 것이다.
`HyperLogLog` 는 `데이터 자체가 아닌 자체적으로 변형한 데이터를 저장` 하는 방식을 가진다.  

>[!note] 이러한 이유로 `HyperLogLog` 는 데이터 개수에 구애받지 않는다. <br><br>**최대 $12KB$ ($2^{64}$ 저장가능)크기를 가지며, 시간복잡도는 $O(1)$ 이다**.<br>하지만 `HLL` 알고리즘은 **부정확하며, 일반적으로 $1\%$ 미만의 표준오류가 존재**할수 있다.<br><br>그럼에도, 매우 유용한 알고리즘으로 많이 사용된다.


이는, `중복되지 않으며, 유일한 원소의 개수` 를 파악하는데 좋은 방식이다.

---

>[!info] `PF` 라는 접두어는 `HLL` 데이터 구조를 발명한 사람인 `Philippe Flajolet` 의 앞글자를 딴 접두어이다.

다음은 `PFADD` 를 사용하여, `userId` 을 `HyperLogLog` 에 추가한다.

```sh
127.0.0.1:6379> PFADD "Counting:Olive Garden" "0000123" 
(integer) 1 
 
127.0.0.1:6379> PFADD "Counting:Olive Garden" "0023992" 
(integer) 1 
```

`FACOUNT` 를 사용하여 `userId` 가 몇개 저장되었는지 출력한다.

```sh
127.0.0.1:6379> PFCOUNT "Counting:Olive Garden" 
(integer) 2
```

만약, 주간 인기를 표시하기 위해 일주인동안 `Olive Garden` 의 고유한 방문자수를 표시한다고 가정하자.
이때, 각 날짜마다 7일단위로 묶어야 일주일이 되므로, 각 날짜의 방문객들을 `Merge` 해야 한다.

이때 사용하는 것이 `PFMERGE` 이다.

```sh
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170903" "0023992" "0023991" "0045992" 
(integer) 1 
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170904" "0023992" "0023991" "0045992" 
(integer) 1 
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170905" "0024492" "0023211" "0045292" 
(integer) 1 
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170906" "0023999" "0063991" "0045922" 
(integer) 1 
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170907" "0023292" "0023991" "0045921" 
(integer) 1 
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170908" "0043282" "0023984" "0045092" 
(integer) 1 
127.0.0.1:6379> PFADD "Counting:Olive Garden:20170909" "0023992" "0023991" "0045992" 
(integer) 1 
 
127.0.0.1:6379> PFMERGE "Counting:Olive Garden:20170903week" "Counting:Olive Garden:20170903" "Counting:Olive Garden:20170904" "Counting:Olive Garden:20170905" "Counting:Olive Garden:20170906" "Counting:Olive Garden:20170907" "Counting:Olive Garden:20170908" "Counting:Olive Garden:20170909" 
OK 
 
127.0.0.1:6379> PFCOUNT "Counting:Olive Garden:20170903week" 
(integer) 14
```

위는 총 7일의 `HyperLogLog` 의 날짜를 `PFMERGE` 로 병합하고 `PFCOUNT` 를 센 결과이다.

