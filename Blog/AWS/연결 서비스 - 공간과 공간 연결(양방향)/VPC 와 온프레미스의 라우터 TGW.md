
---

`PCX` 는 `VPC` 를 연결하고, `VPN` 및 `DX` 는 `VPC` 와 `On-Premise` 사이를 연결한다 
`PCX` 는 `VPC` 간 `1:1` 연결만 가능하며, `VPN` 역시 `IPSec` 터널 종단간 연결만 가능하다.

`DX` 는 `DXGW` 로 `1:1` 한계를 극볼할수 있지만 다수 `VPC` 를 온프레미스와 잇는 게이트웨이 역할만 수행할뿐, 서로간 트래픽 송수신은 불가능하다

`AWS` 는 이러한 단점을 개선하고자 `전송 게이트웨이` `Transit Gateway`(`TGW`) 를 만들었다.

> [!note] `TGW` 에 참여하는 각 연결 주체들은 다른 모든 연결을 대상으로 트래픽을 전송 또는 수신한다

## 전송 게이트웨이(TGW) 개요

> [!note] `TGW` 는 `Region` 의 중앙 네트워크 허브다.<br>상호간 트래픽 교환 목적으로 `TGW` 에 참여하는 리전 내부의 각 주체를 `Resource` 라 한다.

다음은 `Resource` 유형이다 

| 유형  | 리소스 (Attachment Type)         | 목적                  | 연결(Attachment) 생성메뉴                                            |
| :-- | ----------------------------- | ------------------- | -------------------------------------------------------------- |
| A   | VPC                           | VPC 연결              | VPC -> Transit Gateway 연결 -> Create Transit Gateway Attachment |
| B   | VPN                           | VPN 연결              | VPC -> Transit Gateway 연결 -> Create Transit Gateway Attachment |
| C   | Peering Connection (PCX)      | TGW 간 피어링           | VPC -> Transit Gateway 연결 -> Create Transit Gateway Attachment |
| D   | Connect                       | 가상 어플라이언 연결(GRE 패킷) | VPC -> Transit Gateway 연결 -> Create Transit Gateway Attachment |
| E   | DXGW (Direct Connect Gateway) | DX 연결               | Direct Connect -> Direct Connect 게이트웨이 -> TGW 선택               |

`TGW` 에 연결할때 `VGW` 를 사용하지 않는다.
`VPN` 과 `DX` 연결을 `TGW` 의 `Resource` 로 사용하면 `VGW` 가 생략되고 `TGW` 자체가 `VPC` 라우팅 타깃이 된다.

>[!note] `TGW` 참여 의사를 밝히는 행위를 `Attachment 를 생성한다` 라 하고, 참여 의사를 표시한 각 리소스를 연결하는것을 `Association` 이라 한다.

`VPC Attachment` 생성 단계에서 최소 1개 가용영역을 선택해야 한다. 
해당 가용영역마다 1개 서브넷을 선택할수 있다.

이렇게 선택하면, `TGW Attachment ENI` 가 생성된다.
이 `ENI` 를 통해 각 가용영역에 포한된 모든 서브넷의 통신 거점이 된다.

>[!info] `VPC Attachment` 생성뒤 다른 `VPC` 변경은 불가능하지만, 가용영역이나 서브넷 변경은 가능하다.

`Association` 은 통시주체를 참여시킨것이라고 볼수 있다.
`Attachment` 는 `대상 리소스`라고 보면 되고, `Association` 은 `대상 리소스의 통신 주체` 를 연결하는 행위이다.

이는 라우팅 테이블을 통해 연결하며, `Region` 수준의 `TGW` 는 `1` 개 이상의 `Routing Table` 생성이 가능하다.

`Subnet` 을 `Routing Table` 에 `Associate` 하듯, `VPC` 도 `TGW` 의 `Routing Table` 에 `Associate` 한다.

`VPC Association` 은 `TGW` 에 참가한 `VPC` 연결(`Attachment`)을 라우팅 테이블에 연결(`Association`) 한다.

---

`TGW 통신 요건`은 다음과 같다

- `TGW`  에 참여하면 다른 참여 주체들와 통신할수 있는 자격이 생긴다.

- `TGW` 에 참여하려면 `TGW`  와 리소스를 한쌍으로 만든 `Attachment` 를 `TGW` 라우팅 테이블에 `Association` 해야 한다.

- 각 `Association` 리소스의 `CIDR`를 `Routing Table` 에 등록해야 한다.

---

`TGW` 는 일련의 작업을 간소화 하고자 2가지 옵션을 제공한다

- 기본 라우팅 테이블에 `Association`

- 기본 라우팅 테이블에 `Propagation`

---

이 옵션중 하나라도 `Enable` 되면 `Default Routing Table` 이 생성된다.
모두 `Desabled` 되면 생성되지 않는다.





