
`Serverless Framework` 에서 배포시 `AWS S3 bucket` 으로 산출물(`artifact` ) 을 저장할때 필요한 설정이다.  

`bucket` 은 자동적으로 생성되며, `Serverless` 에 의해 관리된다.
그러나 필요하다면, 명시적으로 설정할수 있다.

```yml
provider:
	# 배포된 `artifacts` 가 S3 에 저장될때 붙는 접두사이다. [optional]
	# [default: serverless]
	deploymentPrefix: serverless

	# Serverless Framework 에서 코드 패키지를 Lambda 에 배포하는데
	# 사용되는 S3 bucket 구성 [optional]
	deploymentBucket:
		# 사용할 기존 bucket 의 이름 [optional] 
		# [default: serverless 에 의해 자동 생성된 S3]
		name: com.serverless.${self:provider.region}.deploys

		# 배포시, servierless 에서 이 값보다 오래된 artifacts 를 제거한다.
		# [default: 5]
		maxPreviousDeploymentArtifacts: 10

		# ACLs 또는 bucket policies 를 통한 공개 접근(`public access`) 를 방지
		# note: 배포 bucket 은 기본적으로 public 하지 않다. 이는 추가 ACLs 이다.
		# [default: false]
		blockPublicAccess: true

		# 배포 bucket 이 생성될때, 기본 bucket 정책의 생성을 skip 한다.
		# [default: false]
		skipPolicySetup: true

		# bucket 버저닝을 활성화 [default: false]
		versioning: true

		# server-side 암호화 메서드
		serverSideEncryption: AES256server-side 

		# server-side 암호화인 경우
		sseKMSKeyId: arn:aws:kms:ap-northeast-2:xxxxxxxxxxxx:key/aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa

		# custom keys 와함께 server-side 암호화인 경우
		sseCustomerAlgorithim: AES256
		sseCustomerKey: string
		sseCustomerKeyMD5: md5sum

		# 각 배포 resources 에 추가될 tags
		tags:
			key1: value1
			key2: value2
```

