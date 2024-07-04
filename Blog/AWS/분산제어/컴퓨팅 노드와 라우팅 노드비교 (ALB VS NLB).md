
---

>[!info] `ENI` 는 `2가지 유형` 이 존재한다.<br>`Computing ENI` 와 `Routing ENI` 이다.<br><br>`Computing ENI` 는 `SG` 를 가지며, `Instance` 접근을 제어한다.<br><br>반면 `Routing ENI` 는 `SG` 를 가지지 않으며, 단순 `Traffic` 을 다른 `ENI` 로 `Routing` 하는 역할을 갖는다.<br><br>[[인터넷 게이트웨이(IGW) 와 NAT Gateway]] 에서 `NAT Gateway` 가 `Routing ENI` 로써의 역할을 한다고 볼수 있다.

내용을 정리하면 다음과 같다. 

1. `Source / Dest check`: `Routing ENI` 는 `Source/dest check` 설정이 해제된 상태로 생성되며, `Traffic` 을 수신하면 데이터 변경 없이 `Fowarding` 한다

2. `SG 강제 적용`: `Computing ENI` 는 `SG` 를 사용하고, `Routing ENI` 는 사용하지 않는다<br>`ELB Node` 도 `ENI` 를 기반하므로, `2` 가지 유형중 하나일 것이다<br><br>`ALB` 와 `CLB` 는 `SG` 가 연결되어 있지만, `NLB, GWLB` 는 `SG` 가 연결이 불가하다.<br><br>이는 `ALB` 와 `CLB` 가 `Computing ENI` 인것으로 유추 가능하며, `NLB`, `GWLB` 는 `Routing ENI` 로 유추될수 있다<br><br>[[AWS 의 ELB]] 의 `ELB` 유형을 보면 `SG` 를 사용하는지 알수 있다.

이를 보고, `Computing ENI` 를 사용하는 `ALB` 와 `Routing ENI` 를 사용하는 `NLB` 로 구분하여 차이점을 보도록한다.

다음은 `ALB` 의 `SG` 사용 프로세스 이다.

![[ALB 의 SG 프로세스.png]]

`ELB` 에 연결된 `SG` 는 다음 접근만 허용해야 한다

- `Inbound`: `Client IP` 와 `ELB Listener` 의 `Protocol` 
- `Outbound`: `ELB` 가 로드밸런싱할 대상 `IP` 와 `Protocol`

>[!info] `VPC` 내부의 `SG` 로 통신하니, 직접 `Private IP` 를 사용하기 보다는 `SG-2*` 이나 `SG-1*` 으로 `Inbound` 및 `Oubound` 를 지정해주는것이 좋다.

다음은 `NLB` 의 사용 프로세스다

![[NLB 의 SG 사용 프로세스.png]]

`NLB` 는 `SG` 를 직접 연결해서 사용하지 않는다
이는 `Target Group` 의 `Target Instance` 에서 접근을 제어해야 한다

>[!warning] `SG` 를 사용하지 않는 `NLB` 같은 유형은, `Target Instance` 에서 직접 접근을 제어해야 하므로 특히 더 유의해서 설정해주어야 한다.

`source/dest check` 를 하지 않는 `NLB` 는 `유입 트래픽` 의 `데이터 조작 없이` 즉시 로드밸런싱하지만, `ALB` 는 별도 `데이터 처리 과정을 거친뒤` 로드밸런싱한다.

`ALB`, `NLB` 에서는 `라우팅 방법` 이 존재하는데, `ELB` 생성 이후 `편집` 기능으로 변경 가능하다.
`전달 대상` 옵션만 선택가능한 `NLB` 와는 달리 `ALB` 는 다음 4가지 옵션이 있다

- `전달대상`: 일반적은 로드밸런싱이다.

- `리다이렉션`: `Client` 에게 다른 `Routing` 을 제시한다<br>`http://92.75.200.3` 요쳥을 `https://92.75.200.33` 으로 변경 처리한다`

- `고정 응답 반환`: `Client` 요청 데이터와 관계없이 단 하나의 응답만 제공한다.<br>