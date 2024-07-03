
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

`Application Load Balancing`(`ALB`) 

![[ALB 프로세스.png]]