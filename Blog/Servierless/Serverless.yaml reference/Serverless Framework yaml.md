
`serverless framework` 는 `nodejs` 기반으로 작성된 도구이다.

`serverless.yaml` 은 `serverless framework` 에서 스택을 관리하는 선언 파일로 배포할 클라우드 서비스, 대상, 런타임, 함수와 이벤트, 권한 및 기타 자원을 선언할수 있다.

이는 다음처럼 이루어져있다.

---
- [[Root Properties]] : Root` 에 사용되는 `properties`
- [[Provider General Settings]]:  `Provider` 에서 사용되는 일반적인 설정들
- [[Provider General AWS Lambda settings]]: `Provider` 에서 사용되는 `Lambda` 설정들

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

	# 모든 functions runtime 을 어떻게 관리할지 설정, [default: auto]  
	# 이는 'auto 혹은 'onFunctionUpdate', 'manual' 이 있다. 
	# - auto: 보안 패치와 마이너 버전 업데이트가 자동으로 적용
	# - manual: runtime 업데이트를 수동으로 관리
	# - onFunctionUpdate: 함수별로 runtime 설정을 다르게 한다.
	runtimeManagement: auto

	# functions 의 default memory size [default: 1024MB]
	memorySize: 512

	# functions 의 default timeout [default: 6 seconds]
	timeout: 10

	# 모든 functions 의 AWS Lambda 환경 변수 [optional]
	environment:
		APP_ENV_VARIABLE: FOOBAR

	# ColudWatch log 의 유지기간, [optional] [default: forever]
	# 이는 각 functions block 에서 override 할수 있다.
	logRetentionInDays: 14

	# CloudWatch logs 의 민감한 데이터 마스크와 모니터에 대한 정책 [optional]
	logDataProtectionPolicy:
		Name: data-protection-policy

	# 모든 AWS Lambda functions 의 암호화에 사용할 KMS key ARN
	kmsKeyArn: arn:aws:kms:ap-northeast-2:XXXXXX:key/some-hash

	# AWS Lambda functions 패키징을 위한 Serverless Framework 에서 사용하는
	# Hasing Algorithme 의 version
	lambdaHashingVersion: 20201221

	# AWS Lambda function 의 versioning 을 사용 [optional] [default: true]
	vertionFunctions: false

	# AWS Labmda Processor 아키텍처 [optional] [default: 'x86_64']
	# - x86_64
	# - arm64
	architecture: x86_64
```




