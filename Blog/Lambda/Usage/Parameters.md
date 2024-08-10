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



