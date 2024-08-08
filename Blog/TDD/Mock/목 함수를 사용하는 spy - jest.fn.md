
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
		expect.objectContaining({
			feature: { spy: true }
		})
	)
})
```

## 웹 API 목 객체 처리

다음은 `server` 에서 `유효성 검사` 를 한다고 가정하며, `유효성 검사` 는 다음의 함수를 사용한다

>[!info] checkLength
```ts
export class ValidationError extends Error {};

export function checkLength(value: string) {
	if (value.length === 0) {
		throw new ValidationError("한 글자 이상의 문자를 입력해주세요.")
	}
}
```

다음은, `mock` 객체를 생성하여, 검증한다.

```ts
import { httpError, postMyArticleData } from "./fixtures"
import { checkLength } from "./checkLength"
import * as fetchers from "./fetchres"
import { postMyArticle } from "./postMyArticle"

function mockPostMyArticle(input: ArticleInput, status = 200) {
	const postMyArticle = jest
		.spyon(fetchres, "postMyArticle")
	
	if (status > 299) {
		return postMyArticle.mockRejectedValueOnce(httpError)
	}

	try {
		checkLength(input.title)
		checkLength(input.body)
		return postMyArticle.mockResolvedValue({ 
			...postMyArticleData, 
			...input 
		})
	} catch (err) {
		return postMyArticle
			.mockRejectedValuesOnce(httpError)
	}
}

function inputFactory(input?: Partial<ArticleInput>) {
	return {
		tags: ["testing"],
		title: "타입스크립트를 사용한 테스트 작성법",
		body: "테스트 작성시 타입스크립트를 사용하면 테스트의 유지보수가 쉬워진다",
		...input
	}
}

// inputFactory() 유효성 통과
// inputFactory({ title: "", body: "" }) 유효성 통과 x

test("유효성 검사를 성공하면 성공 응답 반환", async () => {
	// input 생성
	const input = inputFactory()
	// mocking 
	const mock = mockPostMyArticle(input)
	// input 으로 실행할 함수호출
	const data = await postMyArticle(input)
	// data 에 input 이 포함되었는지 확인
	expect(data).toMatchObject(expect.objectContaining(input))
	// mock 이 호출되었는지 확인
	expect(mock).toHaveBeenCalled()
})

test("유효성 검사에 실패", async () => {
	expect.assertions(2)
	// 통과못하도록 input 생성
	const input = inputFactory({ title: "", body: "" })
	// mocking
	const mock = mockPostMyArticle(input)
	// input 검증: rejected 되었는지 확인
	await postMyArticle(input).catch((err) => {
		expect(err).toMatchObject({ err: { message: expect.anyting() } })
		// mock 호출되었는지 확인
		expect(mock).toHaveBeenCalld()
	});
})

test("500 error", async () => {
	expect.assertions(2)
	// 통과못하도록 input 생성
	const input = inputFactory()
	// mocking status = 500
	const mock = mockPostMyArticle(input, 500)
	// input 검증: rejected 되었는지 확인
	await postMyArticle(input).catch((err) => {
		// rejected
		expect(err).toMatchObject({ err: { message: expect.anyting() } })
		// mock 호출되었는지 확인
		expect(mock).toHaveBeenCalld()
	});
})
```

## 현재 시각에 의존하는 테스트

현재 시각에 의존하는 로직이 테스트 대상에 포함되었다면, 테스트 결과가 실행 시각에 의존하게 된다.
이렇게 특정 시간대에는 `CI` 테스트 자동화가 실패하는 불안정한 테스트가 된다

이때 태스트 실행 환경의 현재 시각을 고정하면 언제 실행하더라도 같은 테스트 결과를 얻는다.

>[!info] greet.ts
```ts
export const greetByTime = async () => {
	const hour = new Date().getHours();
	if (hour < 12) {
		return "좋은 아침입니다"
	} else if (hour < 18) {
		return "식사는 하셨나요"
	}
	return "좋은 밤 되세요"
}
```

- `jest.userFakeTimers`: 제스트에 가짜 타이머를 사용하도록 지시하는 함수
- `jest.setSystemTime`: 가짜 타이머에서 사용할 현재 시각을 설정하는 함수
- `jest.useRealTimers`: 제스트에 실제 타이머를 사용하도록 지시하는 원상 복귀 함수

`beforeEach` 와 `afterEach` 에서 타이머를 교체하는 작업을 수행하여 테스트마다 가짜 타이머를 설정한다

```ts
describe("greetByTime", () => {
	beforeEach(() => {
		jest.useFakeTimers()
	})
	afterEach(() => {
		jest.useRealTimers()
	})

	test("hour < 12", () => {
		jest.setSystemTime(new Date(2023, 4, 23, 8, 0, 0))
		expect(greetByTime()).toBe("좋은 아침입니다")
	})

	test("hour < 18", () => {
		jest.setSystemTime(new Date(2023, 4, 23, 14, 0, 0))
		expect(greetByTime()).toBe("식사는 하셨나요")
	})

	test("hour >= 18", () => {
		jest.setSystemTime(new Date(2023, 4, 23, 19, 0, 0))
		expect(greetByTime()).toBe("좋은 밤 되세요")
	})
})
```



