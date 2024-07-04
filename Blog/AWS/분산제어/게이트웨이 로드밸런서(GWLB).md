
---

`GWLB` 도 `Load Balancing` 한다
그러나 형태가 다른 `3개` 유형과 확연히 구분된다.

`Client` 가 `3개` `ALB`, `NLB`, `CLB` 로 요청하면 분산 대상 중 하나가 `Client` 의 요청을 처리하고 응답한다.

이는 `Target Instance` 가 `Client` 의 목적지가 된다.

**`GWLB` 의 `Target` 은 `Client` 의 최종 목적지가 아니다. `Client` 의 `Request Traffic` 은 `GWLB` 가 `Load balancing` 하는 `Target` 에 잠깐 들린후, 최종 목적지로 다시 이동한다.**

> [!info] 이러한 행동의 목적은 `Client` 의 요청하는 `Traffic` 의 접근 제어를 위해서다.<br>가상의 방화벽을 둬 악성 트래픽을 차단하고, 허용 클라이언트만 통과시키거나, `IPS` 또는 `Packet 검사` 를 하는 `가상 어플라이언스` 를 `Target` 으로 등록하고 `Load Balancing` 한다.<br><br>**필터링 정책에 따라 차단된 트래픽은 소실될 것이고, 통과된 트래픽은 `Client` 의 최종 목적지로 이동한다**

다음은 `GWLB` 의 트래픽 처리 과정이다.
`GWLB` 는 `Endpoint` 를 기반으로 하는 `ELB` 이



