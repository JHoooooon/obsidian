
`API Gateway` 를 사용하면 `HTTP APIs`  로 배포할수 있다.
이는 총 $2$ 가지 `version` 이 존재한다.

- v1 은 `REST API` 라 부른다
- v2 는 `HTTP API` 라 부르며, 이는 `v1` 보다 저렴하고 빠르다.

이 서비스는, `v2` 를 설명한다.

>[!info] [Choose between REST APIs and HTTP APIs](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vs-rest.html)를 보면 `API Gateway`  가 총 $2$ 가지 타입을 제공하는것을 볼수 있다.
>
>
>두 기능 다 `REST APIs` 이지만, 기능적인 면에서 차이가 있다.
>
>**HTTP APIs**: `HTTP APIs` 는 저렴한 가격과 최소한의 기능을 사용하도록 구성되어 있다.
>
>**[[REST APIs]]**: `REST APIs` 는 `API keys`, `client 제한`, `request validation`, `AWS WAF`, `private API Endpoint` 등 필요한 더 많은 기능을 제공한다.
>
>만약, 이러한 많은 기능이 필요없다면, 더 저렴한 `HTTP APIs` 를 사용할것을 권장한다.

혼란스러운 이름에도 불구하고, 두 버전 모두 `HTTP API`  배포를 허용한다.
여기에서는  `httpApi` `event` 를 통해 `API Gateway v2` `HTTP API` 를 사용한다.

다음은 일반적인 `setup` 방식이다. 

>[!info] General setup
```yml
functions:
	simple:
		handler: handler.simple
		events:
			- httpApi: 'PATCH /elo'
	extended:
		handler: handler.extended
		- httpApi:
			method: POST
			path: /post/just/to/this/path
```

다음은 모든 요청을 받아 처리하는 방식이다.

>[!info] Catch-alls
```yml
functions:
	catchAllAny:
		handler: index.catchAllAny
		events:
			- httpApi: '*'
	catchAllMethod:
		handler: handler.catchAllMethod
		events:
			- httpApi:
				method: '*'
				path: /any/method
```

첫번째 `catchAllAny` 는 모든 `path` 와 모든 `method` 를 보내면 전부다 받아 처리한다.
두번째 `catchAllMethod` 는 `path` 가 `/any/method` 에 대한 모든 `method` 에 대해 처리한다

다음은 `httpApi` 에서 `parmas` 를 처리하는 설정이다

```yml
functions:
	params:
		handler: handler.params
		events:
			- httpApi:
				method: GET
				path: /get/for/any/{param}
```

이는 `/get/for/any/{param}` 경로에 `{param}` 값을 사용하여 `handler` 에서 받아 처리가능하다
이는 보통 동적으로 `path` 값이 변경되는 경우 많이 사용한다.

>[!info] `express` 의 `/get/for/any/:param` 과 비슷한 설정이다.<br>`express` 의 `callback` 에서 `res.params.param` 형식으로  참조하는데, `event` 객체를 사용하여 처리한다.<br><br>위 같은 경우 `event.pathParameters.param` 값으로 `{param}` 값을 가져올수 있다.

## Endpoints timeout
`
`API Gateway` 는 $30s$ 시간 제한이 있다. 
그래서 `function` `timeout` 을 $29s$ 미만으로 유지해야한다.

그렇지 않으면, $503$ `status code` 와 함께, 성공적인 `lambda` 호출을 보게된다.

매우 혼란스러운 상황이다.

정리하자면,  `lambda` 와 `API Gateway` 의 `timeout` 이 다르게 작동한다는 것이다.
`lambda` 의 `timeout` 이 $30s$ 보다 크더라도, `API Gateway` 에서는 $30s$ 동안 어떠한 응답을 생성하지
않으면 $503$ `status code` 를 보내는 것이다.

이는 `lambda` 가 그 이후에 완료되더라도, `API Gateway`  는 자신의 `timeout` 시간을 못지켰으므로 제대로된 처리가 이루어지지 않는다.

## CORS Setup

`HTTP APIs` 와 함께 `CORS` 설정이 가능하다. 
이는 


