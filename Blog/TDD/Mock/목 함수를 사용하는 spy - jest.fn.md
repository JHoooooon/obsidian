
`jest` 에서 `mock fn` 로 `spy` 를 구현할수도 있다.
`spy` 는 `test`대상에 발생한 입출력을 기록하는 객체이다.

## toBeCalled: 호출되었는지 검증

`spy` 에 기록된 값을 검증하면 의도한 대로 기능이 작동하는지 확인할수 있다.

```ts
test("mock is being called", () => {
	// jest.fn 을 사용하여 mock fn 객체 생성
	const mockFn = jest.fn()
	// mockFn 호출
	mockFn()
	// 호출된 mockFn 이 호출되었는지 검증
	expect(mockFn).toBeCalled(); // true
})

test("mock is not being called", () => {
	// jest.fn 을 사용하여 mock fn 객체 생성
	const mockFn = jest.fn()
	// 호출된 mockFn 이 호출되지 않았는지 검증
	expect(mockFn).not.toBeCalled(); // true
})
```

## toHaveBeenCalledTimes: 몇번 호출되었는지 검증

```ts
test("to have been called times", () => {
	const mockFn = jest.fn()
	// 1 번 호출
	mockFn()
	// 1 번 호출했는지 검증
	expect(mockFn).toHaveBeenCalledTimes(1)
	// 2 번 호출
	mockFn()
	// 2 번 호출했는지 검증
	expect(mockFn).toHaveBeenCalledTimes(2)
})
```

## toHaveBeenCalledWith: 전달된 인수검증

`mock` 함수는 실행시 전달된 인수를 기록한다.

```ts
test("to have been called with args", () => {
	const mockFn = jest.fn()
	const greet = (name: string) => {
		mockFn(name)
	}
	greet("yogafire")
	expect(mockFn).toHaveBeenCalledWith("yogafire") // true
})
```

## spy 로 활용

`jest.fn`  을 사용하는 `spy` 는 테스트 대상의 인수에 함수가 있을때 유용하게 활용할수 있다.

>[!info] greet.ts
```ts
export function greet(name: string, cb?: (msg: string) => void) {
	cb?.(`Hello, ${name}!`)
}
```

>[!info] greet.test.ts
```ts
import { greet } from "greet"

test("목함수를 테스트 대상의 인수로 활용", () => {
	const mockFn = jest.fn()
	const name = "yogafire"
	greet(name, mockFn)
	expect(mockFn).toHaveBeenCalledWith(`Hello, ${name}!`) // true
})
```

이를 사용하여, `cb` 가 호출되었는지, 어떠한 인자와함께 호출되었는지 검증가능하다

## 인수가 객체일때 검증

>[!info] checkConfig.ts
```ts
const config = {
	mock: true,
	feature: { spy: true },
}

export const checkConfig(cb?: (payload: object) => void) {
	callback?.(config)
}
```

>[!info] `{}` 은 `typescript` 에서 `non-nullable any` 와 같다.<br>이는 `null`, `undefined` 를 제외한 모든 타입을 할당할수 있다. 

>[!info] checkConfig.test.ts
```ts
import { checkConfig } from 'checkConfig'

test("목 함수는 실행시 인수가 객체일대도 검증가능하다", () => {
	const mockFn = jest.fn()
	checkConfig(mockFn)
	expect(mockFn).toHaveBeenCalledWith({
		mock: true,
		feature: { spy: true },
	})
})
```

만약 객체의 일부만 검증하고 싶다면 `expect.objectContaining` 이라는 보조함수를 사용한다

>[!info] checkConfig.test.ts
```ts
import { checkConfig } from 'checkConfig'

test("목 함수는 실행시 인수가 객체일대도 검증가능하다", () => {
	const mockFn = jest.fn()
	checkConfig(mockFn)
	expect(mockFn).toHaveBeenCalledWith(
		expect.objectContainin
	{
		mock: true,
		feature: { spy: true },
	})
})
```