
>[!info] [[연결 서비스]] 에서 이어지는 챕터이다.

---

`On-Premise` 와 `AWS` 를 연결하는 서비스중 하나는 `DX`(`Direct Connection`) 이다.
`DX` 는 흔히 말하는 `전용회선`(`Leased Line`) 을 뜻한다. 

`Internet` 기반이 아니므로 높은 `보안성`, `안전된 대역폭` 이 주 장점이다.

>[!warning] 단, 서비스 자체가 암호화를 제공하지 않는다.

![[AWS Direct Connect.png]]

`DX Location` 은 `AWS` 가 지정한 공급 사업자의 데이터센터이다.

`Seoul` 리전의 `DX Location` 은 서울 가산(`KINX`) 와 경기 평촌(`LG U+`) 에 위치한다.

`AWS DX Location` 의 `AWS 전용 Cage` 에 `DX용 Router` 를 설치하고, `AWS DX Router` 부터 리전의 모든 가용영역까지 전용 회선으로 연결해 놓았다. 

따라서, 서울리전에서 `DX` 를 사용하려면 `DX Location` 은 가산 과 평촌 둘중 하나여야 한다.
`Customer/Parner cage` 에는 `AWS DX router` 와 `On-Premise` 를 중계할 라우터 장비가 위치한다.

>[!info]
>중계 역할은 `AWS` 의 파트너 인증 사업자뿐만 아니라 국내 모든 회선 사업자도 가능하지만 파트너가 회선 대역폭 조정 측면에서 유연하다. `2022` 년 `1`월 `DX Location` 의 파트너 인증 사업자는 드림라인, `KINX`, 세종 텔레콤, LG U+, SK Telecom 총 5개다. 

회선 사업자를 결정하면 `Customer/Pargner Router` 와 온프레미스 사이를 전용 회선으로 잇는 작업을 선행한다.

여기까지가 물리적 `DX` 연결 작업이다

>[!info] 이처럼 `AWS` 리전에서 `DX Location` 그리고, `On-Premise` 까지 모든 연결 라인은 전용선이다.

>[!note] 여기까만 공부한다. `DX` 관련해서는 시스템 관리자가 아닌이상 더이상 불필요해 보인다.<br>추가 필요하면 나중에 더 공부하도록 한다.

>[!note] 참고로, `DX` 전용 `Gateway` 인 `DXGW` 역시 존재하지만, 현재로써 `On-Premise` 를 직접적으로 다룰일이 없으므로, `DXGW` 는 넘어간다.



