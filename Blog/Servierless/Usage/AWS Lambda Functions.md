
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

>[!info] [Understanding how Lambda manages runtime version updates](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-update.html) 에서 `Lambda` 에서  `version update` 를 어떻게 관리하는지에 대해 나와있다

`Runtime Management` 는 `lambda` 함수에서 호환성 문제가 발생하는 경우 `runtime` 을 세부적으로 제어할수 있도록 해준다. 

만약, `runtimeManagement` 를 `auto` 로 설정하길 원한다면, `runtimeManagement` 를 지정할 필요없이 `default` 로 암묵적으로 `auto`  설정된다.

만약, `function` 을 재배포할때, `runtime` 을 `update` 하길 원한다면, `onFunctionUpdate` 로 설정할수 있다. 

모든 `functions` 의 `runtime` 을 관리설정하도록, `provider.runtimeManagement` 에 다음처럼 설정할수도 있다.

```yml
provider:
	...
	runtimeManagement: onFunctionUpdate
```

>[!info] `function` 마다 개별적으로 설정도 가능하다 

마지막으로, `mode` `property` 로 `auto` , `onFunctionUpdate`  를 설정할수 있다.
이러한 경우, 다른 `variable` `source` 에 `runtimeManagement` 를 적용할때 사용된다.
이는 다음과 같다

```yml
functions:
	hello:
		...
		runtimeManagement:
			mode: manual
			arn: <aws runtime arn>
```

## SnapStart

`Lmabda`  의 `JAVA` 용 `SnapStart`  는 대기시간에 민감한 `application` 의 시작 성능을 향상시킨다.

>[!info] `Cold start` 를 말하는듯 하다.

`lambda` 함수에 `SnapStart` 를 활성화하려면 함수 구성에 `snapStart` `property`  를 추가하면 된다.
이 `property`  를 `true`  로 설정하면 이 함수에 대한 `PublishedVersions` 값이 생성된다고 말한다.

```yml
functions:
	hello:
		...
		runtime: java11
		snapStart: true
```

>[!info] `Lambda` `snapStart` 는 오직 `Java11`, `Java17`, `Java21` `runtime` 에서만 지원되며, `provisioned concurrency`, `arm64 archtecture`, `Lambda Extensions API`, `Amazon Elastic System`, `AWS X-Ray`, `512MB` 보다 큰 임시 스토리지는 지원하지 않는다.

## VPC Configuration

`function` 에 `vpc` 객체 `property` 추가해서 `serverlss.yml`는 `VPC` 설정을 추가했다.

이 객체는 `function` 을 위한 `VPC` 구축을 위해 `secrityGroupIds` 와 `subnetIds` 배열 `property` 가 필요하다.

```yml
service: service-name
provider: aws

functions:
	hello:
		handler: handler.hello
		vpc:
			securityGroupIds:
				- securityGroupId1
				- securityGroupId2
			subnetIds:
				- subnetId1
				- subnetId2
```

또는, `service` 의 모든 `functions` 에 `VPC` 설정을 적용하길 원한다면, `provider.vpc` 에 설정을 추가할수 있다. 그리고 `function` `level` 에 설정하여, `service` `level` 에서 설정한것을 덮어씌울수도 있다

```yml
service: service-name
provider:
	name: aws
	vpc:
		securityGroupIds:
			- securityGroupId1
			- securityGroupId2
		subnetIds:
			- subnetId1
			- subnetId2

functions:
	hello:
		handler: handler.hello
		# service level 에서 설정한 vpc overwrite
		vpc:
			securityGroupIds:
				- securityGroupId1
				- securityGroupId2
			subnetIds:
				- subnetId1
				- subnetId2
				
	# service level 에 설장한 vpc 적용됨
	users:
		handler: handler.users
```

`serverles deploy` 를 실행할때, `VPC` 설정은 `lambda` 와 함께 배포된다.

만약, 특정 `functions` 가  `VPC` 없이 설정되길 원한다면, `functions` 안에 `vpc` `property` 에 `~`(null) 을 설정할수 있다.

```yml
service: service-name
provider:
	name: aws
	vpc:
		securityGroupIds:
			- securityGroupId1
			- securityGroupId2
		subnetIds:
			- subnetId1
			- subnetId2

functions:
	hello:
		handler: handler.hello
		vpc: ~
	users:
		handler: handler.users
```

### VPC IAM Permissions

`Lambda` `function` `execution` `role` 은 `ENI` 를 `create`, `describe`, `delete` 권한(`permission`)을 가져야 한다 

`VPC Configuration` 을 제공할때, 기본적으로 `AWSLambdaVPCAccessExecutionRole`  이 `Labmda execution role` 과 함께 제공된다.

이  `custom role` 이 있는 경우, 적절한 `ManagedPolicyArns` 이 포함되어 제공되야 한다.

[Required IAM Permissions](https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html#configuration-vpc-permissions) 에서 보면, `Lambda` 함수는 `AWS VPC` 를 연결하려면, `Lambda` 는 `network interface` (`ENI`)를 관리하고 생성할수 있는 `permission`  이 필요하다.

이를 위해 `Lambda` 함수에 `AWS` `managed policy` 인 `AWSLambdaVPCAccesExecutionRole` 을 연결할 필요가 있다.

이는 `Lambda` 콘솔에서 새 함수를 생성하고, 이를 `VPC` 에 연결하면 `Lambda`가 자동으로 이 권한 정책을 추가한다. 

>[!note] 함수 생성시 `default` 로 `execution role` 로 제공된다는 말이다.

만약, 자체적으로 `IAM` `permission` `policy` 를 생성하려면 다음권한을 모두 추가한다.

- `ec2:CreateNetworkInterface`
- `ec2:DescribeNetworkInterfaces`: 이 액션은 `Resource: "*"` 인 경우에만 작동한다
- `ec2:DesribeSubnets`
- `ec2:DeleteNetworkInterface`:  `DeleteNetworkInterface` 를 위한 `resource ID` 가 지정되지 않는다면, 함수가 `VPC` 접근을 할수 없을수도 있다.<br><br>이는 고유한 `resource ID` 혹은 모든 `resource ID` 를 포함하도록 둘중 하나를 지정한다 <br><br>ex: `Resource: "arn:aws:ec2:us-west-2:123456789012:*/*`
- `ec2:AssignPrivateIpAddresses`
- `ec2:UnassignPrivateIpAddresses`

>[!warning] `ENI` 생성을 위해 함수의 `role` 에만 이러한 `permission` 이 필요하며, 함수 호출에는 필요하지 않는다. `VPC` 에 연결된 상태에서도 이러한 권한이 제거된 경우에도 함수 호출은 성공적으로 수행될수도 있다.

함수를 `VPC` 에 연결하려면, `Lambda` 는 `IAM` 사용자 `role` 을 사용하여 `network` `resources` 를 확인해야 한다.

다음과 같은 `IAM` `permission` 이 있는지 확인하라

- `ec2:DescribeSecurityGroups`
- `ec2:DescribeSubnets`
- `ec2:DescribeVpcs`

## Environment Variables

`serverless.yml` 에 특정 `function` 에 `environment` 객체 `property `를 추가하여 환경변수 설정을 추가할수 있다

```yml
service: service-name
provider: aws

functions:
	hello:
		hanlder: handler.hello
		environment:
			TABLE_NAME: tableName
```

또는, `provider` `level` 에 설정을 추가하여, 모든 `functions` 에 환경변수를 적용할수 있다. 
`function` `level` 에 설정한 환경변수는 `provider` `level` 의 환경변수와 `merge` 되므로, `provider` `level` 에 정의한 환경변수에 접근할수 있다.

만약, `function` `level` 과 `provider` `level` 의 `envoriment` `key` 가 중복된다면, `function` `level` 의 환경변수로 `overwrite` 된다

```yml
service: service-name
provider:
	name: aws
	environment:
		SYSTEM_NAME: mySystem
		TABLE_NAME: tableName
functions:
	hello:
		handler: handler.hello
	users:
		handler: handler.users
		environment:
			TABLE_NAME: tableName2
```

## Tags

`functions` 에 대한 `tags` 를 추가하는것이 가능하다
이러한 `tags`  는 `AWS consle` 에 나타나며 이를 통해 `tag`별 함수를 그룹화하거나 공통 `tag` 가 있는 함수를 쉽게 찾을수 있다고 한다.

```yml
functions:
	hello:
		handler: handler.hello
		tags:
			foo: bar
```

또는 `service` 안의 모든 `functions` 에 `tag` 를 적용할수 있다.
`provider` `level` 에 설정하면 되며, `function` `level`  에 적용시, `tags` 는 `merge` 되어 `provider` `level` 의 `tag` 도 같이 적용된다. 

만약, `function` `level` 의 `tags` 와 `provider` `level` 의 `tag` 가 겹친다면, `function` `level` 의 `tags` 로 `overwrite` 된다.

```yml
# serverless.yml
service: service-name
provider:
  name: aws
  tags:
    foo: bar
    baz: qux
functions:
  hello:
    # 이 함수는 `provider.tags` 를 적용한다
    handler: handler.hello
  users:
    # 이 함수는 `provider.tags` 와 `merge` 되지만
    # foo 값을 overwrite 한다
    handler: handler.users
    tags:
      foo: quux
```

## Layers

`function` 에 `layer` 설정을 할수 있다.

```yml
functions:
	hello:
		handler: handler.hello
		layers:
			- arn:aws:lambda:region:xxxxxx:layer:layerName:Y
```

`Layder` 는 `Lambda` 에서 구현한 사용자 지정 `runtime` 을 통해 `runtime: provided` 와  함께 사용될수 있다

## Log Group Resources

기본적으로, `framework` 는 `Lambda` 에 대한 `LogGroups` 를 생성한다.
이는 `service` 를 삭제할 경우 쉽게 정리할수 있으며, `Labmda IAM permission` 을 훨씬 구체적이고 안전하게 만든다.

이러한 `default` 설정을 비활성화 하고 싶다면 `disalbeLogs: true` 를 사용할수 있다.

`CloudWatch` `log` 의 보존(`retention`) 기간을 지정하고 싶다면 `logRetentionInDays` 로 설정할수 있다.

`LogGroup` 에 대한 `DataProtectionPolicy`(데이터 보호 정책) 을 지정하고 싶다면, `logDataProtectionPolicy` 를 정의할수 있다.

```yml
functions:
	hello:
		handler: handler.hello
		# log 비활성화
		disabledLogs: true
	goodBye:
		handler: handler.goodBye
		# 14 일동안 log 유지
		logRetentionInDays: 14
		# log 데이터 보호 정책
		logDataProtectionPolicy:
			Name: data-protection-policy
```

## Versioning Deployed Functions

기본적으로, `framework` 는 매번 배포할때 마다 함수 `version` 을 생성한다.
이 동작은 선택적이며, `qulifier`(한정자) 로 이전 `version` 을 호출하지 않도록 끌수도 있다

이를 수행하려면, `arn:aws:lambda:...:function/myFunc:3` 으로 함수를 호출하여 `version` `3` 을 호출할수 있다.

`version`은 `serverless` 에 의해 정리되지 않으며, 오래된 버전을 정리하기위한 `plugin` 이나 기타 도구를 사용해야 한다.

`version`  을 `framework` 에서 정리하지 않는 이유는, 호출된 함수의 `version` 이 오래되었는지 아닌지에 대한 정보가 없기 때문이다.

함수 `version` 은 참조하는 함수와 별도의 `resource` 이기 때문에 이 기능을 사용하면 총 `stack` 출력 및 `resource` 수를 증가시킨다.

>[!warning] 위의 글은 번역체로, 명확하게 이해는 가지 않는다...

다음은 `versionFunctions` 옵션을 `provider` `level` 에 설정하여, `versioning` 을 `off` 시킨다.

```
provider:
	versionFunctions: false
```

## Dead Letter Queue (DLQ)

`AWS` `lambda` 함수가 실패하면, 재실행한다.
만약, 재실행역시 실패하면, `AWS` 는 실패 `request`에 대한 정보를 보내는 `SNS topick` 또는 `SQS queue` 기능이 있다.

이러한 기능을 [Dead Letter Queue](https://docs.aws.amazon.com/ko_kr/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html) 라 부르며, `Lambda` 실패를 추적, 진단 대응하는데 사용할수 있다.

이러한 `Dead Letter Queue` 를 `SNS topic` 과 `onError` 설정 `parameter` 와 함께 `serverless` 함수에 설정할수 있다.

>[!warning] `function` 당 하나의 `onError`  제공해야만 한다.

### DLQ with SNS

`SNS topic` 은 미리 생성되어야 하며, `function` `level`  에 `arn` 으로 제공되야 한다.

```yml
service: service
provider:
  name: aws
  runtime: nodejs14.x
functions:
  hello:
    handler: handler.hello
    onError: arn:aws:sns:us-east-1:XXXXXX:test # Ref, Fn::GetAtt and Fn::ImportValue are supported as well
```

### DLQ with SQS

`DLQ` 를 통해 `SQS queue` 와 `SNS topic` 둘 다 제공한다.
하지만 `onError` 설정은 현재 오직 `SNS topic` `arn` 만 지원하며, `SQS queue` `arn` 은`IAM role` 을 업데이트 할때 발생하는 `race condition` 때문에  `onError` 설정을 지원하지 않는다.

## KMS Keys

`Lambda` 는 `KMS`(`AWS Key Management Service`) 로 미사용 환경 변수를 암호화한다.
`kmsKeyArn` 설정 변수를 활성화하여, 암호화된 `KMS key` 를 정의할수 있다

```yml
service:
	name: service-name

provider:
	name: aws
	kmsKeyArn: arn:aws:kms:us-east-1:xxxxxx:key/some-hash
	environment:
		TABLE_NAME: tableName1

functions:
	hello:
		handler: handler.hello
		kmsKeyArn: arn:aws:kms:us-east-1:xxxxxx:key/some-hash
		enviroment:
			TABLE_NAME: tableName2
	goobdye:
		handler: handler.goodbye
```

## AWS X-Ray Tracing

옵셔널한 `property` 인 `tracing` 을 설정하여 `AWS-X-Ray Tracing` 을 활성화 할수 있다.

```yml
service: myService
provider:
  name: aws
  runtime: nodejs14.x
  tracing:
    lambda: true
```

또한 `function` `level` 에 적용하여, `provider` `level` 의 `tracing` 을 `override` 할수 있다.

```
functions:
  hello:
    handler: handler.hello
    tracing: Active
  goodbye:
    handler: handler.goodbye
    tracing: PassThrough
```

## Asynchronous invocation

함수를 비동기적으로 호출하려는 경우 다음과 같은 추가 설정을 구성할수 있다.

### destinations

`destination`  은 `service` 또는 기타 `qualified` 와 함께 배포하는 다른 `lambda` 함수일수 있다
``
>[!info] 외부에서 관리되는 `lambda`, `EventBridge event bus`, `SQS` ,`SNS topic` 같은 ...

이는 `ARN` 또는 참조를 통해 설정할수 있다.

```yml
functions:
  asyncHello:
    handler: handler.asyncHello
    destinations:
	  # 성공시 ohterFunctionInService 로 
      onSuccess: otherFunctionInService
	  # 실패시 arn 에 명시된 sns topic 으로
      onFailure: arn:aws:sns:us-east-1:xxxx:some-topic-name
  asyncGoodBye:
    handler: handler.asyncGoodBye
    destinations:
	  # 실패시, 
      onFailure:
		# CloudFormation 내장 함수를 사용하는 경우 사용하는 arn 
		# 이렇게 CF 를 통한 arn 으로 `target`의 실행 권한을 정확하게 보장하려면, 
		# `type` 에 `sns`, `sqs`, `eventBus`, `function` 을 지정해야 한다.
        type: sns
        arn:
          Ref: SomeTopicName
```

### Maximum Event Age and Maximum Retry Attempts 

- `maximumEventAge` 는 최대 `Event` 를 처리할수 있는 최대 한계를 정의,<br>이벤트가 `labmda` 함수로 전달된 이후 설정한 값에 설정한 시간안에 이벤트를 처리하지 않으면, 이벤트를 더이상 처리되지 않는다.<br><br>`60` 에서 `6 hour` 시간 값(`seconds`) 를 허용한다

- `maximumRetryAttempts` 는 최대 재실행 시도값으로 `0` 에서 `2` 까지의 값을 허용한다.

```yml
functions:
  asyncHello:
    handler: handler.asyncHello
    maximumEventAge: 7200
    maximumRetryAttempts: 1
```

## EFS Configuration

`Lambda` 에서 `fileSystemConfig`  를 통해 `EFS` (`Elastic File System`) 을 사용할수 있다.
`fileSystemConfig` 는 `arn` 과 `localMountPath` `property` 를 포함하는 객체이다.

- `arn` : `EFS` 접근을 가리키는 참조 주소를 지정
- `localMountPath`: 마운트된 `file system` 의 절대 경로를 지정

```yml
# serverless.yml
service: service-name
provider: aws
functions:
  hello:
    handler: handler.hello
    fileSystemConfig:
      localMountPath: /mnt/example
      arn: arn:aws:elasticfilesystem:us-east-1:111111111111:access-point/fsap-0d0d0d0d0d0d0d0d0
    vpc:
      securityGroupIds:
        - securityGroupId1
      subnetIds:
        - subnetId1
```

## Ephemeral Storage (임시 스토리지)

기본적으로 `Labmda` 는 `/tmp` 디렉토리에  `512 MB` 의 임시 스토리지를 할당한다.
하지만, `emphmeralStorageSize`  를 통해 용량을 증가시킬수 있다.

>[!info] 증가 값은 `MB` 이며, `512` 에서 `10240` 까지 가능하다.

```yml
functions:
  helloEphemeral:
    handler: handler.handler
    ephemeralStorageSize: 1024
```

