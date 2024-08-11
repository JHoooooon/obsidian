
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
