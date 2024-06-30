
---

## Internet Gateway

인터넷 게이트웨이(IGW)는 VPC 내부 서비스가 인터넷으로 접속하게 해주는 Resource 이다.
IGW 를 통과한 트래픽은 인터넷으로 전송된다

- **IGW  의 패런트는 Region 이고 연결대상은 VPC 이다.**
- **수명주기 동안 리전 내부의 모든 VPC 에 1:1 로 연결하고 해제할수 없다.**

![[VPC 인터넷 통신 요건]]

## Internet Gateway 와 NAT 테이블

**IGW** 는 VPC 내부 서비스가 인터넷으로 접속하게 해주는 리소스다.
다시말해 IGW 를 통과한 트래픽은 인터넷으로 전송된다.

>[!info]
**IGW 는 인스턴스에 연결되 Private IP 와 Public IP 혹은 EIP 에 대한 매핑 주소를 저장한다.**
>
>이후 서비스가 인터넷 접속을 시도하면 Private IP 를 Public IP 로 변환해 인터넷으로 보낸다.
>이는 **NAT 테이블에 프라이빗 IP 와 매핑되는 Public IP 가 없다면 트래픽을 보낼수 없는 구조**이다.

>[!warning] IGW 와 연결된 서브넷은 Public Subnet 이어야 한다.

이는 다음과 같다.

![[IGW 의 NAT 테이블 변환 원리]]

작동순서는 다음과 같다

1. **92.75.100.128 Instance 가 108.128.15.25 와 연결되고, IGW 는 두 IP 를 NAT 테이블에 저장한다.**
2. **92.75.100.128 인스턴스가 122.248.192.71 에게 트래픽을 전달한다.**
3. **IGW 는 92.75.100.128 Source IP 를 NAT 테이블에서 찾아서, Translatred IP 로 변환해 트래픽을 전송한다**
 
이를 통해 IGW  가 Public Subnet 이어야 하는 이유와 Instance 에 EIP 및 Public IP 를 사용하여 통신해야 하는 이유를 알게 되었다.

### 퍼블릭과 프라이빗 서브넷의 경계

앞에서 IGW 를 사용하기 위해서는 Public Subnet 이어야 한다고 말했다

Public Subnet 은 Public IP 및 EIP 를 가진 ENI 를 부착한 인스턴스가 IGW 로 아웃바운드 트래픽 및 인바운드 트래픽을 처리한다

그럼 Public IP 및 EIP 가 없는 일반 ENI 를 부착한 인스턴스는 IGW 와 통신할수 없는것일까?
**이는 통신할수 없다.** 

앞에서 설명했듯 **IGW 의 실행동작은 NAT Table 과 연관있으며, 이는 인스턴스의 ENI 의 Public IP 와 Private IP 의 매핑된 주소값을 가지고** 있다.

이러한 매핑 주소값을 사용하여, IGW 는 아웃바운드일때, 인스턴스 ENI 의 Private IP 를 Public IP 로 translated 한 다음 인터넷으로 요청을 보낸다

반면, 인바운드일때 IGW 는 Public IP 주소와 매칭된 Private IP 로 translated 한후, 해당 Private IP 로 인바운드된 트래픽을 보낸다. 

이것이, 현재 **IGW 와 Public Subnet 의 동작원리**이다.

> [!info] ENI 는 Subnet 에 속한다.
> **책에서 계속 강조하는건 이러한 연관관계이다. Subnet 은 ENI 의 패런트이므로, Instance 에 부착한 ENI 역시 Subnet 에 속한다.**
> 
> 그러므로 ENI 에 Public IP 혹은 EIP 를 부착하면, 이 Subnet 은 Public Subnet 이라 부르는듯하다.
> **실제로 IGW 와 통신하는 Subnet 이라는 의미도 가지고 있는듯하다.**

**반면, Private Subnet 은 IGW 와 통신하지 못한다.**

이는 Subnet 의 **ENI 가 기본 할당받은 Private IP 만을 가지며, Public IP 혹은 EIP 를 부착하지 않은 상태이므로 IGW 가 NAT Table 의 매핑된 IP 를 찾지 못한다**

>[!info] IGW 는 ENI 에 Public IP 및 EIP 가 연결되면, NAT Table 에 주소매핑 정보를 저장한다고 말한다.

**프라이빗 Subnet 은 직접적으로 IGW 와 통신할수 없으며, 간접적으로 Public Subnet 을 통한 통신은 가능하다**

**Private Subnet 은 VPN 내부에서만 통신이 가능한 Subnet 이다.**

Private Subnet 은 DB 에 대한 직접적인 접근, 혹은 인증이 필요한 서버에 대한 접근을 차단할수 있으며, 내부의 인증 시스템을 통해서만 접근 가능하도록 만들수 있다.

이러한 인증 시스템을 Client 와 Interaction 을 통해 처리하는 서버가 바로 Public Subnet 의 Server 일 것이고, 인증 처리가 완료되면, 그때서야 Private Subnet 안의 Server 로 요청을 전달한후 처리하는 방법이 필요할것이다.

즉, Client 가 직접적인 접근은 Public Subent 의 서버가 역할을 하며, 이를 통해 접근이 허가되면, Public Subnet 이 내부의 Private Subnet 의 DB 혹은 Server 에 접근하여 해당하는 정보를 받아 Client 로 제공해주는 역할을 한다

이는 다음과 같다.

![[퍼블릿 서브넷과 프라이빗 서브넷]]

위를 보면 Public Subnet 은 VPC 서브넷으로 보내는 outbound 처리가 있으며, **요청 Target 은 Local 로 되어있으며, 122.245.192.71 은 IGW-* 인것으로 볼수있다.**

이는 122.245.192.71 는 인터넷 게이트웨이를 통해 outbound 하라는 것이다.
**반면 Private Subnet 은 Target 이 Local 밖에 없다**

**이는** Private Subnet 은 Local 통신만 가능하다는 이야기이다

>[!warning] AWS 에서는 On-Link 를 Local 이라 한다. 
>[[AWS VPC 의 라우팅 개념]] 과 [[온프래미스의 라우팅 개념]] 에서 내용확인 가능하다

 **그럼 프라이빗 Subnet 은 Internet 을 사용할수 없는가?**

