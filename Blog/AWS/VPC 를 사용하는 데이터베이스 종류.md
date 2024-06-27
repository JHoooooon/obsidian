<hr>

#vpc #aws #rds #eni 

> [ 참고책 ] AWS VPC 네트워킹 원리와 보안

##  VPC 네트워킹을 사용하는 서비스

| DB 서비스<br>               | VPC 기반 | RDS 용 ENI | 퍼블릭 엑세스 |
| :----------------------- | ------ | --------- | ------- |
| RDS                      | o      | o         | o       |
| Neptune                  | o      | o         | x       |
| Amazon DocumentDB        | o      | o         | x       |
| ElasticCache             | o      | x         | x       |
| Redis 용 Amazon Memory DB | o      | x         | x       |
| DynamoDB                 | x      | x         | x       |
| Amazon QLDB              | x      | x         | x       |
| Amazon Keyspaces         | x      | x         | x       |
| Amazon Timestream        | x      | x         | x       |

VPC 네트워킹을 사용하는 서비스는 총 5개이며, 이는 VPC 보안 통제영역에 있다

이중 **ENI** 를 사용하는 **DB** 는 **RDS용 ENI** 를 사용한다.
이러한 **RDS용 ENI** 를 사용하는 **DB** 는 총 3개로 **RDS, Neptune, Amazon DocumentDB** 이다.

> 책에서는 **SDK** 를 사용할때 주의점이 있다고 한다.
> 
> **RDS용 ENI** 를 사용하는 위의 3 가지 **DB** 는 **ENI** 의 정보(설명)이 **RDSNetworkInterface** 으로 작성되었으므로, 이를 보고 **RDS DB** 라 생각하면 안된다는것이다.
>
> 구분하기 위해서는 데이터베이스 종류를 보고 구분하라고 한다.

인터넷에서 접속 가능한 퍼블릭엑세스 기능은 **RDS** 에만 있다





