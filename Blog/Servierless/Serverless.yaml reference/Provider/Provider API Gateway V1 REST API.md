
[[REST APIs (API Gateway v1)]] 와 websocket API 를 적용한다

```yml
provider:
	# API Gateway API 의 사용자 지정이름
	apiName: custom-api-name

	# API Gateway 의 endpoint 타입을 지정: edge or regional (default: edge)
	endpointType: REGIONAL

	# websocket API 에 대한 사용자 지정이름
	websocketsApiName: custom-websockets-api-name

	# 사용자 정의 `route` 선택 표현식
	websocketsApiRouteSelectionExpression: $request.body.route

	# websockets API 에 대한 사용자 정의 설명
	websocketsDescription: Custom Serverless Websockets

	# API Gateway REST API 전역 설정
	apiGateway:
		# ID 를 통해 외부 REST API 연결
		restApiId: xxxx

		# '/' 경로로써 표시되는 Root Resource ID, 
		# apiGateway 에서 resource 를 경로로 생각해도 된다
		restApiRootResourceid: xxxx

		# REST API 에 생성된 기존 resource 목록
		# 이는 필수사항으로, 정의하지 않으면 stack 이 충돌한다고 말한다
		# 말했듯이 resource 는 경로를 말한다
		restApiResources:
			'/users': xxxx
			'/users/create': xxxx

		# ID 로 외부에서 생성된 Websocket API 를 연결
		websocketApiId: xxxx

		# `execute-api` 에서 생성된 기본 HTTP Endpoint 를 disabled (default: false)
		disableDefaultEndpoint: true

		# 사용량 계획(usage plan)에 대한 API key 의 소스: HEADER or AUTHORIZER 
		apiKeySourceType: HEADER
		
```