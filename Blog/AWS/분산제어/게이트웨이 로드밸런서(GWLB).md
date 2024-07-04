
---

`GWLB` 도 `Load Balancing` 한다
그러나 형태가 다른 `3개` 유형과 확연히 구분된다.

`Client` 가 `3개` `ALB`, `NLB`, `CLB` 로 요청하면 분산 대상 중 하나가 `Client` 의 요청을 처리하고 응답한다.

이는 `Target Instance` 가 `Client` 의 목적지가 된다.

**`GWLB` 의 `Target` 은 `Client` 의 최종 목적지가 아니다. `Client` 의 `Request Traffic` 은 `GWLB` 가 `Load balancing` 하는 `Target` 에 잠깐 들린후, 최종 목적지로 다시 이동한다.**

> [!info] 이러한 행동의 목적은 `Client` 의 요청하는 `Traffic` 의 접근 제어를 위해서다.<br>가상의 방화벽을 둬 악성 트래픽을 차단하고, 허용 클라이언트만 통과시키거나, `IPS` 또는 `Packet 검사` 를 하는 `가상 어플라이언스` 를 `Target` 으로 등록하고 `Load Balancing` 한다.<br><br>**필터링 정책에 따라 차단된 트래픽은 소실될 것이고, 통과된 트래픽은 `Client` 의 최종 목적지로 이동한다**

다음은 `GWLB` 의 트래픽 처리 과정이다.


![[Blog/AWS/assets/Images/GWLB 로드밸런싱 프로세스.png]]

>[!info] `GWLB` 는 `Endpoint` 를 기반으로 하는 `ELB` 이다. 이 내용은 추후에 [[Endpoint]] 에서 설명한다.

과정을 정리하면 다음과 같다.

1. `Internet Client` 가 `92.75.20.100` (`EIP`: `13.124.5.233`) 으로 `FTP` 접속을 시도한다.

2. `IGW` 에 유입된 트래픽은 `Edge Routing Table` 의 안내를 받아 `Endpoint`(`vpce-*`) 로 이동한다.

3. `Endpoint` 에 유입된 `Traffic` 은 `AWS Endpoint Server` 인 `AWS PrivateLink` 를 통과해 `Appliance VPC` 의 `GWLB` 로 엑세스한다. 

4. 로드밸런싱 알고리즘에 따라 2개 `Appliance` 중 `13.246.50.101` 로 트래픽을 보낸다<br><br>`GWLB`는 `Client` 가 보낸 트래픽을 `GENEVE` 헤더로 `캡슐화`(`Encaptulation`) 해서 `13.246.50.101` 로 전달한다.<br><br>전송시 `UDP 6081` 포트를 사용한다.

5. 가상 어플라이언스에 도착한 `GENEVE` 패킷은 `역캡슐화` (`Decaptulation`) 를 거쳐 원본 패킷으로 변환한다.<br><br> 어플라이언스가 방화벽이라면 정책에 적용된 규칙 검사를 수행한다.<br><br>통과 패킷은 다시 `GENEVE` 헤더로 `캡슐화` 되어 `GWLB` 로 전달된다.

6. `GWLB` 는 전달된 트래픽의 `GENEVE` 헤더를 다시 `역캡슐화` 하고, `GWLB` 는 처음 자신을 호출한 `Endpoint` 인 `vpce-*` 을 찾아 원본 트래픽을 회신한다.

7. `Endpoint` 에 도착한 원본 트래픽을 최종 목적지인 `92.75.20.100` 으로 보낸다.

8. 여태까지의 반대의 과정으로 `Internet` 에 접속하여 `Client` 로 응답한다

`GWLB` 를 사용하려면 다음의 조건을 만족해야 한다.

- `Load Balancing` 대상은 `GENEVE` 프로토콜(`UDP 6081`) 을 지원하는 가상 어플라이언스여야 한다.

- `GWLB` 엔드포인트 서비스(`AWS PrivateLink`) 와 `EndPoint` 를 생성해야 한다.<br> 클라이언트 트래픽의 첫 진입점은 이 엔드포인트(`vpce-*`) 이다.





