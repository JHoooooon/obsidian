[[Provider API Gateway v2 HTTP API]] 에서 자세한 내용을 찾아볼수 있다

```yml
provider:
	httpApi:
		# 외부에서 생성한 `HTTP API` 를 `ID` 를 사용하여 연결
		id: xxxx
		
		# `API Gateway API` 대한 사용자 지정 이름
		# deafult: ${sls:stage}-${self:service}
		name: dev-my-server

		# default 인 execute-api HTTP endpoint 를 disabled 한다 (default: false)
		# custom domain 을 사용할때 유용하다
		disableDefaultEndpoint: true

		# 디테일한 CloudWatch 지표를 활성화 (default: false) 
		metrics: true

		# 기본 설정과 함께 CORS HTTP Header 를 활성화 (allow all)
		# 특정 옵션으로 미세조정 가능하다
		cors: true

		authorizers:
			# JWT API authorizer
			someJwtAuthorizer:
				identitySource: $request.header.Authorization
				issuerUrl: https://cognito-idp.us-east-1.amazonaws.com/use-east-1/xxx
				audience:
					- xxxx
					- xxxx

			# 사용자 지정 Lambda request authorizer
			someCustomLambdaAuthorizer:
				# 사용자 지정 Lambda authorizer 의 경우
				# request 를 설정해야 한다.
				type: request

				# function 이름
				# `functionArn` 과는 상호 배타적이다
				functionName: authorizerFunc

				# function ARN
				# `functionName`과는 상호 배타적이다
				functionArn: arn:aws:lambda:us-east-1:111111111:function:external-authorizer 

				# 생성된 authorizer 의 사용자 지정 이름
				# 선택적 값이다.
				name: customAuthorizerName

				# 캐시된 authorizer 결과에 대한 ttl 설정 (선택적 값이다)
				# 0 부터 3600(1 hour) 까지 허용가능하며, 
				# non-zero 값일때 `identitySource` 를 반드시 정의해야 한다
				resultTtlinSeconds: 300

				# 간단한 포멧인 authorization 응답을 반환한다 (default: false)
				enableSimpleResponses: true

				# authorizater 함수로 보낼 payload 의 version (default: '2.0')
				payloadVersion: '2.0'

				# `$request.header.Auth` 형식의 요청 매개변수에 대한 하나 이상의
				# 매핑 표현식을 사용한다
				# 지정된 값은 authorizer 에 의해 비어있지 않고 null 이 아닌것으로
				# 확인된다.
				# `authorizer` 응답 캐싱을 위한 캐시키로 사용된다.
				#
				# 옵셔널한 값이다 단,`resultTtlInSecond` 가 `0` 이 아닐경우 필수
				identitySource:
					- $request.header.Auth
					- $request.header.Authorization

				# 외부에 정의된 authorizer 기능(permission 을 말한다)을 사용할때  
				# 사용하는 flag 이다. 이는 permission 자동 생성을 방지한다
				managedExternally: true
				
```