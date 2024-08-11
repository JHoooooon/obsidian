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
