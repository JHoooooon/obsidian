>[!info] [[AWS 의 ELB]] 에서 이어진 내용이다.

---

>[!info] 클라이언트 요청이 자극이라면 로드밸런싱은 반응이다. 자극을 기다리고, 반응하는것 모두 `Listener` 의 역할이다

`Listener` 는 `Port` 를 열어놓고 대기한다.

`Client` 요청이 대기중인 `ELB` 리스너의 `IP` 및 `Protocol` 쌍과 일치하면, `Targe Group` 으로 `Load Balancing` 한다.

`listener` 마다 `load balancing` 하는 `Target Group` 을 지정할 수 있다.

`Listener` 의 동작방식은 다음과 같다.

![[ALB 리스너 라우팅.png]]

1. `ELB` 리스너는 설정된 `Protocol` 을 `Listen` 한다. 

2. `Listener` 가 대기하는 `protocol` 로 요청이 들어오면 `ELB` 는 `Listener` 에 등록된 `Target Group` 의 `Port` 로 `Load Balancing` 한다.<br><br> `Listener` 마다 `load balancing` 하는 `Target Group` 이 정해져 있다.

