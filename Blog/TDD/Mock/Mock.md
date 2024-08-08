
`Mock` 은 `TDD` 에서 많이 쓰인다.
보통, 외부 `API` `Module`  에의해 참조되어야 하는 객체이나, 함수를 테스트용으로 대체해야할때 사용하는 방법이다.

이렇게, 대체를 통해 취득한 `Test Double` 을 `jest` 에서는 `Mock Object` 라 한다.

>[!info] `Test Double` 은 `Stunt Double` 에서 유래한 단어이다.<br>이는 `movie` 나 `TV Show` 에서 위험한 `scenes` 를 위한 `대역` 을 말한다.<br><br>`Test Double` 은 `Test` 를 위한 `대역` 이다.

>[!warning] 뭔가 `programming` 용어로 `대역` 을 사용하기에  약간의 오해가 있을듯 싶다.<br>`대역` 은 `역할을 대신한다` 는 뜻도 있지만, `어떠한 폭으로써 정해진 범위` 라는 의미로도 쓰인다.<br><br>`programming` 에서는 이러한 `대역` 을 `어떠한 폭으로써 정해진 범위` 의 의미로 자주쓰인다.<br> 그러므로, 이를 구분하기위해 `Test Double` 이라 할것이다.

보통의 `TDD` 에서 가장많이 언급하는것중 하나가 `Stub` 과 `Spyon` 이다.

## Stub

`Stub` 은 `Test Double`(`대체역할`) 로 사용하는 용도이다.

- 의존중인 모듈의 대체 용도
- `TDD` 에서 사용할 정해진 값을 반환하는 용도
- `TDD` 대상에 할당하는 입력값을 지정하는 용도

`Test` 대상이 `Stub` 에 접근하면, 의존중인 모듈을 대체하여 `TDD` 에서 사용될 용도로 로직을 수정할수 있다.

이러한 로직의 수정은 `parameter` 값을 지정할수 있으며, 심지어 반환하는 값을 미리 지정해둘수도 있다.

>[!info] `Stub` 이라는 단어가, `쓰다남은 토막`, `그루터기` 라는 뜻이 있는데, 마치 `나루토` 에서 [바꿔치기술](https://namu.wiki/w/%EB%B0%94%EA%BF%94%EC%B9%98%EA%B8%B0%EC%88%A0)을 연상해 볼수 있다.

이는 기존의 함수가 구현되어 있지만, 외부 `API` 에 의존성을 가짐으로 `TDD` 가 복잡해지거나, 어려울때 유용하다.

외부 `API` 의 의존성으로, 그 결과값을 이미 지정해두면 그 결과값만을 사용하여, 의존성에 의한 복잡함을 `대체` 할수 있기 때문이다.

>[!info] greet.ts
```ts
export function greet(name: string) {
	return `Hello, ${name}`;
}

export function sayGoodBye(name: string) {
	return new Error(`미구현`);
}
```

>[!info] greet.test.ts
```ts
import { greet } from './greet';
jest.mock("./greet")

test(`greet()`, () => {
	expect(greet("Taro")).toBe(undefined) // True
})
```

`jest.mock` 을 호출하면 테스트할 모듈을 대체한다.
이러한 대체된 모듈의 메서드들은 `mock fn` 으로 생성되었지만, 아직 구현되지 않은 상태이므로 `undefined` 이다.

이때, 구현사항을 다음처럼 정의한다.

>[!info] greet.test.ts
```ts
import { greet } from './greet';
jest.mock("./greet", () => ({
	sayGoodBye: (name: string) => `Good bye, ${name}`,
}))

test(`greet()`, () => {
	const name = "Taro"
	expect(greet(name)).toBe(undefined) // True
	expect(sayGoodBye(name)).toBe(`Good by, ${name}`) // True
})
```

`sayGoodBye` 의 반환값을 구현했다.
실제, `sayGoodBye` 는 `new Error("미구현")` 으로 `error` 를 `throw` 하지만, `test` 에서는 `return` 값으로 대체하여 다른 결과값을 비교하도록 만들었다.

이러한 동작을 하는것이 `Stub` 이다.
다음은 일부 모듈을 `Stub` 으로 대체한다.

>[!info] greet.test.ts
```ts
import { greet, sayGoodBye } from './greet'

jest.mock('/greet', () => ({
	...jest.requireActual('./greet'),
	sayGoodBye: (name: string) => `Good bye, ${name}`,
}))

test(`greet()`, () => {
	const name = "Taro"
	expect(greet(name)).toBe(`Hello, ${name}`) // True
	expect(sayGoodBye(name)).toBe(`Good by, ${name}`) // True
})
```

`jest.requireActual` 함수를 사용하면, 기존의 구현 함수를 가져와 `mocking` 화 한다.
이렇게 생성된 `mock` 함수에 `sayGoodBye` 를 구현하여 덮어씌워 `Test` 용으로 구현한다. 

이로인해 이전의 `test` 에서 `greet` 함수가 `undefined` 가 아닌 `Hello, ${name}` 값을 반환하게 된다.
이러한 `stub` 은 매우 용용하게 사용되는데, 다른 라이브러리 자체를 `mock module` 로 만들수 있다.

```ts
jest.mock("next/router", () => require("next-router-mock"))
```

이는, `nest/router` 의존 모듈을 `next-router-mock` 으로 대체한다.


## Spy

`Spy` 는 기록하는 용도로 사용된다.
>[!info] `Spy fn` 을 `stub` 으로 만들수 있기도 하다.

`jest` 에서 생성하는 `jest.fn` 은 내부적으로, 해당 함수에 대한 `meta` 데이터가 저장된다.

>[!info] 이러한 `meta` 를 통해, `몇번 호출되었는지`, `첫번째 args가 무엇인지`, `return 값이 무엇이지`, `이 함수의 context 가 무엇인지`, `이 함수로 인해 생성된 instance 가 몇개인지`, `이 함수의 name은 무엇인지`, `마지막으로 호출한 함수의 args 가 무엇인지` 등등... 을 알수있다.   

이러한 정보를 토대로하여, 해당 함수를 추적하는데 용이하다.
말 그대로 `Spy`(`도청`) 하도록 만든다.

여기서 생각해볼것이, `jest.fn` 과 `jest.spyOn` 의 차이이다.
`jest.fn` 은, 그저 비어있는 함수이지만, `mock` 으로써, 추적 가능한 `mock` 함수이다.

>[!info] `mock` 함수는 기본적으로, `mocking` 된 함수이므로 추적가능하다.<br>`Stub` 역시 이러한 `mock` 함수로 구현된다.

하지만, `jest.spyOn` 은 이미 구현되어있는 함수의 구현을 그대로 사용하면서, `mocking fn` 으로써 사용한다.





