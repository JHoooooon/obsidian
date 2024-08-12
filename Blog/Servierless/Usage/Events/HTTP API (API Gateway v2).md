
`API Gateway` 를 사용하면 `HTTP APIs`  로 배포할수 있다.
이는 총 $2$ 가지 `version` 이 존재한다.

- v1 은 `REST API` 라 부른다
- v2 는 `HTTP API` 라 부르며, 이는 `v1` 보다 저렴하고 빠르다.

이 서비스는, `v2` 를 설명한다.

>[!info] [Choose between REST APIs and HTTP APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html)를 보면 `API Gateway`  가 총 $2$ 가지 타입을 제공하는것을 볼수 있다.
>
>
>두 기능 다 `REST APIs` 이지만, 기능적인 면에서 차이가 있다.
>
>**HTTP APIs**: `HTTP APIs` 는 저렴한 가격과 최소한의 기능을 사용하도록 구성되어 있다.
>
>**[[REST APIs]]**: `REST APIs` 는 `API keys`, `client 제한`, `request validation`, `AWS WAF`, `private API Endpoint` 등 필요한 더 많은 기능을 제공한다.
>
>만약, 이러한 많은 기능이 필요없다면, 더 저렴한 `HTTP APIs` 를 사용할것을 권장한다.

혼란스러운 이름에도 불구하고, 두 버전 모두 `HTTP API`  배포를 허용한다.
여기에서는  `httpApi` `event` 를 통해 `API Gateway v2` `HTTP API` 를 사용한다.

다음은 일반적인 `setup` 방식이다. 

>[!info] General setup
```yml
functions:
	simple:
		handler: handler.simple
		events:
			- httpApi: 'PATCH /elo'
	extended:
		handler: handler.extended
		- httpApi:
			method: POST
			path: /post/just/to/this/path
```

다음은 모든 요청을 받아 처리하는 방식이다.

>[!info] Catch-alls
```yml
functions:
	catchAllAny:
		handler: index.catchAllAny
		events:
			- httpApi: '*'
	catchAllMethod:
		handler: handler.catchAllMethod
		events:
			- httpApi:
				method: '*'
				path: /any/method
```

첫번째 `catchAllAny` 는 모든 `path` 와 모든 `method` 를 보내면 전부다 받아 처리한다.
두번째 `catchAllMethod` 는 `path` 가 `/any/method` 에 대한 모든 `method` 에 대해 처리한다

다음은 `httpApi` 에서 `parmas` 를 처리하는 설정이다

```yml
functions:
	params:
		handler: handler.params
		events:
			- httpApi:
				method: GET
				path: /get/for/any/{param}
```

이는 `/get/for/any/{param}` 경로에 `{param}` 값을 사용하여 `handler` 에서 받아 처리가능하다
이는 보통 동적으로 `path` 값이 변경되는 경우 많이 사용한다.

>[!info] `express` 의 `/get/for/any/:param` 과 비슷한 설정이다.<br>`express` 의 `callback` 에서 `res.params.param` 형식으로  참조하는데, `event` 객체를 사용하여 처리한다.<br><br>위 같은 경우 `event.pathParameters.param` 값으로 `{param}` 값을 가져올수 있다.

## Endpoints timeout
`
`API Gateway` 는 $30s$ 시간 제한이 있다. 
그래서 `function` `timeout` 을 $29s$ 미만으로 유지해야한다.

그렇지 않으면, $503$ `status code` 와 함께, 성공적인 `lambda` 호출을 보게된다.

매우 혼란스러운 상황이다.

정리하자면,  `lambda` 와 `API Gateway` 의 `timeout` 이 다르게 작동한다는 것이다.
`lambda` 의 `timeout` 이 $30s$ 보다 크더라도, `API Gateway` 에서는 $30s$ 동안 어떠한 응답을 생성하지
않으면 $503$ `status code` 를 보내는 것이다.

이는 `lambda` 가 그 이후에 완료되더라도, `API Gateway`  는 자신의 `timeout` 시간을 못지켰으므로 제대로된 처리가 이루어지지 않는다.

## CORS Setup

`HTTP APIs` 와 함께 `CORS` 설정이 가능하다. 
이는 구성된 모든 `endpoints` 에 적용된다.

`Default` 로 구성된 `CORS` 는 다음처럼 구성하면 된다.

```yml
provicer:
	httpApi:
		cors: true
```

이는 다음과 같은 `header` 와 같다.

| Header                       | Value                                                                                                       |
| :--------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Access-Control-Allow-Origin  | *                                                                                                           |
| Access-Control-Allow-Headers | Content-Type, X-Amz-Date, Authorization, X-Api-Key, X-Amz-Security-Token, X-Amz-User-Agent, X-Amzn-Trace-Id |
| Access-Control-Allow-Methods | OPTIONS, (...all defined in endpoints)                                                                      |
만약, `CORS` `headers` 를 조정해야 한다면, 각 `설정`은 다음처럼 각각 설정가능하다.

```yml
provider:
	httpApi:
		cors:
			allowdOrigins:
				- https://url1.com
				- https://url2.com
			allowedHeaders:
				- Content-Type
				- Authorization
			allowedMethod:
				- GET
			allowCredentials: true
			exposedResponseHeaders:
				- Special-Response-Header
			maxAge: 6000
```

## JWT Authorizers

설정된 `HTTP API` `endpoints` 로 접근을 제한하는 한가지 방법중 하나로 `JWT` `Authorizer` 가 있다.

만약, `API` 의 `route` 에서 `JWT` `Authorizaer` 를 설정한다면, `API Gateway` 는 `API` 요청과 함께 제출하는 `JWT` 의 유효성을 검사한다.

`API Gateway` 는 이 `token` 검증을 기반으로 요청을 허락하거나 거부한다.

만약, `route` 에 의해 설정된 `scope` 가 정의되어 있다면, `token` 은 반드시 `route` 의 `scope` 중 하나 이상이 포함되어야 한다.

`API` 의 각 `route` 에 의해 별도의 `authorizer` 구성이 가능하며, 또한 여러 `routes` 가 같은 `authorizer` 를 사용할수도 있다.

>[!info] `JWT Access Token` 을 `OpenID`, `Connect ID` 처럼 다른 유형의 `JWT` 와 구별하는 표준 매커니즘은 없다.<br><br> 이에 대해서, `authorization` `scope` 를  요구하도록 `routes` 를 구성하는것을 추천하다.<br>또한 `ID` `provider` 는 `JWT Acess Token` 을 발급할때만 사용할때만 사용하는`issuers` 또는 `audiences` 를 요구하도록 하여 `JWT authorizer` 를 설정할수 있다.<br> <br><br>[AWS Control access to HTTP APIs with JWT authorizers in API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-jwt-authorizer.html)에서 이렇게 설명한다.) 

### Authorizing API requests with a JWT authorizer

`API Gateway` 는  다음의 일반적인 `workflow` 를 사용하여, `route` 에 대한 `authorization` `request` 를 승인한다.

이는 `JWT Authorizer` 를 사용하도록 구성된다.

1. [Authorizer Properties](https://docs.aws.amazon.com/apigatewayv2/latest/api-reference/apis-apiid-authorizers-authorizerid.html#apis-apiid-authorizers-authorizerid-properties) 중 `token` 의 `identitySource` 를 확인한다.<br>`identitySource` 는 오직 `token` 혹은 `Bearer` `prefix` 를 가진 `token`  을 포함하고 있다.

2. `token` 을 `decode` 한다.

3. `issuer` 의 `jwks_uri` 에서 가져온 `public key` 를 사용하여, `token` `algorithm` 과 `signature` 를 확인한다. <br><br>현재는 `RSA-based Algorithm` 만 지원가능하다.<br>`API Gateway` 는 $2$ 시간 동안 `public key` 를 `cache` 할수 있다.<br><br>가장 좋은방법은, `key` 를 교체할때 이전 `key` 와 새로운 `key` 에 유효한 유예 기간을 허용하는것이다.

4. `claims` 를 검증한다. `API Gateway` 는  다음의 `claims` 를 평가한다. <br><br>**kid**: 이 `claim` 에는 `token` 에 서명한 `jwks_uri` 의 `key` 와 일치하는 `header` `claim` 이 있어야 한다.<br><br>**iss**: 이 `claim` 은 `authorizer` 에 대해 설정된 `issuer` 와 일치해야 한다.<br><br>**aud**: 이 `claim` 은  `authorizer` `property` 인 [audience](https://docs.aws.amazon.com/apigatewayv2/latest/api-reference/apis-apiid-authorizers-authorizerid.html#apis-apiid-authorizers-authorizerid-model-jwtconfiguration) 중 하나와 일치해야 한다.<br>`API Gateway` 는  `aud` 가 없는 경우에만 `client_id` 의 유효성을 검사한다.<br>만약, `aut` 와 `client_id` 둘다 제공된다면, `API Gateway`  는 `aud` 를 평가한다.<br><br>**exp**: 이는 `UTC` 인 현재 시간이후이어야 한다.<br>`expiration time` 만료시간 의 약자이다<br><br>**nbf**: 이는 `UTC` 인 현재 시간 이후이어야 한다.<br>`Not Before` 의 약자이며, 특정 시간 이후에만 유효한 시간을 설정한다<br>이는 말 그대로 지정한 특정 시간 이전에는 유효한 `token` 이 아니며, 반드시 이 시간 이후에만 유효한 `token` 으로 사용된다.<br><br>**iat**: 이는 `UTC` 인 현재 시간 이후이어야 한다.<br>이는 `token` 발급 시간을 나타낸다.<br><br>**scope or scp**: 이 `claim` 은 `route` 의 `authorizationScopes` 안의 `scope` 중 하나이상을 반드시 포함해야 한다.

>[!info] `kid` 관련해서 내용을 정리해야 겠다.
`jwks_uri` 와 같다고 했는데, 이는 `AWS` 에서 `JWT` 기반 인증을 사용할때, 특히 `API Gateway`  와 같은 서비스에서 `AWS Lambda Authorizer` 를 설정할때 사용한다.
>
>`jwks_uri` 의 `jwks` 는  `JSON Web Key Set` 의 약자이며, `JSON` 형식으로 된 공개키 집합이다.
이 `key` `set` 은 `JWT` 를 검증하기 위해 사용된다.
>
`JWT` 는 `signed` 되어있으며, 이 `sign` 을 검증하기 위해 `public key` 가 필요하다.
>
그럼 뒤의 `uri` 가 붙는데, 이는 `JWKS` 를 제공하는 `URI` 라는 뜻이다.
이 `URI` 는 `server` 가 `public key` 를 제공하는 위치를 나타낸다.
이를 통해 `public key` 를 얻는것이다.
>
즉, `kid` 가 `jwks_uri` 의 `key id` 와 일치한 `key id` 를 저장하고, `jwks_uri` 에서 해당 `key id` 를 조회하여, `JWKS` 를 얻어오는 방식인듯하다.

만약 이중 하나의 `step` 에서 `fail` 한다면, `API Gateway`  는 `request` 를 거부한다.

`JWT` 를 확인한후 `API Gateway`  는 `token` 의 `claim` 을 `API route` `integration` 에 전달한다. 
`Lambda functions` 같은 `backend` `resources` 는 `JWT claims` 에 접근할수 있다.

예를 들어, 만약 `JWT` 가 `emailID` `claim` 을 포함하고 있다면, 이는 `Lambda integration` 안의 `$event.requestContext.authorizer.jwt.claims.emailID` 에 존재한다.

이에 대한 더 많은 정보를 얻고 싶다면, [Create AWS Lambda proxy integrations for HTTP APIs in API Gateway](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-lambda.html) 을 확인하는것이 좋을듯 싶다.

>[!info] `API Gateway` 에서는 `Integration` 이라는 개념을 갖는다.<br><br>간단하게 연결되어 있는 `service` 에 `request` 및 `response` 를 하기 전에, 전처리 및 후처리하는 `proxy` 이다.<br><br>실제로, `request` 를 받아 `proxy` 에서 `integraion` 을 통해 캡슐화 하면, 이는 `IntegrationRequest` 로 갭슐화 되며,  `response` 는 `IntegrationResponse` 로 캡슐화 된다.

이를 통해 `Serverless.yaml` 에서 `authorizer` 와 함께 지원하려면 다음 처리한다. 

>[!info] provider.httpApi.authorizers
```yml
provider:
	httpApi:
		authorizers:
			someJwtAuthorizer:
				# type 은 jwt 이다
				type: jwt
				# identitySource 는 token 혹은 Beare toekn 
				identitySource: $request.header.Authorization
				# issuerUrl 은 jwks 이 저장된 url 여기서는 cognito 를 사용
				issuerUrl: https://cognito-idp.${region}.amazonaws.com/${cognitoPoolId}
				# audience entries 설정
				# 이후 발급된 jwt 의 aud claim 에서 이 entries 에
				# 포함되어 있는지 검증
				audience:
					- ${client1Id}
					- ${client2Id}

```

>[!info] 설정된 `endpoint` 는 접근 제한이 될것으로 예상
```yml
functions:
	someFunction:
		handler: index.handler
		events:
			- httpApi:
				method: POST
				path: /some-post
				authorizer:
					# authorizer 로 someJwtAuthorizer 를 사용
					name: someJwtAuthorizer
					# scopes 는 요청이 검증된 이후에 사용된다.
					# 이는 `scopes claim` 함수의 scopes 를 가졌는지 인증을 확인한다
 					scopes: # Optional
						- user.id
						- user.email
```

### Custome Authorizer 설정

`HTTP API` `endpoint` 접근 제한을 하는 다른 방법으로 `customer` `Lambda` `Authorizer` 를 사용한다.
`custom authorizer` 를 위한 `options` 는 다음과 같다,

- **type**: `Lambda authorizer` 를 위해 `request` 로 설정해야 한다. (고정값인듯 하다)

- **name [optional]**: 생성된 `authorizer` 의 이름

- **functionName**: `authorizer` `function` 으로 사용되는 `service` 이름을 정의한다.<br>`functionArl` 을 설정할때, 정의할수 없다. 

- **functionArn**: `authorizer` `function` 으로 사용될 `function`  `ARN` <br>이는 `CloudFormaion` 의 `Instrinsic Functions` 을 허용한다.<br>`functionName` 을 설정할때, 정의할수 없다.

- **resultTtlInSeconds [optional]**: `chache` 된 `authorizer` 결과를 위한 `TTL` 이다.<br>이는 $0$ (`no caching`) 부터 $3600$ ($1 hour$) 까지 값을 허용한다.<br>$0$ 이 아닌 값을 설정할때, `identitySource` 도 반드시 정의해야 한다.

- **enableSimpleResponses [optional]**: `flag` 가 지정된다면, 간단한 `format` 인 `authorization` `response` 을 리턴한다.<br>`default` 는 `false` 이다.

- **payloadVersion [optional]**: `payload` 의 `version` 을 `authorizer` `fuction` 에 보낸다.<br>`default` 는 `2.0` 이다.

- **identitySource [optional]**: `request` `parameters` 에 대한 하나 이상의 매핑된 표현식을 지정한다. 예를 들어, `$request.header.Authorization` 형식으로 지정할수 있다.<br><br>이 지정된 값은 `authorizer` 에 의해 `null` 이 아니고 비어있는 값이 아님을 확인된다.<br> <br>`resultTtlinSeconds` 가 `non-zero` 가 아닌경우, `identitySource` 는 캐시키로 추가로 사용된다. 이경우 `identitySource` 는 인증 응답 캐싱으로 사용된다.

- **managedExternally [optional]**: 만약, `authorizer` 기능이 외부에서 관리되는 지 여부를 지정하는 `flag` 이다.<br>(예, 다른 `AWS` 계정에 존재하는 기능) <br><br>`true` 로 설정한다면,  `authorizer` 기능에 대한 `permission` `resource` 생성을  건너띄게 된다. 

>[!info] `managedExternally` 가 번역체로 해석되다 보니 내용이 애매하다.<br>명확하게 말하자면, `authorizer` 생성시 `permission` 이 자동 생성되는듯하다.<br><br>이때, `permission` 생성을 하지 않고, 이미 만들어 놓은  `authorizer`  기능을 사용하는 경우 `true` 로 설정하여 생성하지 않도록 만들수 있다는것으로 이해하고 있다.

```yml
provider:
  name: aws
  httpApi:
    authorizers:
      customAuthorizer:
        type: request
        functionName: authorizerFunc # `functionArn` 과 함께 사용할수 없다
        functionArn: arn:aws:lambda:us-east-1:11111111111:function:external-authorizer # `functionName` 과 함께 사용할수 없다.
        name: customAuthorizerName
        resultTtlInSeconds: 300
        enableSimpleResponses: true
        payloadVersion: '2.0'
        identitySource:
          - $request.header.Auth
          - $request.header.Authorization
        managedExternally: true # `resource permission` 생성 방지를 위해 외부에서
								# 정의한 `Authorizer` 기능을 사용하는 경우에만 적용가능
```
### Lambda (Request) Authorizers

다음은 `serverless.yam` 의 `service` 를 `cusotom authorizer` 로 설정한다. 
아래는 `provider.httpApi.authorizers` 에서 `labmda` 로 생성한 `authorizerFunc` 를 `request` 시 사용하여 검증한다. 

이렇게 생성한 `customeAuthorizer` 는 `hello` 에서 `authorizer` 로 사용한다.

```
provider:
	name: aws
	httpApi:
		authorizers:
			customAuthorizer:
				type: request
				functionName: authorizerFunc
	functions:
		hello:
			handler: handler.hello
			events:
				- httpApi: 
					method: get
					path: /hello
					authorizer:
						name: customAuthorizer

		authorizerFunc:
			handler: authorizer.handler
```

`authorizer` 로써 `service` 의 바깥에서 사용된 `function` 을 사용할수도 있다.
이는 다음처럼 `arn` 을 사용하여 지정한다.

```yml
provider:
	name: aws
	httpApi:
		authorizers:
			customerAuthorizer:
				type: request
				functionArn: arn:aws:lambda:ap-northeast-2:1111111111:function:external-authorizer

functions:
	hello:
		handler: handler.hello
		events:
			- httpApi:
				method: get
				path: /hello
				authorizer:
					name: customAuthorizer
```

## AWS IAM Authorization

`AWS IAM` 정책을 활용하여 `HTTP API` `endpoint` 를 보호하는 것도 가능하다.

### Control access for invoking an API

[Control access for invoking an API](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-control-access-using-iam-policies-to-invoke-api.html) 은 `IAM permission` 을 사용하여 `API` 접근을 제어하기 위한 `permission` 모델이다.

이 `policy` 










