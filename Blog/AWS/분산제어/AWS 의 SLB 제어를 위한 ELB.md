
---

`AWS` 도 온프레미스와 비슷한 용어를 사용한다.
`AWS` 의 `SLB` 를 `Elastic Load Balancing` 이라하고, 이 기능을 제공하는 서비스를 `ELB` 라 한다.

>[!info] [[온프레미스의 SLB 제어 - L4 스위치]] 에서 개념을 살펴볼수 있다

다음은, `L4 S/W` 와 `ELB` 를 비교한 테이블이다

| 제어요소               | L4 / L7 스위치                                    | ELB                                           |
| :----------------- | ---------------------------------------------- | --------------------------------------------- |
| **요청수신**           | Virtual Server                                 | ELB 의 IP + Listener                           |
| **가상서버(리스너) 구성요소** | Virtual IP + Protocol / Port                   | Listener 의 Protocol 과 Port                    |
| **VIP 개수**         | Virutal Server 당 1개                            | ELB 당 1개 도메인                                  |
| **포함관계**           | Virtual Server 의 모음 ⊂ L4 S/W                   | Listener 의 모음 ⊂ ELB                           |
| **요청 전달 대상**       | Target Group + Target Group 의 Protocol 과 Port  | Target Group + Target Group 의 Protocol 과 Port |
| **요청 분산 대상**       | Real Server IP + Real Server 의 Protocol 과 Port | Target + Target 의 Protocol 과 Port             |
| **다중화**            | 장비 다중화 가능                                      | ELB 다중화 불가                                    |

이는 다음처럼 그려진다.

![[L4 스위치와 ELB 의 비교.png]]

> [!info] 가상 서버마다 `다수의 서비스`(`프로토콜과 포트`) 를 생성하듯, `ELB` 도 내부에 `다수의 listener` 를 생성할 수 있다 

## ELB 의 유형

**aws 는 4개의 ELB 유형**을 제공한다

##### `Application Load Balancer`(`ALB`) 

`ALB` 는 `HTTP` 와 `HTTPS` 서비스에 최적화 되어 있다.

| 보안그룹 사용 | 대상 그룹 사용 | 대상 (그룹) 유형             | 계층  | 리스너 프로토콜    |
| :------ | -------- | ---------------------- | --- | ----------- |
| O       | O        | `인스턴스`, `IP`, `Lambda` | L7  | HTTP, HTTPS |
![[ALB 프로세스.png]]

##### `Network Load Balancer` (`NLB`)

`L4` 의 `대용량`, `고성능 트래픽` 을 분산하기에 최적화 되어 있다.

| 보안그룹 사용 | 대상 그룹 사용 | 대상 (그룹) 유형          | 계층  | 리스너 프로토콜               |
| :------ | -------- | ------------------- | --- | ---------------------- |
| X       | O        | `인스턴스`, `IP`, `ALB` | L4  | TCP, UDP, TCP_UDP, TLS |
![[NLB 프로세스.png]]
##### `Classic Load Balancer` (`CLB`)

`CLB` 는 `VPC` 가 아닌 `EC2-Classic` 플랫폼에서도 사용할수 있으나, 2013 년 이후 생성된 `AWS` 계정은 `EC2-Classic` 플랫폼 지원이 안된다.

>[!warning] `EC2-Classic` 은 2202년 8월부로 공식지원 종료된다.

| 보안그룹 사용 | 대상 그룹 사용 | 대상 (그룹) 유형 | 계층      | 리스너 프로토콜                  |
| :------ | -------- | ---------- | ------- | ------------------------- |
| O       | X        | `인스턴스`     | L4 / L7 | HTTP, HTTPS, TCP, SSL/TLS |
![[CLB 프로세스.png]]

##### `Gateway Load Balancer` (`GWLB`)

`GWLB` 는 대상이 트래픽의 최종 목적지는 아니다.
`GWLB` 는 `IPS` 나 `방화벽` 같은 `어플라이언스`(`Appliance`) 로 로드밸런싱한다

>[!info] 로드밸런싱 대상(`Appliance`) 는 트래픽 검사나 필터링을 위한 중간 경유지이며, 트래픽 최종 목적지는 아니다.
>
> `따라서 모든 트래픽을 수용해야 한다.` 

| 보안그룹 사용 | 대상 그룹 사용 | 대상 (그룹) 유형         | 계층      | 리스너 프로토콜 |
| :------ | -------- | ------------------ | ------- | -------- |
| X       | O        | `GENEVE 지원 어플라이언스` | L3 / L4 | IP       |
![[GWLB 프로세스.png]]

