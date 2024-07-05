
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

![[전이적 라우팅 시도.png]]

보면 `24.183.75.0/24` 의 서브넷은 `37.120.0.0/24` 로 `Traffic` 을 보내기 위해 `pcx-1*` 로 `Target` 을 설정했다.

이는 다음처럼 거쳐가기(`전이적`)를 원한다

`24.183.75.0/24` -> `pcx-2` -> `92.75.0.0/16` -> `pcx-1*` -> `37.120.0.0/24`

`Routing Table` 만으로 보면, 될듯 하지만, `되지 않는다`
이는, 앞에서 말한대로 `PCX` 의 특성으로 인해 `직접 연결된 PCX` 의 `VPN` 만 연결가능하기 때문이다.

이를 연결하기 위해서는 다음처럼 처리해야 한다.

![[PCX 주체간 통신.png]]

이는 `VPC` 주체와의 직접적인 연결이 가능하도록 `PCX` 를 추가 연결한것이다

## Full-mesh 피어링

`PCX` 의 [[#전이적 VPC 피어링 - 불가]]] 이유로 인해, 간단해지지만 `VPC` 가 많아지면 많아질수록 복잡한 구성이 발생할수 있다.

아래는 모든 `VPC` 가 서로 통신할수 있는 상황이라 가정하고 `VPC` 개수당 `PCX` 가 몇개 필요한지 보여주는 예이다.

`VPC=2, PCX=1`
`VPC=3, PCX=3`
`VPC=4, PCX=6`
`VPC=5, PCX=10`

모든 `VPC` 가 연결되어 통신하도록 만든것을 `Full-mesh` 형태라고 한다
이러한 예시를 보면 다음처럼 계산될수 있다

`VPC 가 N 일때,`
`PCX = N(N-1)/2`

이는 `VPC` 가 많아질수록 `PCX` 가 엄청나게 많아질수 있다.
`AWS` 에서는 이러한 단점을 보완하고자 `TGW` 를 출시한다

`TGW` 는 자신에게 연결된 모든 공간을 서로 이어주는 `Full-Mesh` 구성이 불필요하며, `연결`(`Attachment`) 개수는 `VPC` 개수만큼이다.

위에서 `VPC = 5` 일때 `PCX = 10` 개 필요하지만 `TGW` 는 `5` 면 된다 

>[!warning] `TGW` 는 트래픽 전송 비용과 별개로 유지 비용이 `시간당 0.05 ~ 0.09 USD` 가 부과 되므로 소수의 `VPC` 간 연결이나 `Network` 추가 확장 계획이 없다면 `PCX` 를 권장한다 

## PCX 의 특징 및 기능 정리

- `VPC` 피어링 가능한 대상 `VPC` 의 위치는 리전과 무관하다

- `PCX` 는 두 `VPC` 만의 전용 통로이다.<br> 다른 `VPC` 트래픽이 `PCX` 를 통과하지 못한다. 즉 `VPC` 트래픽은 `PCX` 를 경유해 `전이적 관계` 를 형성할수 없다 

- 두 `VPC` 를 제외한 다른 모든 `VPC` 는 둘만의 `PCX` 를 `Routing Taget` 으로 사용할수 없다.

- `3개 이상의 VPC` 를 하나의 `PCX` 로 연결할수 없다.

- 두 `VPC` 간 서로 다른 `PCX` 를 생성할 수 없다. `VPC` 사이의 `PCX` 는 유일하다.

- `Source VPC` 를 `요청자`(`Requester`) `VPC` 라 하고 연결을 수락한 `VPC` 를 `수락자`(`Accepter`) 라 한다. 누가 `요청` 했는지의 여부만 다를뿐 기능상 우열은 없다

- `Requester` 와 `Acceptor` 모두 `PCX` 를 삭제할 수 있다. 한쪽에서 `PCX` 를 삭제하면 그 `Peering` 은 유효하지 않다. 

- `PCX` 로 연결된 두 `VPC` 의 `Account` 와 `Region` 이 같으면 `SG` 의 `source/dest check` 를 허용할수 있다.

- `Peering` 이 형성되더라도 `Subnet Routing` 을 지정해야 네트워킹 가능하다 

- 보안을 강화하기 위해서는 네트워킹이 필요한 `Subnet` 만 `PCX` 네트워킹 타깃으로 지정한다

- 라우팅 대상을 넓은 범위(`VPC CIDR`) 로 설정하면 안된다. 네트워킹이 필요한 영역(`CIDR`) 로만 한정한다





