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



