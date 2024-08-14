`API Gateway` 를 사용하면 `HTTP APIs`  로 배포할수 있다.
이는 총 $2$ 가지 `version` 이 존재한다.

- v1 은 [[REST APIs (API Gateway v1)]] 라 부른다
- v2 는 [[HTTP API (API Gateway v2)]] 라 부르며, 이는 [[REST APIs (API Gateway v1)]] 보다 저렴하고 빠르다.

이 서비스는, `v2` 를 설명한다.

---

`AWS Lambda` 에 대한 `Eevent`소스로  `HTTP endpoint`  를 생성하려면, `Servelress Framework` 의 간편한 `API Gateway` `Event` 를 사용하면 된다.

`AWS Lambda` 함수와 함께 `HTTP endpoint` 로 `integrate` 하기 위해 총 $5$ 가지 `method` 로 설정할수 있다.
> [!info] `통합하다` 로 해석하는데, `통합` 이 뭔가 잘 안들어온다. `integrate` 로 해석한다.

###  `lambda-proxy` / `aws-proxy` / `aws_proxy` (`Recommended`)

이 `Framwork` 는 사용자에 의해 다른 메서드들을 제공하지 않는한, `lambda-proxy` 메서드를 `default` 로 사용한다. 

`lambda-proxy` 는 자동적으로  `HTTP Request` 의 `content` 를 `AWS Lambda` 함수로 전달한다.
그리고, `AWS Lambda` 함수의 `code` 안에서 `response` 설정이 가능하도록 허용한다.

>[!info]  `lambda-proxy` `aws-proxy` `aws_proxy` 의 차이
>`AWS Lambda` 함수와 `API Gateway` 를 통합할 유형을 지정하는데 사용하는 용어이다.
>이는 용어를 정리하는 과정에서 파생된 동의어이다.
>
>처음에는 `lambda-proxy` 라는 용어를 사용했지만, 이후, `aws-proxy` 라는 이름으로 변경했다.
>이러한 이유로 인해, 여러 용어가 동의어로써 사용되며, `Serverless Framework` 에서는 보통 `lambda-proxy` 로 사용되는 듯하다.
>
>`aws_proxy` 는 언더스코어를 사용하는 명명 규칙과의 호환성을 의해 추가된것이라고 한다.

### `lambda` / `aws`

`lambda` 혹은 `aws` 메서드를 사용하면, 각 `API Gateway` 에서 명시적으로 `headers`, `status codes` 와 더불어 더 많은 설정을 해야한다.

이러한 `lambda` 메서드는 매우 지루한 방법이 될수 있으므로, `lambda-proxy` 를 사용할것을 권장한다.

### `http`

`HTTP Backend` 와 `integrating` 하기 위해 사용된다

- `http-proxy` / `http_proxy`

`HTTP proxy` `integration` 과 함께 `integrating` 하기 위해 사용된다.

### `mock`

실제 `back end` 호출없이 `testing` 하기 위해 사용된다.

## Lambda Proxy Integration

간단한, `HTTP Endpoint` 를 만드는 방법이다.

```yml
functions:
	index:
		handler: handler.hello
		events:
			- http: GET hello
```

>[!info] handler.js
```js
'use strict'

module.exports.hello = function (event, context, collback) {
	console.log(event); // incoming request data 를 담은 event
						// query params, headers and more

	const res = {
		statusCode: 200,
		headers: {
			'x-custom-haeder': "My Header Value"
		},
		body: JSON.stringify({ message: 'Hello Wolrd' })
	};

	callback(null, res);
};
```

### Lambda-proxy event 객체 예시

```json
{
  "resource": "/",
  "path": "/",
  "httpMethod": "POST",
  "headers": {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
    "Accept-Encoding": "gzip, deflate, br",
    "Accept-Language": "en-GB,en-US;q=0.8,en;q=0.6,zh-CN;q=0.4",
    "cache-control": "max-age=0",
    "CloudFront-Forwarded-Proto": "https",
    "CloudFront-Is-Desktop-Viewer": "true",
    "CloudFront-Is-Mobile-Viewer": "false",
    "CloudFront-Is-SmartTV-Viewer": "false",
    "CloudFront-Is-Tablet-Viewer": "false",
    "CloudFront-Viewer-Country": "GB",
    "content-type": "application/x-www-form-urlencoded",
    "Host": "j3ap25j034.execute-api.eu-west-2.amazonaws.com",
    "origin": "https://j3ap25j034.execute-api.eu-west-2.amazonaws.com",
    "Referer": "https://j3ap25j034.execute-api.eu-west-2.amazonaws.com/dev/",
    "upgrade-insecure-requests": "1",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36",
    "Via": "2.0 a3650115c5e21e2b5d133ce84464bea3.cloudfront.net (CloudFront)",
    "X-Amz-Cf-Id": "0nDeiXnReyHYCkv8cc150MWCFCLFPbJoTs1mexDuKe2WJwK5ANgv2A==",
    "X-Amzn-Trace-Id": "Root=1-597079de-75fec8453f6fd4812414a4cd",
    "X-Forwarded-For": "50.129.117.14, 50.112.234.94",
    "X-Forwarded-Port": "443",
    "X-Forwarded-Proto": "https"
  },
  "queryStringParameters": null,
  "pathParameters": null,
  "stageVariables": null,
  "requestContext": {
    "path": "/dev/",
    "accountId": "125002137610",
    "resourceId": "qdolsr1yhk",
    "stage": "dev",
    "requestId": "0f2431a2-6d2f-11e7-b799-5152aa497861",
    "identity": {
      "cognitoIdentityPoolId": null,
      "accountId": null,
      "cognitoIdentityId": null,
      "caller": null,
      "apiKey": "",
      "sourceIp": "50.129.117.14",
      "accessKey": null,
      "cognitoAuthenticationType": null,
      "cognitoAuthenticationProvider": null,
      "userArn": null,
      "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36",
      "user": null
    },
    "resourcePath": "/",
    "httpMethod": "POST",
    "apiId": "j3azlsj0c4"
  },
  "body": "postcode=LS17FR",
  "isBase64Encoded": false
}
```

## HTTP Endpoint 에 대한 확장 옵션

`POST` `endpoint` 에 `posts/create` 경로를 정의한다

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
```

### CORS 활성화

`HTTP` `endpoint` 에 `CORS` 설정을 활성화 한다. 
이는 다음의 `event` 설정에서 간단하게 수정할수 있다.

```yml
functions:
	create:
		handler: handler.hello
		events:
			- http:
				path: hello
				method: get
				cors: true
```

`cors: true` 는 다음의 구문과 같다 

```yml
functions:
	create:
		handler: handler.hello
		events:
			- http:
				path: hello
				method: get
				cors:
					origin: '*' # 모든 origin 허용
					headers:
						- Content-Type # 요청 본문의 미디어 타입
						- X-Amz-Date # AWS 요청에 대한 날짜와 시간
						- Authorization # 인증 토큰 정보
						- X-Api-Key # API 키를 전달하는데 사용
						- X-Amz-Security-Token # AWS 임시 보안 자격 증명에 사용
						- X-Amz-User-Agent # AWS SDK 나 클라이언트 정보를 전달
						- X-Amzn-Trace-Id # AWS 에서 요청을 추적하는데 사용
					allowCredentials: false # CORS 에 쿠키, HTTP 인증, 
											# 클라이언트 측 SSL 인정서 포함여부
```

또한, 여러 `origin` 정의를 허용한다.
이때. `origins` 를 사용하여 배열로 설정해주어야 한다.

```yml
functions:
	create:
		handler: handler.hello
		events:
			- http:
				path: hello
				method: get
				cors:
					origins: # 여러 origin 설정
						- http://example.com
						- http://example2.com
					headers:
						- Content-Type # 요청 본문의 미디어 타입
						- X-Amz-Date # AWS 요청에 대한 날짜와 시간
						- Authorization # 인증 토큰 정보
						- X-Api-Key # API 키를 전달하는데 사용
						- X-Amz-Security-Token # AWS 임시 보안 자격 증명에 사용
						- X-Amz-User-Agent # AWS SDK 나 클라이언트 정보를 전달
						- X-Amzn-Trace-Id # AWS 에서 요청을 추적하는데 사용
					allowCredentials: false # CORS 에 쿠키, HTTP 인증, 
											# 클라이언트 측 SSL 인증서 포함여부
```

또한, 이러한 `origin` 을 작성할때 `waildcard` 를 허용한다.
다음은 `example.com` 의 모든  `subdomain` 을 가진 `domain` 을 허용한다..

```yml
cors:
	origins:
		- http://*.example.com
		- http://example2.com
```

>[!warning] `origins` 에 여러 `Domain` 작성시, `Access-Control-Allow-Origin` `header` 에 여러 값을 보낼수 없다. 

이렇게 `Access-Control-Origin` 헤더에 여러값을 보낼수 없는 이유는, `browser` 의 보안 정책때문이다.

이러한 부분을 해결하기 위해서는, `response` `template` 를 사용해야 한다.
이러한 `template` 는 다음과 같은 작업을 수행한다.

- 요청 출처가 `origin` 인지 확인한다.
- 이 출처가 허용된 `origins` 목록과 일치하는지 확인한다.
- 일치하는경우, `Access-Control-Allow-Origin` `header` 를 해당 출처로 동적으로 설정한다.

여기에 사용된 `response` `template` 는 다음과 같다. 

```sh
#set($origin = $input.params("Origin")
#if($origin == "http://example.com" || $origin == "http://*.amazonaws.com") #set($context.responseOverride.header.Access-Control-Allow-Origin = $origin) #end
```

`cors` 속성을 설정하면, `CORS` `preflight` `response` 에서 `Access-Control-Allow-Origin`, `Access-Control-Allow-Headers`, `Access-Control-Allow-Methods` , `Access-Control-Allow-Credentials` `Header` 가 설정된다.

`Lambda` `Integration` 을 사용하는 경우, `Access-Controll-Allow-Origin` 및 `Access-Control-Allow-Credentials` `header` 도 `method` 및 `integration response` 에 모두 제공된다.

여기서 주의할점은 `Access-Control-Allow-Credentials` `header` 는 명시적으로 `true` 로 설정하지 않는한, 제외되어 처리된다.

다음은, `preflight` `response` 에 `Access-Control-Max-Age` 를 활성화 한다.
이를 위해서는 `cors` 안에 `maxAge`  `property` 를 작성한다.

```yml
functions:
	hello:
		handler: handler.hello
		events:
			- http:
				path: hello
				method: get
				cors:
					origin: '*'
					maxAge: 86400
```

`API Gateway` 에  `CloudFront` 나 다른 `CDN` 을 사용한다면, 추가 `hop` 을 피하기 위해 `OPTIONS` 요청을 `cache` 할수 있도록  `Cache-Control` `header` 설정을 할수도 있다,

`preflight` `response` 에 `Cache-Control` `header` 를 활성화하는 `property` 가 `cors.cacheControl` 이다.

```yml
functions:
  hello:
    handler: handler.hello
    events:
      - http:
          path: hello
          method: get
          cors:
            origin: '*'
            headers:
              - Content-Type
              - X-Amz-Date
              - Authorization
              - X-Api-Key
              - X-Amz-Security-Token
              - X-Amz-User-Agent
              - X-Amzn-Trace-Id
            allowCredentials: false
			# 브라우저와 프록시에 10분동안 캐시, 프록시가 오래된 컨텐츠를 제공하는것을
			# 허용하지 않는다.
            cacheControl: 'max-age=600, s-maxage=600, proxy-revalidate'
```

`CORS` `header` 는 단일한 값도 허용한다.

```yml
functions:
	hello:
		handler: handler.hello
		events:
			- http:
				path: hello
				method: get
				cors:
					headers: '*'
```

만약, `lambda-proxy` 와 함께 `CORS` 를 사용하길 원한다면, `headers` 객체안에 `Access-Control-Allow-*` 를 포함해야 한다. 이는 다음과 같다.

```js
'use strict'

module.exports.hello = function(event, context, callback) {
	const res = {
		statusCode: 200,
		headers: {
			'Access-Control-Allow-Origin': '*',
			'Access-Control-Allow-Credentials': true,
		},
		body: JSON.stringify({ message: 'Hello Wolrd!' })
	};

	callback(null, response);
};
```

### HTTP Endpoint with `AWS_IAM` Authorizers

`caller` 가 `Lambda` 함수 호출을에 대한 인증을 받기 위해 `IAM` 사용자의 `access key` 를 제출하도록 요구하려면, 다음처럼 `Authorizer` 에 `AWS_IAM` 으로 설정해야 한다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				authorizer: aws_iam
```

이는  간편한 작성법 이며, 다음과 같은 내용이다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				authorizer: 
					type: aws_iam
```

### HTTP Endopoint with custom Authorizers

`Custom Authorizer` 는 대상 `AWS Lambda` 함수 이전에, 실행되는 `AWS Lambda` 함수이다.
이는 매우 `Microservice Architecture` 또는 `businiess logic` 실행전에 간단한 인증을 원할때 유용하게 사용된다.

다음과 같이 `http event` 의 `Authorizer` 를 동일한 `service` 의 다른 함수로 설정하여, `HTTP Endpoint` 에 대한 `Custom Authorizer` 를 활성화 할수 있다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				authorizer: authorizerFunc
		authorizerFunc:
			handler: handler.authorizerFunc
```

또는, 만약 더 많은 `options` 와 함께 `Authorizer` 를 설정하길 원한다면, `authorizer` `property` 를 객체로 전환할수 있다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				authorizer: 
					name: authorizerFunc
					resultTtlInSeconds: 0
					identitySource: method.request.header.Authorization
					identityValidationExpression: someRegex
					type: token
		authorizerFunc:
			handler: handler.authorizerFunc
```

`Authorizer` 함수가 `service` 에 존재하지 않고, `AWS` 에 존재한다면, 함수이름 대신 `Lambda` 함수 `ARN`  을 전달할수 있다. 

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				authorizer: xxx:xxx:Lambda-Name
```

이렇게 `ARN` 으로 전달한 `authorizer` 에 더많은 `Authorizer` 옵션을 설정하길 원한다면, `authorizer` `property` 를 사용하여 전환할수 있다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				authorizer: 
					arn: xxx:xxx:Lambda-Name
					managedExternally: false
					resultInSecond: 0
					identitySource: method.request.heaser.Authorization
					identityValidationExpression: someRegx
```

만약 `Authorizer` `function` 에 대한 `permission` 이 외부에서 관리되는 경우, `ManagedExternally: true` 를 설정하여 함수에 대한 `permission` 생성을 건너뛸수 있다.

```yml
functions:
  create:
    handler: posts.create
    events:
      - http:
          path: posts/create
          method: post
          authorizer:
            arn: xxx:xxx:Lambda-Name
            managedExternally: true
```

>[!warning] `API Gateway` 에 의해 호출할수 있도록 하는 `permission` 이 허용된 `authorizer` 함수는 반드시 `stacke` 을 배포하기전에 존재해야 한다. 그렇지 않으면 배포는 실패한다.

`type` `property` 를 설정하면 `Request` 유형 `Authorizer` 를 사용할수 있다.
이 경우, `identitySource` 는 정책 `cache` 에 대한 여러 항목이 포함될수 있다.
`type` `property` 의 `default` 는 `token` 이다.

```yml
functions:
  create:
    handler: posts.create
    events:
      - http:
          path: posts/create
          method: post
          authorizer:
            arn: xxx:xxx:Lambda-Name
			resultTtlInSeconds: 0
			identitySource: method.request.header.Auhtorization, context.identity.sourceIp
			identityValidationExpression: someRegex
			type: request
```

존재하는 `Cognito User Pool` 의 `authorizer` 로 설정할수도 있다.
이는 선택적 `access token` 에 허용 `scopes` 가 포함된 다음의 예시를 볼수있다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				authorizer:
					arn: arn:aws:cognito-idp:ap-northeastp2:xxx:userpool/ap-norteast-2_ZZZ
					scopes:
						- my-app/read
```

만약, 기본적인 `lambda-proxy` `integration` 을 사용한다면, `attributes` 는 `event.requestContext.authorizer.claims` 에 노출된다.

`attributes` 로 드러나게된 `claims` 를 더 많이 제어하길 원한다면, `integration: lambda` 와 다음의 설정을 추가해야 한다. 

이러한 `claim` 들은 `events.cognitoPoolCalims` 를 통해 노출된다.

```yml
sources
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				integration: lambda
				authorizer:
					arn: arn:aws:cognito-idp:ap-northeastp2:xxx:userpool/ap-norteast-2_ZZZ
				claims:
					- email
					- nickname
```

동일한 `template` 의 `resources` 섹션 안에 `CognitoUserPool` 을 생성한다면, `CloudFormation` 의 `Fn::GetAtt` 속성을 사용하여 `ARN` 참조할수있다.

이렇게 하려면, `authorizer` 에 이름을 부여하고, `COGNITO_USER_POOLS` 유형을 지정한다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				integration: lambda
				authorizer:
					name: MyAuhtorizer
					type: COGNITO_USER_POOLS # authorizer 타입
					arn: # resource 에서 생성한, CognitoUserPool 의 Arn 을 get
						Fn::GetAtt:
							- CognitoUserPool
							- Arn
---
resources:
	Resources:
		CognitoUserPool:
			type: 'AWS::Cognito::UserPool'
			properties: ...
```

### HTTP Endpoint with `operationId`

`endpoint` 메서드와 함께 이름을 제공하길 원한다면 `operationId` 를 포함할수 있다.
이렇게 하면 `AWS::ApiGateway::Method` 내부에 `OperationsName` 이 설정된다.

```yml
functions:
	create:
		handler: users.create
		events:
			- http:
				path: users/create
				method: post
				operationId: createUser
```

>[!info] AWS::ApiGateway::Method
>`AWS CloudFormation` 에서 사용되는 리소스 유형이다.
>이는 `Amazion API Gateway` 에서 `API` 메서드를 정의하는데 사용된다.
>
>여기에 `OperationName` 이 사용되는데, 이는 `AWS::ApiGateway::Method` 의 속성중 하나이다.
>이는 `API` 문서화시 유용하게 사용될수 있도록 `API` 작업에 대한 사용자 정의이름을 지정한다
```yml
MyApiMethod:
  Type: AWS::ApiGateway::Method
  Properties:
    RestApiId: !Ref MyRestApi
    ResourceId: !Ref MyApiResource
    HttpMethod: GET
    AuthorizationType: NONE
    Integration:
      Type: AWS_PROXY
      IntegrationHttpMethod: POST
      Uri: !Sub arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${MyLambdaFunction.Arn}/invocations
    OperationName: GetUserData
```

이는 `AWS::ApiGateway::Method` 에 대한 예시이다.
이 `method` 는 `GET` 이며, `OperationName` 은 `GetUserData` 인것을 볼 수 있다.

### Using aynchronous integration

`event` 호출방식으로 `integration` 할때,  `async: true` 을 사용하면, 비동기 호출로 처리된다.
이렇게 하면, `labmda` 가 실행되는 동안에도 `API Gateway` 는 $200$ 상태코드와 함께 즉시 반환된다.

>[!info] `Labmda` 가 실행되는 동안이라는 말은 `background` 에서 실행되며, `API Gateway `는 응답을 받는 즉시 `200` 상태코드를 보낸다는 말이다.<br><br>이는 `javascript` 의 비동기처리와 연결되어 있으며, 비동기 처리되는 함수는 `callstack`  전부 비워진후, `event loop` 에 의해 처리 완료된 `task queue` 에서 함수를 `callsatck` 으로 넘겨 실행하기 때문이다.<br><br>이는 `API Gateway` 가 바로 응답을 보내도록 하는 합리적인 이유이다.

반면, 이를 명시하지 않으면 기본적으로 `AWS` `integration` 유형으로 사용된다.
`default` 값으로 `false` 이므로, `Labmda` 함수 실행시 `sync` 적으로 처리된다.

>[!warning] 여기서 혼란스러운 부분이 있다. 내가 알기로는 [[#Lambda Proxy Integration]] 부분에서 볼수 있듯이, `default` 로 `lambda-proxy` 를 사용한다고 했다.<br><br>그런데, 명시하지 않으면 `aws` 로 사용한다고?<br><br>뭔가 앞뒤가 안맞는다..<br><br>일단 여기서 말하고자 하는 핵심은, `integration` 은 `async` 로 처리가능하며, 그렇지 않은경우 `default` 로 `sync` 로 처리된다는것이다.

### Lambda 함수의 예외 처리

`Lambda` 함수안에서 예외가 `throw` 될 경우, `AWS` 는 `process existed before complateing request` 라는 `error` `message` 를 보낸다.

이는 $500$ HTTP 코드에 대한 정규 표현식으로 포착되며, 이로인해 $500$ 상태코드가 반환된다.

### setting API keys for your Rest API

`serverless.yml` 안의 `provieder.apiGateway` 에 `apiKeys`  배열 `property` 를 추가함으로써 `Rest API` `service` 에 사용될 `API Keys` 리스트를 지정할수 있다.

또한, `private` 로 설정하려는 `http` `event` 객체에 `private` `boolean` `property` 을 추가하여 `private` `endpoint` 를 명시적으로 지정하고 `API keys` 중 하나가 요청에 포함되도록 요구해야 한다.

`API keys` 는 전역적으로 생성되므로, `service` 를 여러 `stage` 에 배포하려면 `API key` 정의에 `stage` `variable`  을 포함해야 한다.

>[!info] 여기에서 여러 `stage` 에 배포하려면 `API Key` 에 정의에 `stage` `variable` 을 포함해야 한다. 라는 말이 애매할수 있다.<br><br>이 말은 `API` 키 이름이나 설명에 `stage` 이름을 포함시킬수 있으므로, 



