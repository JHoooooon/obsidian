
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
		# true 는 'Active' 와 같고 false 는 'PassThrough' 와 같다
		lambda: true
		
# 함수 적용
functions:
	hello:
		handler: handler.hello # AWS Lambda handler 설정 (필수)
		name: ${sls:stage}-lambdaName # 배포될 Lambda 이름 (옵셔널)
		
		# lambda 함수 설명 (옵셔널)
		description: Description of what the lambda function does 
		
		# provider 의 runtime 을 덮어씌운다 (옵셔널)
		runtime: python3.11
		
		# runtime 관리
		# Lambda 함수의 런타임 버전을 관리 및 업데이트 방식 설정
		# provider.runtimeManangement 값을 덮어씌운다
		runtimeManagement:
			mode: manual # auto, onfunctionUpdate, manual 3가지 선택가능
			arn: <aws runtime arn> # mode 가 manual 일때 요구되는 aws runtim arn 주소
		# 메모리 사이즈 MB (옵셔널) 
		# provider.memorySize 를 덮어씌운다
		memorySize: 512
		
		# 함수실행시 시간초과 정의 seconds (옵셔널) 
		# provider.timeout 을 덮어씌운다
		timeout: 10

		# provisioned lambda 인스턴스의 수 정의 (옵셔널)
		provisionedConcurrency: 3

		# 이 함수에 대한 예약된 동시성 제한,  
		# AWS 는 기본적으로 계정 동시성 제한을 사용
		reservedConcurrency: 5

		# 추적 설정 'Active' 또는 'PassThrough' (옵셔널)
		# provider.tracing 을 덮어씌운다
		tracing: PassThrough
```

`handler` 속성은 함수에서 실행하려는 코드가 포함된 파일과 모듈을 가리킨다

```js
// handler.js
module.exports.functionOne = (event, context, callback) => {...}
```

`handler` `property` 에 원하는 함수를 추가할수 있다.

```yml
service: myService

provider: 
	name: aws
	runtime: nodejs14.x

functions:
	functionOne:
		handler: handler.functionOne
		description: optional description for you lambda
	functionTwo:
		handler: handler.functionTwo
	functionThree:
		handler: handler.functionThree
```

`provider` `property` 에서 설정한 정의를 함수에 상속할수 있다.

```yml
service: myService

provider:
	name: aws
	# 모든 함수의 runtime 정의
	runtime: nodejs14.x
	# 모든 함수의 memorySize 정의
	memorySize: 512

# 모든 함수는 provider 에 정의된 함수 관련 프로퍼티를 상속받는다
functions:
	functionOne:
		handler: handler.functionOne
```

또한 `function` `level` 에서 `properties` 를 정의할수 있다

```yml
service: myService

provider:
	name: aws
	runtime: nodejs14.x

functions:
	functionOne:
		handler: handler.functionOne
		# 함수 레벨에서 memorySize 정의
		memorySize: 512
```

함수들에 대한 배열을 사용하여 정의할수 있으며, 함수를 다른 파일에 나누어 정의했다면 유용하게 사용할수 있다.

```yml
functions:
	- ${file(./foo-functions.yml)}
	- ${file(./bar-functions.yml)}
```

```yml
foo-function.yml
getFoo: 
	handler: handler.foo
deleteFoo:
	handler: handler.foo
```

## Permissions

모든 `AWS Lambda`  인프라 `resource` 와 상호작용하기 위해 함수는 다른 `AWS` `permission` 