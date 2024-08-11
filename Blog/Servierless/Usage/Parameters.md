`parameters` 는 `servierless.yaml` 에 정의할수 있으며, `Serverless Dashboard` 또는 `CLI` 를 통해 전달되기도 한다.

이는 다음의 상황에서 사용된다.

- `stage` 를 기반으로 설정을 조정
- 보안을 위해 `secrets` 저장
- `team` 맴버들간에 설정값 공유 

## CLI parameters

`--param` `flag` 를 전달하여 실행한다
이때, `--param="<key>=<value"` 방식을 취한다

```sh
serverless deploy --param="domain=myapp.com" --param="key=value"
```

이렇게 전달된 `parameters` 는 `${param:xxx}` 변수로 사용될수 있다.

```yml
provider:
	environment:
		APP_DOMAIN: ${param:domain}
		KEY: ${param:key}
```

## Stage parameters

`stages` 는 `stage` 별 설정마다 다르게 `parameters` 설정이 가능하다 
이는 `Serverless V4` 에서 새롭게 `parameters` 를 정의하는 선호되는 방법이다.

앞으로 `stage` 속성에 대한 많은 기능을 출시할 예징이므로, 이를 수용하여 처리하는것이 좋다고 `Docs` 에서는 말한다.

```yml
stages:
	prod:
		params:
			domain: myapp.com
	dev:
		params:
			domain: preview.myapp.com
```

`default` `key` 를 사용하면, 기본적으로 모든 `stages` 에 적용되는 매개변수를 정의한다
이는 특정 스테이지를 재정의하지 않는한 사용되는 기본값이다.

```yml
stages:
	default:
		params:      
			domain: ${sls:stage}.preview.myapp.com  
	prod:    
		params:      
			domain: myapp.com  
	dev:    
		params: domain: preview.myapp.com
```

`parameters` 는 `${param:xxx}` 을 통해 사용할수 있다

```yml
provider:
	environment:
		APP_DOMAIN: ${param:domain}
```

이 `domain` `parameter` 변수는 현재 `stage` 를 기반에 따라 다르게 적용된다.

### Params property

`top level` `property` 인 `params` 를 사용하여, `stage` 별 `parameters` 를 설정할수 있다
그러나, `Serverless Framework V4` 에서 `parameters` 를 설정하는 방법으로 같은 `top level` `property`  인 `stages` 를 사용하기를 추천한다

>[!info] `Serverless V4` 에서 `stages` `property` 를 더 확장하여 사용하려는듯 하다.

```yml
params:
	default:
		domain: ${sls:stage}.myapi.com
	prod:
		domain: myapi.com
	dev:
		domain: dev.myapi.com
```

## Serverless Dashboard Parameters

`Serverless Dashboard` 는 `SaaS` 솔루션이다.
이는 `Servelress Framework CLI` 는 `AWS` 계정에서 서버리스 애플리케이션을 개발, 배포, 테스트, 보호 및 모니터링할수 있는 강력하고 통홥된 환경을 제공한다.

이러한 `Serverless Dashboard` 는 매개변수를 생성하고, 관리할수 있다.
이는 `team` `member` 들과 설정 값을 공유하거나, 안전하게 `secrets` 를 저장하기에 완벽한 환경을 제공한다.

`Dashboard` `parameters` 는 `service` 혹은 `instance` 에 저장된다.

>[!info] `service` 는 `모든 stage` 에 적용되는 환경이며, `instance` 는 `특정 stage` 에 적용되는 환경을 뜻한다. 

`Dashboard` `parameters` 는 예민한 `value` 로 간주하며, 저장시 암호화 되고, 배포중에만 `decrypted` 되거나, `dashboard` 에서만 값을 볼수 있다. 그 외에서는 암호화된 값으로 처리되어 알수가 없다.

만약 `parameters` 로 `stripeSecret` 이 `Dashboard` 에 저장되었다면, 다음처럼 접근가능하다.

```yml
provider:
	enviroment:
		STRIPE_SECRET_KEY: ${params:stripeSecret}
```

### Inheritance and overriding

`Parameters` 는 `stage` 당 `serverless.yml` 안에 정의할수있다.
이는 `Servierless Dashboard` 역시 마찬가지며, 이는 `service` 또는 `instance`(`stage`) 당 정의가능하다.

여기에서, `${parma:xxx}` 사용시 선호도에 의해 처리되는 방법은 다음과 같다.

1. 첫번째로, `--params` `CLI` `flag` 로 전달한다.
2. 만약 찾지 못한다면, `serverless.yml` 안의 `stages.<stage>.parmas` 를 찾는다.
2. 만약 찾지 못한다면, `serverless.yml` 안의 `stages.default.parmas` 를 찾는다.
2. 만약 찾지 못한다면, `Dashboard` 안의 `instance` `parametres` 를 찾는다.
2. 만약 찾지 못한다면, `Dashboard` 안의 `service` `parametres` 를 찾는다.
2. 만약 찾지 못한다면, `error` 를 `throw` 하거나, `${param:xxx, 'default value'}` 에 제공된 `fallback` `value` 를 사용한다.

이러한 방식은 `serverless.yml`  `parameters` 와 `Serverless Dashboard` `parameters` 유연하게 `mix` 하여 사용할수 있다.

이를 통해 `임시 stage` (`ephemeral stages`) 로 배포할때, 개발시 매우 유용하게 사용할수 있다.
이 `임시 stage` 는 어떠한 `parameter` 를 가지지 않을수 있으며, 이는 기본적으로 `service` 에 설정된 `parameters` 가 사용된다.

그러나, 다른 `prod` 혹은 `dev`, `staging` `stage`  같은 경우, `stage` `level` `parameters` 로 재정의하여, 해당 `stage` 에 고유한 `values` 를 사용할수 있도록 가능하다. 









