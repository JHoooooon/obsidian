
---
[IPSec 이란 무엇인가요?](https://aws.amazon.com/ko/what-is/ipsec/) 에서의 내용을 정리하면 다음과 같다 

> [!note]  
`IETF`(`Interner Engineering Task Force`) 에서는 `Public Network` 엑세스시 데이터의 `기밀성`, `무결성`, `신뢰성` 을 보장하기 위해 `1990` 년대에 `IPSec` 을 만들었다.
> 
> `IP Protocol` 은 전송되는 방식을 결정하는 공통 표준으로, `IPSec` 은 암호화와 인증을 추가하여 포로토콜을 더욱 안전하게 만든다.<br><br>`IPSec` 은 데이터 소스를 `암호화` 한 다음 대상에서 `복호화` 한다

`IPSec` 은 중요한 정보를 암호화해서 `3자` 에 의해 모니터링 되지 않도록 암호화하는 기술이라고 볼수있다.
`IPSec` 은 `비대칭 암호화` 와 `대칭 암호화` 둘다 사용한다. 

>[!note] `비대칭 암호화` 는 `Public Key` 와 `Private Key` 를 사용하여, `Public Key` 로 암호화된 내용을 `Private Key` 로 복호화하는데 사용한다.

>[!note] `대칭 암호화` 는 `복호화` 및 `암호화` 둘다에 같은 `Public Key` 를 사용하여 처리한다

`IPSec` 으로 `보안 연결` 을 설정하고, `대칭 암호화` 로 전환하여 데이터 전송 속도를 높힌다.

## IPSec 프로토콜

`IPSec` 프로토콜은 `Data Packet` 을 통해 전송한다.
`Data Packet`  은 `Network` 전송을 위해 정보를 포맷하고 준비하는 특정 구조이다.

구조는 다음과 같다

- `Header`: `데이터 패킷` 을 올바른 대상으로 라우팅하기 위한 메타정보

- `Payload`: `Data Packet` 의 실제 정보

- `Trailer`: `Data Packet` 의 끝을 나타내기 위한 `Padding` 값 

그리고 다음과 같은 `IPSec` 프로토콜 정보가 작성된다

- `AH`(`Authentication Header`): 인증 해더는 발신자 인증 데이터가 포함된 헤더를 추가하고, 권한이 없는 당사자가 수정하지 못하도록 패킷 컨텐츠를 보호한다<br><br> 데치어 패킷을 수신할때, `Payload` 암호화 해시 계산 결과와 비교해서 두 값이 일치하는지 확인한다.<br>암호화 해시는 데이터를 고유한 값으로 요약하는 수학 함수다


>[!note] 개인적으로 `JWT` 의 `Verify Signature` 가 `AH` 로 생각된다.

- `ESP`(`Encapsulation Security Payload`): 선택한 [[#IPSec 모드]]에 따라, `ESP` 프로토콜은 `IP` 패킷 또는 `Payload` 에 대해서만 암호화를 수행한다. `ESP` 는 암호화할때 데이터 패킷에 헤더와 트레일러를 추가한다.

>[!note] 내용을볼때, `ESP` 가 `Packet` 의 암호화를 수행하면서, `Packet` 에 `header` 와 `Trailer` 를 추가하는 캡슐화 작업을 하는듯하다.

- `IKE`(`Internet Key Exchange`):  `Internet Key Exchange` (`IKE`) 는 두 디바이스간에 보안 연결을 설정하는 프로토콜이다. 이는 후속 데이터 패킷을 송수신하는 보안 연결을 설정한다.

>[!note] 두 디바이스간의 보안 연결을 유지하기 위한 정보를 담는듯하다.

## IPSec 모드

`IPSec` 은 두가지 모드가 존재한다

- `Ternal`: `IPSec` 터널 모드는 권한이 없는 당사자로 부터의 데이터 보호를 강화한다.<br><br>`Public Network` 의 `Data` 를 전송하는데 적합하며, 컴퓨터가 페이로드와 헤더를 포함한 모든 데이터를 암호화하고, 데이터에 새 헤더를 추가한다.

>[!note] `Ternal` 은 `Internet` 에서 정보를 전송할때 사용

- `Transport`: `IPSec` 의 `전송` 모드는 오직 `Packet` 의 `Payload` 만 암호화한다.<br>그리고 `IP Header` 의 기존 포멧은 그대로 유지한다.<br><br>암호화 되지 않은 `Packet Header` 를 통해 `Router` 는 각 데이터 패킷의 대상 주소를 식별할수 있다.그러므로, `IPSec` 전송은 두 컴퓨터 간의 직접 연결을 보호하는 경우처럼, 가깝고 신뢰할수 있는 네트워크에서 사용된다.

>[!note] `Transport` 는 `AWS` 내부 에서 사용하거나, `신뢰할수 있는 네트워크` 시에 사용 <br><br>갠적인 이해로는, `Ternal` 은 패킷의 페더  및 데이터 전부를 `암호화` 하므로, `Transport` 방식보다 `overhead` 가 더 가해지지 않을까 싶다.












