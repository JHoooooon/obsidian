
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

>[!warning] 그럼 프라이빗 Subnet 은 Internet 을 사용할수 없는가?
>
**결론적으로 말하면 사용할수도 있다.** 하지만 Private Subnet 만으로는 힘들다
이를 사용하기 위해서는 NAT Gateway 가 필요하다
