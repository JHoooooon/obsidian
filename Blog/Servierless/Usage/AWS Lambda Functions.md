
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

모든 `AWS Lambda`  인프라 `resource` 와 상호작용하기 위해 `Lambda` 는 다른 `permission` 이 필요하다
이러한 `Permission` 은 `AWS IAM Role` 을 통해 설정한다.

이는 `provider.iam.role.statements` `property` 를 통해 `policy` 를 설정할수 있다.

```yml
service: MyServcie

provider:
	name: aws
	runtime: nodejs14.x
	iam:
		role:
			statements:
				# permission 을 허용하는 정책
				- Effect: Allow
				  # 허용할 permission 액션
				  Action:
					  # dynamodb 관련 액션 정의
					  - dynamodb:DescribeTable
					  - dynamodb:Query
					  - dynamodb:Scan
					  - dynamodb:GetItem
					  - dynamodb:PutItem
					  - dynamodb:UpdateItem
					  - dynamodb:DeleteItem
				  # 액션을 허용할 resource arn
				  Resource: 'arn:aws:dynamodb:us-east-1:*:*'

functions:
	functionOne:
		handler: handler.functionOne
		memeorSize: 512
```

```yml
servcie: myService
provider:
	name: aws
	iam:
		role:
			statements:
				# permsssion 을 허용하는 정책
				- Effect: 'Allow'
				  Action:
					# 허용할 액션
					- 's3:ListBucket'
				  Resource:
					  # CloudeFormation 문법을 넣을수 있다
					  # 이러한 문법은 CloudeFormation 으로 변환된다는점을 기억하라고 말한다
					  # Fn::Join: 문법으로, 첫번째는 구분자를 지정하고,
					  # 두번째 원소로 배열을 받아 배열의 원소들을 첫번째 원소의 구분자로 합친다
					  {
						  'Fn::Join':
							  [
								  '',
								  [
									  'arn:aws:s3:::', 
									  { 'Ref': 'ServerlessDeploymentBucket' },
								  ],
							  ],
					  }
				- Effect: 'Allow'
				  Action:
					  - 's3:PutObject'
				  Resource:
					  Fn::Join:
						  - ''
						  - - 'arn:aws:s3:::'
						    - 'Ref': 'serverlessDeploymentBucket'
							- '/*'
functions:
	functionOne:
		handler: handler.functionOne
		memorySize: 512
```

원래 존재하는 `IAM role` 을 사용하고 싶다면, `iam.role` `property` 에 `IAM Role` `ARN`   를 추가할수 있다

```yml
service: new-service
provider:
	name: aws
	iam:
		role: arn:aws:iam::YourAccoutNumbeR:role/YourIamRole
```

## Lambda Function URLs

Lambda 함수를 `HTTP(S)` `endopint` 로 노출시켜 웹 요청을 받을수 있도록 해야 한다.
이때 사용하는 설정이 `url` 이다. 

이 설정은 `API Gateway` 를 자동으로 설정하고, `Lambda` 함수에 대한 `HTTP(S)` `URL` 을 생성하여 외부에서 호출할수 있게 해준다.

다음은, `url` `property` 를 사용하여 `CORS` 없이 `Public` `URL` 을 생성하게 한다.

```yml
functions:
	func:
		handler: index.hanlder
		url: true
```

`authorizer`, `cors` 와 `invokeMode` 옵션을 추가하여 설정을 변경할수도 있다.

```yml
provider:
	name: aws
	# 
	iam:
		role:
			statements:
				- Effect: Allow
				  Action:
					  - lambda:InvokeFunctionUrl
				  Resource:
					  - arn:aws:lambda:${opt:region, 'us-east-1'}:${aws:accountId}:function:${self:service}-${opt:stage}-func
		

functions:
	func:
		handler: index.handler
		url:
			authorizer: aws_iam
```

`IAM Authorization` 을 사용하는 경우 `URL`  은 `Lambda:InvokeFunctionUrl` 을 허용하는 `AWS` `credentials` (자격증명) 이 있는 `HTTP` 요청만 허용한다. 

또한, `CORST headers` 를 사용하여, `function` `URL` 을 다른 `domain` 에서 호출되게 할수 있다.
`cors` 를 `true` 로 해주면, 모든 `domain` 을 허용한다.

```yml
functions:
	func:
		handler: index.handler
		url:
			cors: true
```

이는 기본적으로 다음의 `table` 의 `HTTP Header` 내용을 가진다.

| Header                       | Value                                                                   |
| :--------------------------- | ----------------------------------------------------------------------- |
| Access-Control-Allow-Origin  | *                                                                       |
| Access-Control-Allow-Header  | Cntent-Type, X-Amz-Date, Authorization, X-Api-Key, X-Amz-Security-Token |
| Access-Control-Allow-Methods | *                                                                       |
다음처럼 `allowedOrins` , `allowedHeaders`, `allowedMethods`, `allowCredentials`, `exposedResponseHeaders` , `maxAge` 속성을 설정하여, `CORS` 설정을 추가로 조정할수 있다.

```yml
functions:
	func: index.handler
	url:
		cors:
			allowedOrigins: # 허용 origin
				- https://url1.com
				- https://url2.com
			allowedHeaders: # 허용 header
				- Content-Type # content-type Header
				- Authrization # authorizaion Header
			allowedMethods: # 허용 메서드
				- GET # GET 메서드
			alloweCredentials: true # 자격증명 허용
			exposedResponseHeaders: # 클라이언트에게 노출하고자 하는 응답 헤더 
				- Sepcial-Response-Header
			maxAge: 6000  # preflight 요청에 대한 브라우저가 응답을 캐시할수 있는 시간
						  # In Seconds (초단위)
```

>[!info] exposedResponseHeaders
> 
> `CORS` 는 브라우저가 다른 도메인에서 `resource` 를 요청할때, 보안상의 이유로 제한을 둔다.
> 기본적으로, 브라우저는 요청에 대한 응답에서 특정 `header` 만을 접근할수 있다.
> 
> `Content-Type`, `Cache-Control`, `Date` 같은 기본적인 `header` 는 클라이언트가 접근할수 있지만, 사용자 정의 헤더나 특정 응답 헤더는 접근할수 없다
> 
>`Access-Control-Expose-Header` 는 서버가 클라이언트에게 노출하고 하는 특정 응답 헤더를 명시한다. 
>

위의 `cors` `properties` 는 다음의 `CORS Header` 를 보여준  `table` 과 같다

| Configuration property | CORS Header                      |
| :--------------------- | -------------------------------- |
| allowedOrigins         | Access-Control-Allow-Origin      |
| allowedHeaders         | Access-Control-Allow-Headers     |
| allowedMethods         | Access-Control-Allow-Methods     |
| allowCredentials       | Access-Control-Allow-Credentials |
| exposedResponseHeaders | Access-Control-Expose-Headers    |
| maxAge                 | Access-Control-Max-Age           |
이는 `CORS` 설정에서 제거하는것도 가능하다. 
제거하기 위해서는 `NULL` 로 설정하면, 기본적으로 설정되는 값을 제거한다.

```yml
functions:
	func:
		handler: index.handler
		url:
			cors:
				allowedHeaders: null
```

`invokeMode` `property` 는 `스트림 응답` (`streaming response`) 을 활성화하는 `RESPONSE_STREAM` 을 설정할수 있다.

만약, 따로 지정하지 않는다면, 기 `BUFFERED` 호출 모드로 가정된다.

```yml
fucntions:
	func:
		handler: index.handler
		url:
			invokeMode: RESPONSE_STREAM
```

>[!info] RESPONSE_STREAM
>`Lambda` 함수가 스트리밍 응답을 반환할때 사용한다.
>이 모드에서 `response` 는 `stream` 으로 처리되어 `client` 에 연속적으로 전달된다.
>
>대량의 데이터를 처리하거나 긴 실행시간이 필요한 작업에 적합하다.
>파일 다운롣, 대규모 데이터 처리, 실시간 로그 스트리밍등에 유용하다

>[!info] BUFFER
>
>`Lambda` 함수가 전체 `response` 를 `buffer` 에 저장한후 `client` 에 반환할때 사용한다.
>
>응답 데이터가 비교적 작고 전체 `response` 를 한번에 전달할수 있는 경우 적합하다.
>`JSON`, `Text file` 등 상대적으로 작은 데이터 크기를 처리할때 유용하다
>
>물론, 이는 `BUFFER` 의 응답크기는 최대 `64MB` 로 제한되며, 이는 `Client` 에게 반환될때 최대 크기이다.
>`BUFFER` 응답은 이 크기 한도내에서 응답을 버퍼링하고 클라이언트에 전송한다.
>
>버퍼링은, `Chunk` 로 나누어 보내는 방식을 말한다.
>이말은 `BUFFER` 라고 해서 항상 하나의 파일을 모아서 한꺼번에 보내지는 않는다.
>파일 크기가 하나의 `BUFFER` 의 기본크기에 들어가기 어려우면 `CUHNK` 라는 덩어리로 쪼개서 보내기도 한다
>단, `Lmabda` 에서는 `64MB` 로 제한한것 뿐이다.
>
>만약 `64MB` 보다 큰 데이터이면 `STREAM` 으로 보내는것이 좋다
>참고로, `Lambda` 에서는 자동적으로 `BUFFER` 의 기본 크기를 관리하므로, 신경쓸 필요는 없다

## Referencing container image as a target

`Lambda` 환경을 `docker image` 로 교체하여 설정할수있다.
`image` 는 `AWS ECR` 저장소에서 참조된다

추가적으로, `local` 로 구축되어 `AWS ECR` 저장소에 업로드될 자체 이미지를 정의할수도 있다.

`Serverless` 는 `image` 를 통한 `ECR` 저장소를 생성한다.
**그러나, 현재는 업데이트 관리를 하지 않는다고 말한다.** (😱 어쩌라는거지?)

>[!warning] 😱 뭐 어쩌라는거지???
>
>내가 번역체로 해석해서 뜻이 애매한데,
>`현재는 업데이트 관리를 하지 않는다` 라는 말은 `ECR` 에 `push` 될때마다 `CVE` `Scan` 이 수행되므로 `AWS` 에서 관리한다는 말이다. 
>
>`sls` 에서는 `repository` 의 업데이트 관리나 `CVEs` 와 같은 보안 관리 기능을 직접 처리하지 않는다
>
>이는 다음의 내용에 더 나온다.
>
>일일히 찾아보지 않으면, 이러한 내용을 이해하는데 약간 애매해진다
>`Docs` 를 원서로 읽다보니, 더 애매한 부분이 있다... ㅠㅠ

 `image` 와 함께 설정된 `function`  이 처음 배포되는 경우 또는 새로운 `service` 에 대해서 생성되는 경우에만 `ECR` 저장소는 생성된다

`service` 설정 상에, `provider.ecr.scanOnPush` `property` 를 통해 `CVEs` 에서 `scan` 을 통해 `ECR` 저장소를 설정할수 있다.

>[!warning] `provider.ecr.scanOnPush` 는 `defalut` 로 `false` 이다

>[!info] CVEs (Common Vulnerabilities and Exposures) 
>해석하자면 `일반적으로 알려진 보안 취약점과 노출` 을 식별하기 위한 표준화된 방법이다.
>
>`AWS Elastic Container Registry` (`ECR`) 에서는 `container` `image` 의 보안 취약점을 관리하기 위해 `CVE` `Scan` 기능을 제공한다.
>
>이 기능을 사용하면, 이미지가 `ECR` 에 `Push` 될때 보안 취약점을 자동으로 검사하고, 발견된 취약점에 대한 정보를 제공받는다.
>
>`provider.ecr.scanOnPush` 는 이러한 `CVE` 스캔을 활성화할지 여부를 결정한다.

`service` 설정상에, `image` 는 `provier.ecr.images` 를 통해 설정한다.

`local` 로 정의한다면, `path` `property`  지정이 필요하며, 이는 유효한 `docker context` `directory` 를 가리켜야 한다.

또한, `file` 을 통해 사용될 `Dockerfile` 을 지정하여 설정할수 있다.

`AWS ECR` 저장소에 이미 존재하는 `images`  정의를 가능하게 할수 있는데, 이러한 목적으로 `uri` `property` 를 지정할수 있다.

`uri` 는 다음의 `format` 을 따른다

`<account>.dkr.ecr.<region>.amazonaws.com/<repository>@<digest>` 또는
`<account>.dkr.ecr.<region>.amazonaws.com/<repository>:<tag>` 

추가적으로,  다음의 `properties` 를 통해 `docker build` `command` 에 전달할 `arguments` 를 정의할수 있다.

- **`buildArgs`** : `buildArgs` `property` 는 `--build-arg` `flag` 와 함께 `docker build` 명령을 전달한다.<br>이는  이후 `Dockerfile` 의 `ARG` 를 통해 참조된다. (see [Documentation](https://docs.docker.com/reference/dockerfile/#arg))

- **`buildOptions`**:  `buildOptions` `property` 는 `docker build` 명령에 전달될 `options` 를 정의할수 있다. (see [Documentation](https://docs.docker.com/reference/cli/docker/buildx/build/#options))

- **`cacheFrom`** : `cacheFrom` `property` 는 캐싱 `layer` 에서 사용할 `image` 를 지정하는데 사용한다.<br>`--cache-from` `flag` 와 함께 `docker build`  명령을 전달한다. (see [Documentation](https://docs.docker.com/reference/dockerfile/#usage))

- **`platform`**: `platform` `property` 는 `architecture target` 을 지정하는데 사용된다.<br>`--platform` `flag` 와 함께 `docker build` 명령을 전달한다. (see [Documentation](https://docs.docker.com/reference/dockerfile/#from))<br><br>만약, 존재하지 않는다면 `docker` 는 기본 `computer architecture`  를 사용한다.<br>`Lambda` 는 `runtime` 설정을 명시하지 않는한, `x86` 을 일반적으로 사용한다.<br><br>`ARM` 기반으로 하는 머신 (`Apple M1 Mac`) 에서 `build` 할때 `error` 를 방지할 목적이라면,`linux/amd64` 을 사용해야 한다.<br><br>이 `flag` 에 대한 옵션은 `linux/amd64` (`x86` based Lambdas), `linux/arm64`(`arm` based Lambdas), 또는 `windows/amd64` 가 있다.

`image` 에 대한 `uri` 를 정의할때, `buildArgs`, `buildOptions`, `cacheFrom`, `platform` 을 정의 할수 없다 

```yml
service: service-name
provider:
	name: aws
	ecr:
		scanOnPush: true
		images:
			baseImage:
				path: ./path/to/context
				fild: Dockerfile.dev
				buildArgs:
					STAGE: ${opt:stage}
				cacheFrom:
					- my-image:latest
				platform: linux/amd64
		anotherimage:
			uri: 000000000.drk.ecr.ap-northeast-2.amazonaws.com/test-lambda-docker@sha256:6bb600b4d6e1d7cf521097177dd0c4e9ea373edb91984a505333be8ac9455d38
```

`functions` 에 설정시, `image`  `property`  를 통해 `images` 는 참조되어야 한다.
이는 `provider.ecr.images` 에 이미 정의된 `image` 를 가리킬수 있으며, 또는 이미 존재하는 `AWS ECR` `image` 에 즉각적으로 가리킬수 있다

다음은 `uri` 와 동일한 형식을 따르모, `image` 를 사용하는 경우 `handler` 및 `runtime` 속성이 모두 지원되지 않는다

```yml
service: service-name
provider:
	name: aws
	ecr:
		images:
			baseimage:
				path: ./path/to/context

functions:
	hello:
		uri: 0000000000.drk.ecr.ap-northeast-2.amazonaws.com/test-lambda-docker@sha256:6bb600b4d6e1d7cf521097177dd0c4e9ea373edb91984a505333be8ac9455d38
	world:
		image: baseimage
```

또한 `functions[].image` 에서 `workingDirectory`, `entryPoint`, `command` `properties` 를 통해 `image` 설정을 추가할수 있다.

`workingDirectory` 는 `string` 유형의 `path` 를 받는다
`entryPoint` 와 `command` 는 `string` 의 `list` 를 정의해야 한다.

```yml
service: service-name
provider:
	name: aws
	ecr:
		images:
			baseimage:
				path: ./path/to/context

functions:
	hello:
		uri: 0000000000.drk.ecr.ap-northeast-2.amazonaws.com/test-lambda-docker@sha256:6bb600b4d6e1d7cf521097177dd0c4e9ea373edb91984a505333be8ac9455d38
		workingdirectory: /workdir
		command:
			- executable
			- flag
		entryPoint:
			- executable
			- flag
	world:
		image: baseimage
		command:
			- executable
			- flag
		entryPoint:
			- executable
			- flag
```

처음 배포할때 `local` 에서 `build` 한 `image` 를 사용하는 경우, `Framework` 는 자동을 `ECR` 저장소를 생성하여 이러한 `image` 를 저장한다.

저장소 이름은 `serverless-<service>-<stage>` 이다.

현재 사용될수 있기 때문에, `sls remove` 명령을 실행하면 생성된 `ECR` 저장소가 제거 된다.
배포중에 `Framework` 는 필요에 따라 `ECR` 에 `Docker` 로그인을 시도한다.

이는  `local` 구성에 따라 `Docker` 인증 토큰이 암호화되지 않은채로 저장될수 있으므로, 주의해야한다.
이에 대해서 [here](https://docs.docker.com/engine/reference/commandline/login/#credentials-store) 를 확인하길 권장한다.

## Architecture 설정 명령어

`Lambda` 는 기본적으로 `64-bit` `x86` `Architecture` `CPUs` 를 사용한다.
하지만, 더 나은 가격과 퍼포먼스를 위해 `arm764 Architecture` (`AWS Gravition2 processor`)를 사용할수도 있다

`AWS Gravition2 processor` 로 번경하려면, `provider.archtecture`   를 다음처럼 설정한다.

```yml
provider:
	...
	architecture: arm64
```

`functions` `level` 에서 독립적으로 설정한다면 다음처럼 한다.

```yml
functions:
	hello:
		...
		architecture: arm64
```

## Runtime Management










