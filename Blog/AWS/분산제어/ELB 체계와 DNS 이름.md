
>[!info]  [[NLB  대상 그룹과 ALB 연결]] 와 이어지는 내용이다.
---

>[!info] 클라이언트 요청을 수신하는 `ELB` 수신부를 본다

무엇이든 `ELB` 요청 주체가 될수 있다. `Client` 나 `On-Premise` 시스템, `AWS` 서비스등 모든 컴퓨팅이 `ELB`  클라이언트다.

`ELB` 는 어떤 클라이언트라도 쉽게 `Access` 하도록 다음 기능을 제공한다

- `ELB` 전용 `DNS 이름` 을 사용한다.<br> `DNS 이름` 은 `ELB` 마다 유일하며 변하지 않는다.<br>클라이언트는 모든 `Node` 의 `IP` 를 몰라도 `DNS` 이름만으로 `ELB` 에 접속할 수 있다.

- `ELB` 는 2가지 요청 유형을 제공한다 이를 `체계`(`Schema`) 라 한다.
	1. `Internet 체계`: `Internet` 상의 `Client` 가 `IGW` 를 통과해 `ELB` 로 접속시 사용한다.<br>`IGW` 의 `NAT` 테이블에 등록된 `Public IP` 가 `Node` 에 할당되 있어야 한다. 
	
	2. `Internal 체계`: `Client` 가 `IGW` 를 통과하지 않고 `ELB` 로 접속할때 사용한다.<br>그러므로 `Public IP` 나 `EIP` 는 없다.<br>`Client` 는 `Region` 위치와는 무관하며 `VPC` 내부 또는 동일 계정의 다른 `VPC` 나 다른계정의 `VPC` 에 존재할 수도 있다.<br><br>또 `Direct Connect`, `VPN` 등 `VGW` 를 경유해 접속할때도 사용한다.<br>이처럼 `VPC` 네트워킹을 사용하지 않는 `Client` 도 내부체계 `ELB` 로 트래픽을 요청할 수 있다.

>[!info] 간단히 말하면 구분 기준은 `IGW` 를 통과했는지 여부이다.

다음은 `Internet 체계` 에 대한 정리이다.

![[인터넷 체계 ALB 의 DNS 이름 호출.png]]

1. `ALB` 는 `Variable Node` 라고 했다. 이는 `Node` 의 `IP` 가 변경될수 있음을 암시한다.
혹시 새로 생성된 `Node` 나, 교체된 `Node` 가 있을경우 `Node`(`ENI`) 는 변경된다.

2. 이러한 변경을 감지하여,  `Amazon DNS` 에 `DNS 이름`  `http://my-alb-44....` 에 `Public IP` 를 매핑할수 있도록 `Update` 처리한다.

3. `Clinet` 가 `http://my-alb-44....` 으로 쿼리하면, `DNS` 는 `Amazon DNS` 를 통해 받은 `Resolved IP` 로 반환한다. 

>[!info] 중요한 부분은, `ELB` 에서 사용하는 2개의 `Node` 의 `IP` 가 번갈아 가면서 선택된다는 점이다.

4. 이렇게 받은 `IP` 를 사용하여 `Internet` 을 통해 요청을 하면 `IGW` 에서 `IP` 에 매핑된 `Original IP` 로 변환한후, 트래픽을 `Original IP` 로 전달한다.

5. `ALB` 는 받은 `IP` 를 수신하고, 대상 그룹으로 `Load Balancing` 한다.


다음은 `Internal 체계`  의 경우이다.

![[Internal 체계 ALB 의 DNS 이름 호출.png]]

보면 `IGW` 를 통한 `Public IP` 에서 `Original IP` 로 변환하는 과정이 없다.
`Internal` 에서는 `Private IP` 로만 통신하기에, `DNS Server` 에서는 `Private IP` 로 `DNS` 가 매핑된다.

기본적인 프로세스는 동일하다. `Public IP` 로 접속하느냐, 아니냐의 차이라고 봐도 될듯하다.

> [!info] [[NLB  대상 그룹과 ALB 연결]]] 에서 말한것처럼 [[가변 노드를 장착한 ELB (ALB, CLB)]]] 로 인해 `ALB`, `CLB` 는 `EIP` 가 아닌 `Public IP` 가 할당된다.<br><br>`NLB` 는 `고정Node` 를 사용하므로 `EIP` 사용이 가능하다

이는 다음의 테이블로 요약가능하다

| 특징            | ALB | NLB | CLB | GWLB |
| :------------ | :-- | :-- | --- | ---- |
| 인터넷 체계 사용 가능  | O   | O   | O   | X    |
| 노드에 EIP 연결 가능 | X   | O   | X   | X    |
