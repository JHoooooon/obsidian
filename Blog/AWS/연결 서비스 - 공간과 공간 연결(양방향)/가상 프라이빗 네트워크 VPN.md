
>[!info] [[연결 서비스]] 에서 이어지는 챕터이다.

---

`VPN` 은 `Virtual Private Network` 의 약자이다.

> [!note] `VPN` 은 물리적으로 격리된 두 공간을 마치 네트워크에 있는것 처럼 이어준다.

`VPN` 은 `DX` 와 달리 `Internet` 을 기반으로 한다

>[!note] `IGW` 를 사용하지 않으며, `Internet` 기반 네트워킹이라도 터널 암호화로 안전하게 통신하므로, 일반적인 `Internet` 네트워킹과 구분한다.

## Site-to-Site (사이트간) VPN 연결 

`사이트간`(Site-to-Site) `VPN` 연결은 `IPSec 터널링` 으로 종단간 데이터를 암호화해 `On-Premise` 와 `VPC` 간 암호화된 `Network` 를 형성한다.

>[!info] [[IPSec 이란?]] 에서 `IPSec` 이란 무엇인지 볼수 있다.

`IPSec Ternaling` 은 `종단간` 데이터를 암호화해서 `On-Premise` 와 `VPC` 간 암호화된 네트워크를 형성한다

> 책에서 `1.24Gbps` 의 대역폭을 사용해서 `2개의 터널` 로 가용성을 확보한다고 말한다.

다음은 `VPN` 연결에 필요한 `3가지 요소` 이다.

![[VPN 연결 3요소.png]]

- `VGW`(`Virtual Private Gateway`):  `On-Premise` 의 `Network` 정보 `CIDR` 을 수신해 `On-Premise` 로 전송할 `VPC` 트래픽의 게이트웨이이다<br><br>`VGW` 의 패런트는 `Region` 이며, 생명주기동안 `Region` 내 다른 `VPC` 에 연결가능하다.

>[!info] `IGW` 와 유사하다

- `CGW`(`Custom Gateway`): `On-Premise` 에 설치된 고객 게이트웨이 디바이스를 의미한다.<br><br>`CGW` 를 생성시 고객 게이트웨이 디바이스(`VPN Device`) 의 `IP` 를 입력하도록 되어 있다.<br>반드시 정적 `IP` 이어야 한다

- `VPN Connection`: `VGW` 와 `CGW` 를 연결해 준다.<br> `AWS 콘솔` 에서 `VGW` 와 `CGW` 를 생성하고 `VPN` 연결 생성시 이 둘을 지정하면 연결된다.

