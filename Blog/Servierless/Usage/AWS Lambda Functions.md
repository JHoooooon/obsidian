
`AWS` 를 `provider` 로 사용한다면,  `service` 내부의 모든 `functions` 는 `AWS Lambda functions` 이다.

## Configuration

`serverless service` 안의 모든 `Lambda function` 은 `serverless.yml` 의 `functions` `property` 아래에서 찾을수 있다

```yml
service: myService

provider:
	name: aws
	# nodejs14.x 버전 사용
	runtime: nodejs14.x
	
	# Lambda 함수의 runtime 관리에 관한 설정을 정의하는 옵션 (옵셔널)
	# Lambda 함수의 런타임 버전을 자동을 관리하고 업데이트하는 기능을 제공한다
	# `auto`: 자동으로 최신 런타임 버전으로 업데이트
	# `onfunctionUpdate`: 함수 코드가 변경될대만 런타임 업데이트
	# `manual`: 수동으로 런타임 버전을 제어
	runtimeManagement: auto

	# 메모리 사이즈 MB (default: 1024) (옵셔널)
	memorySize: 512

	# 함수 실행시 시간초과 정의 seconds (default: 6)
	timeout: 10

	# 함수 버저닝 여부 (default: true)
	versionFunctions: false

	# 함수 추적
	tracing:
		# 모든 함수를 추적을 활성화 (옵셔널)
		lambda: true
		
# 함수 적용
functions:
	hello:
		handler: handler.hello # AWS Lambda handler 설정 (필수)
		name: ${sls:stage}-lambdaName # 배포될 Lambda 이름 (옵셔널)
		
	
```