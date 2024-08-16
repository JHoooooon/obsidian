
`AWS` 를 `provider` 로 사용한다면,  `service` 내부의 모든 `functions` 는 `AWS Lambda functions` 이다.

## Configuration

`serverless service` 안의 모든 `Lambda function` 은 `serverless.yml` 의 `functions` `property` 아래에서 찾을수 있다

```yml
service: myService

provider:
	name: aws
	# nodejs14.x 버전 사용
	runtime: nodejs14.x
	# Lambda 함수의 runtime 관리에 관한 설정을 정의하는 옵션
	# Lambda 함수의 런타임 버전을 자동을 관리하고 업데이트하는 기능을 제공한다
	# `auto`
	
```