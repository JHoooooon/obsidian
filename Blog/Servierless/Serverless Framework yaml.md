
`serverless framework` 는 `nodejs` 기반으로 작성된 도구이다.

`serverless.yaml` 은 `serverless framework` 에서 스택을 관리하는 선언 파일로 배포할 클라우드 서비스, 대상, 런타임, 함수와 이벤트, 권한 및 기타 자원을 선언할수 있다.

이는 다음처럼 이루어져있다.

## Root Properties

이는 `Root` 에 사용되는 `properties` 이다.
`serverless v4` 로 `update` 되면서, `dashboard` 와 연동되도록 만들어졌는데,
이때, `serverless dashboard` 의 `organization` 의 이름을 지정하며, `app` 에 대한 이름역시 지정한다.

```yml

# org name
# 이는 serverless framework 의 organizationa 이름이다.
# 이는 user account 또는 license key 와 연결되어 있다.
# 이는 optional 한 property 이지만, default 로 .serverless file 안에 자동설정된다.
# 비록, optional 한 property 이지만, 항상 올바른 org 로 deploy 하도록 보장하므로 설정하기를
# 추천한다.
org: mya-org

# App name
# 하나 혹은 여러 services 에 대한 container 로써 행동한다.
# 이는 serverless.yaml 안의 app property 로 설정된다.
# 이는 service 에 대한 dashboard 기능을 활성화되도록 설정한다.
# 예를들어, deployment 에 대한 tracking, secrets 그리고 outputs 에 대한 공유, 
# metrics 활성화,logs 등등의 기능을 말한다.
# 이는 optional 한 property 이며, 만약 dashboard 기능을 원치 않는다면,설정하지 않을수있다. 
app: my-app

# Service name
# 이는 /project/app/service/my-service 의 이름이다.
service: my-service
```

## Stages

설정을 `stage` 을 사용하여 `stage` 별 구성을 설정한다.

```yml
# service 이름
service: builling

# stages 를 통해 각 stage 를 정의
stages:
	prod: # prod stage
		observability: true # prod stage 에서 observability 를 활성화
		params: # prod stage 에서 사용되는 parameter 값을 정의
			stripe_api_key: ${env:PROD_STRIPE_API_KEY}
	default: # default stage
		observability: false # default stage 에서 observability 비활성화
		params: # default stage 에서 사용되는 parameter 값을 정의
			stripe_api_key: ${env:DEV_STRIPE_API_KEY}

```

## Parameters

[[Parameters]] 를 통해 내용을 확인할수 있다.

```yml
# Stage parameters
# Parameters 는 stage 별 `values` 이다.
# 이러한 Parameters 는 ${params:my-value} 를 통해 참조가능하다.
# default parameters 는 지정된 stage 를 제외한 모든 stage 에서 사용하는 parameters 이다.
# 그 외의 지정된 stage 에서는 설정한 parameters 를 사용한다.
params:
	default:
		domain: ${sls:stage}.myapi.com
	prod:
		domain: myapi.com
	dev:
		domain: dev.myapi.com

# 예로써, foo 에서 사용되는 ${param:domain} 은 stage 에 따라 그 값이 변경된다.
foo: ${param:domain}
```

>[!info] `Serverless v4` 는 `stage` `property` 안에 `parameter` 설정을 선호한다.<br><br>[[Parameters]] 에서 보면,`top level property` 인 `params` 가 아닌  `stages` 에서 `Params`  설정이 가능하다.

## Provider

`AWS` 별 `service` 전반에 대한 세부 정보를 지정한다.

### General Settings

```yml
provider:
	# Provider 의 이름, `V4` 는 오직 `AWS` 만을 지원한다.
	name: aws
	
	# Default stage 를 지정한다. [optional]
	stage: dev
	
	# Default region 을 지정한다. [default: us-east-1]
	region: ap-northeast-2
	
	# deplay 에서 사용할 local AWS profile [default: "default" profile]
	# 여기서 말하는 local AWS profile 은 AWS CLI 설정시 profile 정보가 저장되는 파일에
	# 식별자를 말한다.
	# 보통은 ~/.aws/credentials 파일에 저장되며,해당 파일안에, profile 식별자에 따른
	# aws_access_key_id 와 aws_secret_asscee_key 가 저장되어 있다.
	# 이 경우 profile 식별자가 production 이라 하는것이다.
	profile: production
	
	# coludFormation stack 을 위한 사용자 지정 이름 [optional]
	stackName: custom-stack-name

	# APIs 와 Function 에 적용할 CloudFormation tags [optional]
	tags:
		foo: bar
		baz: qux

	# stack 에 적용할 CloudFormation tags [optional]
	stackTags:
		key: value

	# CloudFormation 배포에서 사용될 method 설정: direct 와 changeset 이 있다.
	# default: direct
	#
	# direct: stack 을 즉시 업데이트
	# changeset: stack 에서 changeset 을 생성
	#            변경사항을 검토하며, changeSet 을 실행하여 stack update 
	deploymentMethod: direct
```

>[!info] deploymentMethod
```sh
> serverless deploy --changeset # changeSet 생성 but, 실행은 안함

> serverless deploy --changeset --info # 생성될 리소스와 변경될 내용을 보여준다

> serverless deploy --changeset --execute
# 변경사항을 검토하고, 원하는 대로 변경이 이루어지는지 확인한후,
# 실제 배포를 진행. 프로덕션 환경에서 중요한 변경을 수행할때 유용하다.
```

이러한 chageSet 은  다음의 이점이 있다.

1. 배포 전 변경 사항 검토가능
2. 의도하지 않은 변경 방지
3. 복잡한 스택 업데이트에 대한 더 나은 제어

```yml
provider:
	... 이어서

	# stack event 에 대한 notification 이 전송되는 같은 region 의
	# 기존 Amazon SNS 주제 목록 [optional]
	notificationArns:
		- 'arn:aws:sns:ap-northeast-2:xxxxxx:mytopic'

	# AWS Cloudformation Stack Parameters [optional]
	stackParameters:
		- ParameterKey: 'Keyname'
		  ParameterValue: 'Value'

	# Cloudeformaion 이 실패하면 자동 rollback Disabled
	# 이는 production 환경이 아닌곳에서 사용된다. [optional]
	disableRollback: true

	# AWS Cloudformation Rollback 설정 [optional]
	rollbackConfiguration:
		MonitoringTimeInMinutes: 20
		RollbackTriggers:
			- Arn: arn:aws:coludwatch:ap-northeast-2:000000000000:alarm:health 
			  Type: AWS::ColudeWatch::Alarm
			- Arn: arn:aws:coludwatch:ap-northeast-2:000000000000:alarm:latency 
			  Type: AWS::ColudeWatch::Alarm

	# AWS X-Ray Tracing 설정 [optional]
	tracing:
		# 만약 stack 안에 API Gateway 가 있는 경우 true
		apiGateway: true
		# true 로 설정할수 있다. (true == 'Active'), 'Active' 또는 'PassThrough'  
		lambda: true
```

### General AWS Labmda Settings

`provider` `key` 안에 모든 `functions` 에 대한 설정사항을 정의할수 있다.

```yml
provider:
	# Service 에 사용되는 모든 Lambda functions 에 대한 AWS Lambda runtime 
	runtime: nodejs20.x

	# 모든 functions runtime 을 제어할 
```
