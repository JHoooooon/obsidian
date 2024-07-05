
>[!info] [[연결 서비스]] 에서 이어지는 챕터이다.

---

> [!note] 서로 다른 `VPC` 간에 연결을 `Peering` 이라고 하고, 그 서비스를 `Peering Connection` (`PCX`)이라 한다

이는 `VPC` 가 연결 가능한 상대측 `VPC` 는 `계정` 과 `region` 을 가리지 않는다.
`CIDR` 조건만 만족하면 모든 `VPC` 와 연결가능하다.

이는 다음과 같다

![[PCX 토폴로지.png]]

토폴로지를 보면, `Source VPN` 인 `92.75.0.0/16` 은 `Seoul` 리전에 있으며,
연결 대상인 `Target VPN` 은 `N.Verginia VPN` 에 있다 

이를 `PCX`(`Peearing Connection`) 을 사용하여, 연결한 모습이다.

> [!note] `VPC` -> `Peering Connection` 메뉴에서 `피어링 연결 생성` 버튼을 사용해서 생성한다.<br><br>이때, `VPC 요청자` 와 연결한 `VPC 리전` , `VPC ID` 를 입력하여 생성한다.<br><br>생성된 `Peering Connection` 은 `Targe VPN` 에서 `수락` 을 받아야 연결이 가능하므로, `Target VPN` 으로 넘어가 `작업` -> `요청수락` 을 클릭하여 처리해줘야 한다. 

>[!warning] `수락전` 에는 `상태` 가 `수락대기` 중 상태로 표시되므로 참고하자

이후, `Routing Table` 을 만들고, 내부 `Subnet` 에 연결한다

`Source VPN` 은 `37.120.20.0/24` 로 향하는 `Target` 은 `PCX` 로 연결하고,
`Target VPN` 은 `92.75.10.0/24` 와 `92.75.20.0/24` 로 향하는 `Target` 을 `PCX` 로 연결한다.

>[!note] `PCX` 게이트웨이를 지정한 서브넷만 상대측 `VPC` 와 통신할수 있다.

## CIDR 가 겹치는 `VPC` 간 피어링 - 불가

만약, `런던 Region`  에  신규 `VPC`  `92.75.128.0/18` `CIDR` 를 생성했다고 가정하자. 

`서울 Region` 은 `92.75.0.0/16` `CIDR` 범위를 가지고 있으며, `서울 Region` 을 `Source VPN` 으로, `런던 Region` 을 `Target VPN` 으로 설정하여  `PCX` 로 연결해보면, `Fail` 된다.

>[!fail] Failed due to incorrect VPC-ID, Account ID. or overapping CIDR range
 **`VPC-ID`, `Account ID` 가 올바르지 않거나, `CIDR 범위`  가 겹쳐서 실패했다**

`서울 Region` 과 `런던 Region` 의  `CIDR` 범위가 겹치므로, `PCX` 연결할수 없다는 것이다.


## 전이적 VPC 피어링 - 불가

>[!note] `전이적 관계`(`Transitive relations`) 라 함은,<br>`A` 에서 `B` 로 가는 관계가 있으며, `B` 에서 `C` 로 가는 관계가 있을때 `전이적 관계` 라 표현한다 

> [!note] `PCX` 는 `전이적 피어링` 이 `불가` 하다.<br><br> **`PCX` 는 두 `VPC` 만의 전용 연결 통로이며, 둘을 제외한 그 어떠한 `VPC` 트래픽도 `PCX` 에 접근하지 못한다.**

다음의 토폴로지를 보면 어떤 상황인지 이해가 갈수 있다.



