`API Gateway` 를 사용하면 `HTTP APIs`  로 배포할수 있다.
이는 총 $2$ 가지 `version` 이 존재한다.

- v1 은 [[REST APIs (API Gateway v1)]] 라 부른다
- v2 는 [[HTTP API (API Gateway v2)]] 라 부르며, 이는 [[REST APIs (API Gateway v1)]] 보다 저렴하고 빠르다.

이 서비스는, `v2` 를 설명한다.

---

`AWS Lambda` 에 대한 `Eevent`소스로  `HTTP endpoint`  를 생성하려면, `Servelress Framework` 의 간편한 `API Gateway` `Event` 를 사용하면 된다.

`AWS Lambda` 함수와 함께 `HTTP endpoint` 로 `integrate` 하기 위해 총 $5$ 가지 `method` 로 설정할수 있다.
> [!info] `통합하다` 로 해석하는데, `통합` 이 뭔가 잘 안들어온다. `integrate` 로 해석한다.

###  `lambda-proxy` / `aws-proxy` / `aws_proxy` (`Recommended`)

이 `Framwork` 는 사용자에 의해 다른 메서드들을 제공하지 않는한, `lambda-proxy` 메서드를 `default` 로 사용한다. 

`lambda-proxy` 는 자동적으로  `HTTP Request` 의 `content` 를 `AWS Lambda` 함수로 전달한다.
그리고, `AWS Lambda` 함수의 `code` 안에서 `response` 설정이 가능하도록 허용한다.

>[!info]  `lambda-proxy` `aws-proxy` `aws_proxy` 의 차이
>`AWS Lambda` 함수와 `API Gateway` 를 통합할 유형을 지정하는데 사용하는 용어이다.
>이는 용어를 정리하는 과정에서 파생된 동의어이다.
>
>처음에는 `lambda-proxy` 라는 용어를 사용했지만, 이후, `aws-proxy` 라는 이름으로 변경했다.
>이러한 이유로 인해, 여러 용어가 동의어로써 사용되며, `Serverless Framework` 에서는 보통 `lambda-proxy` 로 사용되는 듯하다.
>
>`aws_proxy` 는 언더스코어를 사용하는 명명 규칙과의 호환성을 의해 추가된것이라고 한다.

### `lambda` / `aws`

`lambda` 혹은 `aws` 메서드를 사용하면, 각 `API Gateway` 에서 명시적으로 `headers`, `status codes` 와 더불어 더 많은 설정을 해야한다.

이러한 `lambda` 메서드는 매우 지루한 방법이 될수 있으므로, `lambda-proxy` 를 사용할것을 권장한다.

### `http`

`HTTP Backend` 와 `integrating` 하기 위해 사용된다

- `http-proxy` / `http_proxy`

`HTTP proxy` `integration` 과 함께 `integrating` 하기 위해 사용된다.

### `mock`

실제 `back end` 호출없이 `testing` 하기 위해 사용된다.

## Lambda Proxy Integration










