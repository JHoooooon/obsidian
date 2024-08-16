
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

		# REST API 를 위한 API Keys 리스트
		apiKeys:
			- name: myFirstKey
			  value: myFirstKeyValue
			  description: MyFirstKeyDescription
			  customerId: MyFirstKeyCustomerId
			  # key 를 제거하지 않고 비활성화 시키는데 사용 (defalut: true)
			  enabled: false
			- ${sls.stage}-myFirstKey
			# serverless variable 을 숨기고 싶을때 환경변수에 접근하여 가져온다
			- ${env:MY_API_KEY}

		# 지정한 size 보다 클때 응답을 압축(반드시 0 부터 10485760 byte 사이 이어야 한다)
		minimumCompressionsSize: 1024

		# API Gateway 단계 배포에 대한 설명
		description: Some description

		# API 에서 반환할수 있는 binary media 타입 (옵셔널)
		binarymediaTypes:
			- '*/*'

		# 디테일한 Cloud Watch 지표 활성화 (옵셔널)
		metrics: false

		# API Gateway 에서 defalut 로 `${service}-${stage}` 이름을 사용한다
		# v3 에서는 기본적으로 true 이다
		shouldStartNameWithService: false

		# resource 정책
		resourcePolicy:
			  # 허용 정책
			- Efface: Allow
			  # 엑세스 허용주체 정의
			  Principal: '*'
			  # resource 동작정의: API Gateway 호출동작
			  Action: execute-api:Invoke
			  # API Gateway 가 동작할 resource 정의
			  # arn 형식 설정
			  Resource:
				  # execute-api: API Gateway 서비스를 사용하는 리소스 유형
				  # /*/*/*:
				  # 첫번째 *: API Gateway 의 특정 API ID (* 은 모든 API)
				  # 두번째 *: HTTP 메서드 (* 은 모든 HTTP 메서드)
				  # 세번째 *: API 의 특정 리소스 경로 (* 은 모든 리스소 경로)
				  - execute-api:/*/*/*
			# 정책이 적용될 조건을 정의
			  Condition:
				  # 특정 IP 주소에서만 API 를 호출하도록 제한
				  # IpAddress: IP 주소 기반으로 정의
				  IpAddress:
					# aws:SourceIp: 요청을 보낸 IP 주소
				    aws:SourceIp:
						# 123.123.123.123 으로 지정된 Source IP 만 호출 허용
						- '123.123.123.123'

		# 사용량 계획(Usage Plan) 설정 (선택적)
		usagePlan:
			# 사용한도 설정 (quota: 쿼터)
			quota:
				# 한달동안 API 를 최대 5000 번 호출하도록 제한
				limit: 5000
				# 쿼터의 시작지점을 조정한다
				# 예를 들어, 매월 1일에 리셋되는 대신, 특정 날짜에 리셋되도록 설정할수 있다
				# offset: 0 은 매월 1일을 뜻함
				# offset: 2 는 매월 1일 + 2일 = 3일을 뜻함
				offset: 2
				# 쿼터의 리셋주기를 정의
				# DAY, WEEK, MONTH 등으로 설정
				# MONTH 로 설정하면 매월 1일에 쿼터가 리셋 (offset: 0 이어야함)
				preiod: MONTH
			# 제어 제한 설정
			throttle:
				# API 의 급격한 요청 폭주를 방지하기 위해 한번에 요청되는 최대 요청수
				burstLimit: 200
				# 초당 허용되는 요청수
				rateLimit: 100

		# API Request Integration 
		request:
			# Request 검증 Schema 모델 설정
			# 이는 `http events` 에서 재사용 할수 있다
			# 항상 `application/json` 인 `content type` 으로 정의되어야 한다
			schemas:
				# sls 에서 사용할 schema 식별자
				global-model:
					# 사용할 JSON Schema (JSONSchema 로 정의된 파일)
					schema: ${file(schema.json)}
					# API Gateway model 의 이름 지정 (옵셔널)
					name: GlogalModel
					# API Gateway model 의 설명 (옵셔널)
					description: 'A Global model that can be referenced in functions'
```



