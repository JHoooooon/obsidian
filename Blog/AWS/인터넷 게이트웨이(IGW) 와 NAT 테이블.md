
---

## Internet Gateway

인터넷 게이트웨이(IGW)는 VPC 내부 서비스가 인터넷으로 접속하게 해주는 Resource 이다.
IGW 를 통과한 트래픽은 인터넷으로 전송된다

- **IGW  의 패런트는 Region 이고 연결대상은 VPC 이다.**
- **수명주기 동안 리전 내부의 모든 VPC 에 1:1 로 연결하고 해제할수 없다.**

![[VPC 인터넷 통신 요건]]

## Internet Gateway 와 Nat 테이블

**IGW** 는 VPC 내부 서비스가 인터넷으로 접속하게 해주는 리소스다.
다시말해 IGW 를 통과한 트래픽은 인터넷으로 전송된다.

인터넷으로 전송되기전, VPC 내부의 Instance 는 Local IP 이므로, IGW 의 IP 로 변환해주어야 한다.
즉 Source 값이 Instance IP 에서 