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

>[!info] 여기에서 여러 `stage` 에 배포하려면 `API Key` 에 정의에 `stage` `variable` 을 포함해야 한다. 라는 말이 애매할수 있다.<br><br>`API` 키 이름이나 설명에 `stage` 이름을 포함시킬수 있으므로, `API` 키에 `stage` `variable` 을 포함시키면, 각 `stage` 에 대한 고유한 `key` 가 생성된다.<br><br>이를 통해 `stage` 별로 `API` 사용을 분리하고 관리가능하다.<br> (`API Keys` 는 전역적이므로, `stage` 별로 분리되서 사용할수 없다)<br><br>이렇게 사용하기 위해 `stage` `variable` 을 포함하라고 말하는것이다.<br><br>예: `MyAPI-${stageVariables.stageName}-ApiKey`

`API key` 를 사용할때 `usagePlan` 객체를 통해 선택적으로 사용 계획 할당량(`usage plan quota`) 와 제한 (`throttle`) 을 정의할수 있다.

추가적으로, 선택된 `API key` 를 `enabled` `property` 를 `false` 로 하여 `disabled` 시킬수 있다.
값을 설정할때, 값을 변경하면 교체가 필요하다는 점에 유의해야 하며, `CloudeFormation`  은 같은 이름을 가진 $2$ 개의 `API keys` 를 허용하지 않는다.

이는 `value`  을 변경할때 이름도 변경해야 한다는 의미이다.

만약, `key` 의 이름에 관심이 없다면, 값만 설정하고, `CloudFormation` 에서 `key` 이름을 지정하도록 하는것이 좋다.

#### API key 생성방식

`Servieless framework` 에서 `API Gateway` 의 `apiKeys` 를 설정할때 다음과 같은 사항들이 적용된다.

1. **명시적으로 값만 지정한 경우**

```yml
- myFirstKey
```

이 경우, `myFirstKey` 라는 이름의 `API key` 가 생성된다. `API key` 의 값은 자동으로 생성되며, `myFirstKey` 라는 이름으로 식별된다 

2. **이름과 같이 모두 지정된 경우**

```yml
- name: mySecondKey
  value: mySecondKeyValue
```

이 경우, `mySecondKey` 라는 이름과 `mySecondKeyValue` 라는 값을 가진 `API key` 가 생성된다. 이름과 같이 모두 명시적으로 제공된다.

3. **복수의 `API key` 이름과 `value` 를 지정한 경우**

```yml
- myFirstKey
- ${opt:stage}-myFirstkey
```

이 경우, 두개의 `API key` 가 생성된다.

4. **변수와 환경 변수 사용**

```yml
- ${env:MY_API_KEY}
```

이 경우, `MY_API_KEY` 환경 변수의 값을 `API` `key` 로 사용하게 된다.
예를 들어, 환경 변수 `MY_API_KEY` 가 `customKey` 로 설정되어 있으면, `customKey` 라는 이름의 `API key` 가 생성된다.

>[!info] `API key` 이름은 실제 `value` 가 아니며 `API key` 를 식별하는데 사용되는 `label` 이다.<br>이는 배포전 식별하기 위한 `APi key` 이름만 정의된것일 뿐이다.

```yml
service: my-service
provider:
	name: aws
	apiGateway:
		apiKeys:
			# myFirstKey 생성
			- myFirstKey
			
			# stage 에 따른 api key 생성
			- ${opt:stage}-myFirstKey
			
			# env 의 MY_API_KEY 의 value 로 key 생성
			# env 를 통해 serverless variable 에 숨길수 있다.
			- ${env:MY_API_KEY}
			
			# myThirdKey 이름을 가진 api key 생성
			- name: myThirdKey
			  # myThirdKey 의 값
			  value: myThirdKeyValue
			  
			# cloudformation 에 키 이름 지정 (api key 값 설정시 권장)
			- value: myFourthKeyValue
			  description: api key description # optional
			  customerId: A string that will be set as the customerId for the key # optional
			  
		usagePlan:
			quota:
				limit: 5000
				offset: 2
				period: MONTH
			throttle:
				burstLimit: 200
				rateLimit: 100

functions:
	hello:
		events:
			- http:
				path: user/create
				method: get
				private: true
```

>[!info] `service` 배포한 후에는 `API keys` 에 대한 실제 `value` 가 `AWS` 에서 자동으로 생성된다<br> `value` 는 실제로 `API` 를 호출할때 사용되는 암호화된 문자열이다.<br>이러한 `API` 의 실제값이 생성된후 화면에 출력된다.<br><br>`--conceal` 배포 옵션을 사용하면 출력에서 값을 숨길수 있다.<br>이는 보안상의 이유로 `API key` `value` 를 로그나 화면에 표시하지 않는 경우에 유용하다.
이는 통해 `private` `property` 가 `true` 로 설정된 함수에만 필요하다
`Rest API` 에 연결하는 `client` 는 `request` 의 `x-api-key` `header`  에 `API key` 값을 설정해야 한다. 

이는 당연하게도, `private` `property` 가 `true` 인 함수에서만 이러한 `header` 설정이 필요하다.

>[!info] `private` 는 `API` 를 `API key` 만을 가진 `client` 만 접근가능하도록 만드는 설정값이다.<br>[[#setting API keys for your Rest API]] 에서 `private` `property` 를 `boolean` 값으로 설정하는 내용이 있다.

`API` 를 위한, 여러 `usage plan`  을 설정할수도 있다.
이 경우, `provider.apiGateway.usagePlan` 에서 `API key` 에 대한 `usagePlan` 을 `map` 형식으로 구성가능하다.

```yml
service: my-service
provider:
  name: aws
  apiGateway:
    apiKeys:
      - free: # myFreeKey 와 ${opt:stage}-myFreeKey 를 가진 free 이름 식별자
          - myFreeKey
          - ${opt:stage}-myFreeKey
      - paid: # myPaidKey 와 ${opt:stage}-myPaidKey 를 가진 paid 이름 식별자
	      - myPaidKey
          - ${opt:stage}-myPaidKey
    usagePlan: # usagePlan 사용
      - free: # free apiKeys 식별자에 대한 설정
          quota:
            limit: 5000
            offset: 2
            period: MONTH
          throttle:
            burstLimit: 200
            rateLimit: 100
      - paid: # paid apiKeys 식별자에 대한 설정
          quota:
            limit: 50000
            offset: 1
            period: MONTH
          throttle:
            burstLimit: 2000
            rateLimit: 1000
functions:
  hello:
    events:
      - http:
          path: user/create
          method: get
          private: true # apiKey 사용
						# clietn request 에서 x-api-key header 에 해당하는
						# api-key 를 가져야 접근 가능하다
```

### Configuring endpoint types

`API Endopint` 는 여러개의 타입이 존재한다.

1. **Edge-optimized endpoint** <br>`CloudFront` 를 활용하여 전 세계적으로 `caching` 하고 `deploy` 한다.<br>`global` 사용자에게 최적화된 성능을 제공한다

2. **Regional endpoint** <br>`region` 내에서만 `API` 를 사용하려는 경우 적합하다.<br>`local region` 내에서 처리되며, `global` `network` 를 활용하지 않는다.

3. **Private endpoint**<br>`VPC` 내에서만 접근 할수 잇는 `API endpoint` 이다.<br>`VPC` 와 관련된 `resource` 에 대한 `access` 를 제한할때 사용한다.

>[!info] `REST API` 에서 `provider.endpointType` 의 `default` 값은 `EDGE` `endpoint` 설정을 사용한다. 

만약, `REGIONAL` 혹은 `PRIVATE` 를 설정하고 싶다면, `provider.endpointType` 을 지정하여 설정할수 있다

```yml
service: my-service
provider:
	name: aws
	endpointType: REGIONAL # regional endpoint 설정
functions:
	hello:
		events:
			- http:
				path: user/create
				method: get
```

만약, `API Gateway` 에서 `VPC` `endpoint` 와 연관지어 제공하고 싶다면, `private endpoint` 설정을 사용하여 `REST API` 에서 사용할수 있다.

`VPC endpoint`  를 사용하면, `VPC` 내의 `resource` 가 `API Gateway` 의 `PRIVATE API` 를 호출할수 있다.

이 `endpoint` 는 `API Gateway REST API`  에 대해 다음과 같은 형식의 `Route 53 alias` 를 생성한다.

```sh
https://<rest_api_id>-<vpc_endpoint_id>.execute-api.<aws_region>.amazonaws.com
```

- `<rest_api_id>` 는 `API Gateway REST API` 의 `ID` 이다
- `<vpc_endpoint_id>` 는 `VPC endpoint` 의 `ID` 이다
- `<aws_region>` 은 `API Gateway` 가 배포된 `region` 이다

>[!info] `API Gateway` 가 `PRIVATE` `endpoint` 를 사용할때, `AWS Route 53` 이 자동으로 `API endpoint` 에 대한 `alias` `record` 를 생성한다.<br><br>이 `alias` `record` 는 `VPC` 내의 `resource` 가 `API`를 쉽게 호출할수 있도록 도와준다.

이 설정에 대한 예시이다.

```yml
service: my-service
provider:
	name: aws
	# API Gateway `Private endpoint` 로 구성
	endpointType: PRIVATE
	
	# 이는 `API Gateway` 가 접근할수 있는 `VPC endpoint` 의 `ID` 목록
	vpcEndpointIds:
		- vpce-123
		- vpce-456
```

### Request Parameters

선택 및 필수 `parameters` 를 함수에 전달할수 있다.
`API Gateway` 및 `SDK` 생성에 사용할 수 있도록 `true` 로 표시하면 필수 매개변수가 되며, `false` 로 표시하면 선택적 매개변수가 된다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				requrest:
					parameters:	
						querystrings:
							url: true # querystring 으로 url 필수
						headers:
							foo: false # headers 에 foo 선택적 요구
						paths:
							bar: false # path 에 bar 선택적 요구
```

`path` `variable` 이 작동하려면 `API Gateway` 가 다음과 같이 메서드 `path` 자체에도 해당 변수가 필요하다.

```yml
functions:
	create:
		handler: posts.post_detail
		events:
			- http:
				path: posts/{id} # path 상의 id 경로변수 사용
				method: get
				request:
					parameters:
						paths:
							id: true # path 에 id 필수
```

`request` `parameters` 에 의해 다른 `value` 을 매핑하려면, 요청 `parameters` 의 필수 및 `mappedValue` 속성을 정의해야 한다.

```yml
functions:
	create:
		handler: posts.post_detail
		events:
			- http:
				path: posts/{id}
				method: get
				request:
					parameters:
						paths:
							id: true # path 의 id 는 필수
						headers: # headers 적용
							custom-header: # custom-header 
								required: true # 필수
								mappedValue: cotnext.requestId # 매핑된 값
```

### Request Schema Validators

`API Gateway` 와 함께 `request` `schema` `validator`  를 사용하려면, content 타입에 [JSON Schema](https://json-schema.org/) 를 추가해야 한다.

다음은 `create_request.json`  을 만든 `JSON Schema` 이다.

```json
{
	"definitions": {},
	"$schema": "http://json-schema.org/draft-04/schema#",
	"type": "object",
	"title": "The Root Schema",
	"properties": {
		"username": {
			"type": "string",
			"title": "The Foo Schema",
			"default": "",
			"pattern": "^[a-zA-Z0-9]+$"
		}
	}
}
```

>[!warning] 현재 `API Gateway` 에서 제공하는 `JSON Schema` 는 `draft-04` 이다.

`JSON Schema` 는 `JSON` 으로 표시되므로, 파일에서 포함하는것이 더 쉽다

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				request:
					schemas:
						application/json: ${file(create_request.json)}
```

>[!info] `file(...)`  은 `serverless.yaml` 에서 제공하는 문법으로, 해당 `file` 값을 가져온다.

추가적으로, `name` 과 `description` `property` 와 함께 사용자 지정 모델을 생성할수도 있다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				request:
					schemas:
						application/json:
							schema: ${file(create_request.json)}
							name: PostCreateModel
							description: 'Validation modal for Createing Posts'
```

다른 `events` 에서 같은 `model`  을 재사용하고 싶다면, `provider` `level`  에서 `global` `models` 를 지정할수 있다.

`global` `model` 을 추가할 목적으로, `provider.apiGateway.request.schemas`  에 설정을 추가 정의한다.

`global` `model` 을 정의한 이후에, `key` 에 위한 참조를 통해 `event` 안에서 사용할수 있다.

>[!info] serverless.yml
```yml
provider:
	...
	apiGateway:
		request:
			schemas:
				post-create-model: # post-create-model schema 생성
					name: PostCretaeModel
					schema: ${file(api_schema/post_add_schema.json)}
					description: "A Model validation for adding posts"
functions
	create:
		handler: posts.create
		events:
			- http:
				path: posts/create
				method: post
				request:
					schemas:
						# schema 로 post-create-model schema 사용
						application.json: post-create-model 
```

## Lambda Integration

이 `method` 는 좀더 복잡하며, `HTTP` `event` 구문에 대한 더 많은 구성이 필요하다

### Example "LAMBDA" event (before customization)

기본이 아닌 `LAMBDA` `integaration` 방법을 사용하는 경우에만 이를 참조하라고 한다.

```yml
{
  "body": {},
  "method": "GET",
  "principalId": "",
  "stage": "dev",
  "cognitoPoolClaims": {
    "sub": ""
  },
  "enhancedAuthContext": {},
  "headers": {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
    "Accept-Encoding": "gzip, deflate, br",
    "Accept-Language": "en-GB,en-US;q=0.8,en;q=0.6,zh-CN;q=0.4",
    "CloudFront-Forwarded-Proto": "https",
    "CloudFront-Is-Desktop-Viewer": "true",
    "CloudFront-Is-Mobile-Viewer": "false",
    "CloudFront-Is-SmartTV-Viewer": "false",
    "CloudFront-Is-Tablet-Viewer": "false",
    "CloudFront-Viewer-Country": "GB",
    "Host": "ec5ycylws8.execute-api.us-east-1.amazonaws.com",
    "upgrade-insecure-requests": "1",
    "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36",
    "Via": "2.0 f165ce34daf8c0da182681179e863c24.cloudfront.net (CloudFront)",
    "X-Amz-Cf-Id": "l06CAg2QsrALeQcLAUSxGXbm8lgMoMIhR2AjKa4AiKuaVnnGsOFy5g==",
    "X-Amzn-Trace-Id": "Root=1-5970ef20-3e249c0321b2eef14aa513ae",
    "X-Forwarded-For": "94.117.120.169, 116.132.62.73",
    "X-Forwarded-Port": "443",
    "X-Forwarded-Proto": "https"
  },
  "query": {},
  "path": {},
  "identity": {
    "cognitoIdentityPoolId": "",
    "accountId": "",
    "cognitoIdentityId": "",
    "caller": "",
    "apiKey": "",
    "sourceIp": "94.197.120.169",
    "accessKey": "",
    "cognitoAuthenticationType": "",
    "cognitoAuthenticationProvider": "",
    "userArn": "",
    "userAgent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36",
    "user": ""
  },
  "stageVariables": {},
  "requestPath": "/request/path"
}
```

### Request templates

>[!info] Request template 를 사용하기 위해서는 `integration` 이 `lambda` 이어야 한다
#### Default Request Templates

`Serverless` 는 다음의 즉시 사용가능한 `default` `request` `template` 이 함께 제공된다.

1. `application/json`
2. `application/x-www-form-urlencoded`

이 두 `template` 는 `event` 객체로 접근가능한 다음의 `properties` 를 제공한다. 

- body
- method
- principalId
- stage
- headers
- queryStringParameters
- path
- identity
- stageVariables
- requestPath

#### Custom Request Templates

그러나 다음과 같이 `custom` `request` `template` 를 정의하고 사용할수 있다.
기존 `content` `type` 에 대한 새로운 `request` `template` 를 정의하여 기본 `request` `template` 를 덮어쓸수 있다,

```yml
functions:
	create:
		events:
			- http:
				method: get
				path: whatever
				integration: lambda
				request:
					template:
						text/xhtml: '{ "stage" : "$context.stage" }'
						application/json: '{ "httpMethod": "$context.httpMethod" }'
```

>[!note] 이 `template` 는 `plain text` 로 정의된다. 그러나 `${file(templatefile)}` 구문과 함께 외부 `file` 을 참조할수도 있다.

>[!note] `.yml` 안에서, `:`, `{`, `}`, `[`, `]`, `,`, `&`, `*`, `#`, `?`, `|`, `-`, `<`, `>`, `=`, `!`, `%`, `@`, <code>&#96;</code> 를  포함하는 문자열을 인용부호로 사용한다.<br><br>이는 특정 문자가 포함된 문자열을 제대로 파싱하기 이해 인용부호를 사용하여 문자열을 감싸야 한다.

만약, `event` 객체로 `querystrings` 를 매핑하길 원할때, `API Gateway` 에서 `$input.params('hub.challenge')`  를 사용할수 있다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				path: whatever
				method: get
				integration: lambda
				request:
					template:
						application/json: '{ "foo": "$input.params(''bar'')" }'
```

>[!note] `single-quoted` 로 묶인 `string` 을 사용할때, 내용안의 작은 따옴표 `'` 을 사용하고 싶다면, `doubled` `''`  로 이를 `escape` 한다. <br><br>위의 예시는 `lambda` 함수에서 `event.foo` 로`https://example.com/dev/whatever?bar=123` 의 `querystring` `bar` 에 접근할수 있다.<br><br>만약 문자열을 여러줄로 나누고싶다면, `>` 또는 `|` 구문을 사용할수 있다. 그러나 `yml` 문법에 의해 문자열은 같은 크기의 들여쓰기가 되어야 한다.

다음은 `null`  을 전달하여 `default` `request` `template` 중 하나를 삭제한다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				method: get
				path: whatever
				integration: lambda
				request:
					template:
						application/x-www-form-urlencoded: null
```

#### 통과 동작 (Pass Through Behavior)

`API Gateway` 는 `Content-Type` `header` 가 지정한 매핑 `templates` 에 어떠한 것도 매치되지 않는 `request` 를 처리하는 여러방법을 제공한다. 

설정에 따라, `request` `payload` 가 아무런 변형(`trasfomation`) 없이 `request` `integration` 을 통해 전달되거나, `415 - Unsupported Media Type` 으로 `rejected` 될수 있다. 

다음처럼 동작을 정의할수 있다.
>[!info] 만약 아무것도 지정하지 않았다면, `default` 값으로 `NEVER` 를 사용한다

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				method: post
				path: whatever
				integration: lambda
				request:
					passThrough: NEVER
```

이러한 `passThrough` 는 총 $3$ 가지 `option` 이 존재한다.

| value             | Passed Through When                              | Rejected When                                                   |
| :---------------- | ------------------------------------------------ | --------------------------------------------------------------- |
| NEVER             | Never                                            | 정의된 `template` 가 없거나 `Content-Type` 이 정의된 `template` 와 일치하지 않을때 |
| WHEN_NO_MATCH     | `Content-Type` 이 정의된 `template` 와 `match` 되지 않을때 | Never                                                           |
| WHNE_NO_TEMPLATES | 정의된 `template` 가 없을때                             | 하나 또는 여러 `template` 가 정의되어 있지만, `Content-Type` 은 `match` 되지 않을때 |
>[!info] `Content-Type` 이 없거나 비어있다면, `API Gateway` 는 `default` 값으로 간주된다. (`application/json`)

### Responses

`Servless` 는 `custom header` 와 `http` `event` 를 위한 `template` 설정이 가능하다

>[!info] Request template 를 사용하기 위해서는 `integration` 이 `lambda` 이어야 한다
#### Custom Response Headers

다음은  `custom response` `header` 를 설정하는 예시이다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				method: get
				path: whatever
				integration: lambda
				response:
					headers:
						Content-Type: integration.response.header.Content-Type
						Cache-Control: "'max-age=120'"
```

>[!note] `header value` 를 위한 [integration response variables](https://docs.aws.amazon.com/apigateway/latest/developerguide/request-response-data-mappings.html#mapping-response-parameters)  를 사용할수 있다.<br>`Headers` 에 전달된 것과 똑같이 `API Gateway` 에 전달된다.

#### Custom Response Templates

`API Gateway`  가 `Lambda` 출력으로 변환하는데 사용되는  `custom` `response` `template`  를 정의할수 있다.

다음은 `HTML` 로 렌더링 되도록 `lambda` 반환 값을 변환하는 예시이다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				method: get
				path: whatever
				integration: lambda
				response:
					# header 의 Content-Type 을 text/html 로 변환
					headers:
						Content-Type: "'text/html'"
					# API Gateway 에서 Lambda 응답 본문을 그대로 사용
					template: $input.path('$') 
```

>[!info] `API Gateway` 에서 사용하는 `VTL` 에서 제공하는 변수들에 대한 정보는 [API Gateway mapping template and access logging variable reference](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html) 에서 확인가능하다.

>[!info] `$input.path('$')`  에서 `$input` `variables` 는  `mapping templage` 에 의해 처리할 `request` `payload` 와 `parameters` 를 나타낸다.<br><br>그중 `$input.path` 는 `JSONpath` 표현식 문자열을 가지며, 결과로 보여줄 `JSON object` 를 반환한다.<br><br>이를 통해 기본적으로 `Apache VTL` 에서 `payload` 요소에 엑세스하고 조작할수 있다.

### Status Codes

`Serverless` 는 사용할 `default` `status` 코드와 함게 적제되어 사용할수 있다
예를 들어, `resource` 를 찾을수 없다는 $404$ `signal` 또는 이 `action` 에 대한 수행할 권한이 없는 $401$ `signal` 이 있다.

이 `status` 코드는  정규식으로 정의되어 `API Gateway` 설정에 추가된다 

```js
module.exports.hello = (event, context, callback) => {
	callback(null, {
		status: 404,
		body: 'Not Found',
		headers: { 'Content-Type': 'text/plain' }
	})
}
```

#### Avialble Status Codes

| Status code | Meaning               |
| :---------- | --------------------- |
| 400         | Bad Request           |
| 401         | Unauthorized          |
| 403         | Forbidden             |
| 404         | Not Found             |
| 422         | Unprocessable Entity  |
| 500         | Internal Server Error |
| 502         | Bad Gateway           |
| 504         | Gateway Timeout       |
#### Using Status Codes

특정 상태코드를 반환하려면 선택한 `status code ` 에 대괄호를 추가하기만 하면 된다. 아래는 이러한 예시를 사용하여 $404$ `HTTP status` 가 발생하도록 보여준다.

```js
module.exports.hello = (event, context, callback) => {
	callback(new Error(`[404] Not found`))
}
```

#### Custom Status Codes

`Serverless` 에서 제공되는 `default` `status code ` 를 덮어씌울수 있다.
이를 통해 `default statuscode` 를 바꾸고, `status code` 를 추가 삭제하거나, 각 `status code` 에 사용되는 `template` 및 `header` 를 바꿀수 있다.

`pattern` `key` 를 사용하여 반환되는 `code` 를 지정하는 선택 프로세스를 변경할수있다.

만약, `"` 의 패턴과 함께 `status code` 지정한다면, 이는 `default` `response` `code` 가 될것이다. 
`post` `request` 에 대한 기본값을 $201$ 로 변경하는 방법은 아래와 같다ㅏ

`default status code` 를 생략한 경우, 표준 `default` $200$ `status code` 가 생성된다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				method: post
				path: whatever
				integration: lambda
				response:
					headers:
						Content-Type: "'text/html'"
					template: $input.path('$')
					statusCodes:
						201:
							pattern: '' # default response method
						409:
							 # JSON response
							pattern: '.*"statusCode": 409,.*'
							# JSON return object
							template: $input.path("$.errorMessage")
							headers:
								Content-Type: "'application/json+hal'"
```

`pattern`  부분을보자. $409$ `status code` 는 응답 본문에서 `statusCode: 409,` 패턴이 있는지 확인한다.

만약, 다음처럼 `error` 가 발생하고, `Lambda` 함수가 다음의 응답 본문을 반환한다고 하자.

```json
{
	"statusCode": 409,
	"errorMessage": "Conflict occurred"
}
```

`API Gateway` 는 `statusCode: 409,` 패턴을 찾고, $409$ `pattern` 규칙을 적용한다.

결과적으로 `API Gateway` 는 `response` `body` 에서 `errorMessage` `field` 값인 `"Conflict occurred"` 를 추출하여 `client` 로 반환한다.

또한, `Content-Type` 에 따라 `response` `template` 를 바꾸어 생성할수도 있다.
이때, `template` 의 `key` 는 `Cotent-Type` 으로 지정한다.

```yml
functions:
	create:
		handler: posts.create
		events:
			- http:
				method: post
				path: whatever
				integration: lambda
				response:
					headers:
						Content-Type: "'text/html'"
					template: $input.path('$')
					statusCodes:
						201:
							pattern: '' # default response method
						409:
							 # JSON response
							pattern: '.*"statusCode": 409,.*'
							# JSON return object
							template:
								# JSOn return object
								application/json: $input.path("$.errorMessage")
								# XML return object
								application/xml: $input.path("$.body.errorMessage")
							headers:
								Content-Type: "'application/json+hal'"
```

## API Gateway 에 HTTP Proxy 설정

`HTTP proxy` 를 설정하려면, 두개의 `CloudFormation` `template` 가 필요하다.
하나는 `endpoint` (`CloudFormation` 에서는 `resource` 라 함) 용이고 다른 하나는 `method` 용이다.

이 $2$ 개의 `template` 는 함께 작동하여 `proxy` 를 구성한다.

그래서 만약 `serverless.com` 을 위한 `proxy` 로써 `your-app.com/serverless` 를 설정하기 원한다면, `serverless.yml` 에서 다음의 $2$ `template` 가 필요하다

```yml
service: service-name
provider: aws
functions: ...

resources:
	Resources:
		ProxyResource:
			# ApiGateway endpoint 구성 template
			# CloudFormation 에서는 Resource 라 부른다
			Type: AWS::ApiGateway::Resource 
			Properties:
				# endpoint 의 부모 ID
				# 이는 `ApiGatewayRestApi` 의 `RootResourceId` 이다
				ParentId:
					Fn::GetAtt:
						- ApiGatewayRestApi
						- RootResourceId
				# endpoint 의 path 부분
				# 여기서는 `serverless` 이다.
				PathPart: serverless
				# 참조하는 RestApi 의 Id
				# 여기서는 ApiGatewayRestApi 이다.
				RestApiId:
					Ref: ApiGatewayRestApi

		# Proxy 의 method 생성
		ProxyMethod:
			# ApiGateway 의 Method 구성 template
			Type: AWS::ApiGateway::Method
			Properties:
				# 위에 만들어둔 Proxy Resource id
				ResourceId:
					Ref: ProxyResource
				# RestApi 의 id
				RestApiId:
					Ref: ApiGatewayRestApi
				# HttpMethod 는 Get
				HttpMethod: GET
				# Method 의 응답 코드
				MethodResponses:
					- StatusCode: 200
				# Backend 로 보낼 Integration 설정
				Integration:
					IntegrationHttpMethod: POST
					Type: HTTP
					Uri: http://serverless.com
					IntegrationResponses:
						- StatusCode: 200
```

>[!info] `AWS::ApiGateway::Method` 에서 `Integration` 과 `Method` 는 다르다.<br><br>약간 설정상 헷갈릴까봐 추가해서 적는다<br><br> `httpMethod` 는 클라이언트가 `API Gateway` 에 요청을 보낼때 사용하는 `HTTP` 메서드를 정의한다<br><br> `IntegrationHttpMethod` 는 `API Gateway` 가 요청을 `backend` `service` 에 보낼때 어떤 메서드를 사용할지 결정한다.<br><br>이는 다음과 같은 시나리오가 가능하다

```plantuml
Client->"APIGateway IntegrationHttpMethod" : HTTP GET
"APIGateway IntegrationHttpMethod" -> backend : HTTP POST
```

이 두 `template` 에는 많은 일이 진행되고 있지만, 간단한 `proxy` 를 설정하기 위해 알아야 할것은 `proxy` 의 `method` 및 `endpoint` 와 `proxy` 를 설정하려는 `URI` 를 지정하는것이다.

이 `CloudFormation` `template` 를 정의한후, `serverless deploy` 를 하기만 하면 `service`  와 함께 이러한 사용자 정의 `resource` 를 배포하고 `REST API` 에 `proxy` 를 설정한다.

## VPC Link 를 사용하여 private resource 접근

만약 `Edge` 최적화되거나, `Regional API Gateway` 를 가진다면, `VPC Link` 를 사용하여 내부 `VPC` `resource` 에 접근할수 있다. 

이를 위한 `http-proxy`, `vpc-link` `integration` 이 가진 설정을 사용할수 있다.

```yml
- http:
	path: v1/repository
	method: get
	integration: http-proxy
	connectionType: vpc-link
	connectionId: '{your-vpc-link-id}'
	cors: true
	request:
		uri: http://www.githug.com/v1/repository
		method: get
```

## Mock Integration

`Mock` 은 `API` 에 대한 시뮬레이션된 `method` 를 개발자에게 전달해준다.
이를 위해 `integration` `backend` 없이 즉각적으로 `response` 를 정의해줄수 있다

```yml
functions:
	hello:
		handler: handler.hello
		events:
			- th
```