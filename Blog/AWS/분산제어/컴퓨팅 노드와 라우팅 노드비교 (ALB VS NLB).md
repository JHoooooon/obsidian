
---

>[!info] `ENI` 는 `2가지 유형` 이 존재한다.<br>`Computing ENI` 와 `Routing ENI` 이다.<br><br>`Computing ENI` 는 `SG` 를 가지며, `Instance` 접근을 제어한다.<br><br>반면 `Routing ENI` 는 `SG` 를 가지지 않으며, 단순 `Traffic` 을 다른 `ENI` 로 `Routing` 하는 역할을 갖는다.<br><br>[[인터넷 게이트웨이(IGW) 와 NAT Gateway]] 에서 `NAT Gateway` 가 `Routing ENI` 로써의 역할을 한다고 볼수 있다.

내용을 정리하면 다음과 같다. 

1. `Source / Dest check`: `Routing ENI` 는 `Source/dest check` 설정이 해제된 상태로 생성되며, `Traffic` 을 수신하면 데이터 변경 없이 `Fowarding` 한다

2. `SG 강제 적용`: `Computing ENI` 는 `SG` 를 사용하고, `Routing ENI` 는 사용하지 않는다<br>`ELB Node` 도 `ENI` 를 기반하므로, `2` 가지 유형중 하나일 것이다<br><br>`ALB` 와 `CLB` 는 `SG` 가 연결되어 있지만, `NLB, GWLB` 는 `SG` 가 연결이 불가하다.<br><br>이는 `ALB` 와 `CLB` 가 `Computing ENI` 인것으로 유추 가능하며, `NLB`, `GWLB` 는 `Routing ENI` 로 유추될수 있다<br><br>[[AWS 의 ELB]] 의 `ELB` 유형을 보면 `SG` 를 사용하는지 알수 있다.

이를 보고, `Computing ENI` 를 사용하는 `ALB` 와 `Routing ENI` 를 사용하는 `NLB` 로 구분하여 차이점을 보도록한다.

다음은 `ALB` 의 `SG` 사용 프로세스 이다.

![[ALB 의 SG 프로세스.png]]


