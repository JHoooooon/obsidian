
---

## VPC 네트워킹의 개념

> ENI 를 사용하면서, Security Group, ACL, Routing Table 의 통제 영역에 있으면 VPC 네트워킹을 사용한다고 말한다
>
> ENI 의 존재만으로도 VPC 네트워킹 사용을 확신할수 있다

여기서 말하는 통제 3요소인 **Security Group, ACL, Routing Table** 은 **ENI 를 보호**하고 **트래픽의 방향을 안내**한다.

## 접근제어

**Access Control** 은 필요한 트래픽만 허용하고 불필요한 트래픽의 접근은 차단하여, 컴퓨팅 서비스를 보호하는 장치이다

**VPC** 는 **보안그룹과 ACL 로 이러한 트래픽의 허용 / 차단하는 역할**을 한다

보통의 온프레미스 환경의 트래픽 접근 허용은 FireWall 이 처리한다.
이러한 Firewall 의 접근 제어 방식은 총 2가지이다

1. **White List**
2. **Black List**

White List 방식은 접근을 허용하는 트래픽을 제외하고는 모두 차단한다
반면, Black List 방식은 접근을 차단하는 트래픽을 제외하고는 모두 허용한다

반면 이러한 방식을 사용하여 하이브리드 접근 방식을 사용하기도 한다

1. **White List based**
2. **Black List based**

![[Hybrid 방식 접근제어]]

**White List based** 방식은 **All Deny 규칙을 최하단에 놓고, 그 위로 Allow 및 Deny 규칙을 혼합해서 배치한다**

**Black List based** 방식은 **All Allow 규칙을 최하단에 놓고, 그 위로 Allow 및 Deny 규칙을 혼합해서 배치한다**

모든 트래픽을 기본 허용하는 2가지 블랙 방식은 관리자가 차단 대상을 모두 알고 있어야 하므로 관리가 까다로워 단독 사용이 어렵다

따라서, 화이트 리스트 방식과 이중 보안 체계를 구성하는게 좋다.

여기서 중요한 부분은 **Rule Number** 가 있다는것이다.
다음을 보면 **Whitelist based** 를 사용하여 다음처럼 구성했다고 하자












