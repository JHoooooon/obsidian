다음은 웹 `API` 를 테스트를 `Stub` 으로 교체한다.

>[!info] /fixtures.ts
```ts
export const httpError: HttpError = {
	err: {
		message: "Internal server error",
	},
};
```

>[!info] fetchers.ts
```ts
export type Profile {
	id: string;
	name?: string;
	age?: number;
	email: string;
}

export const getMyProfile = async (): Promise<Profile> => {
	const res = await fetch("https://myapi.test.com/my/profile"))
	const data = await res.json();
	if (!res.ok) {
		throw data; // server 의 error 내용
					// { err: message: "Internal server error" }
	}
	return data; // Profile type 의 반환값
}
```

>[!info] greet.ts
```ts
import { getMyProfile } from "./fetchers"

export const getGreet = async () => {
	const data = await getMyProfile();
	if (!data.name) {
		return "Hello, annoymous user!"
	}
	
	return `Hello, ${data.name}`
}
```

`Test` 를 구현한다.
여기에서 `fetchers` 는 `Stub` 으로 대체하고, `getGreet` 는 `spyOn` 으로 대체한다.

>[!info] greet.test.ts
```ts
import { getGreet } from "./greet"
import * as Fetchers from "./fetchers"
import { httpError } from "./fixtures"

jest.mock("./fetchers")
describe("getGreet", () => {
	test("data fetch successed: not exists name", async () => {
		// 데이터 취득이 성공했을때 `resolve` 응답을 기대하는 객체를 
		// mockResolvedValueOnce 에 지정
		jest.spyOn(Fetchers, "getMyProfile").mockResolvedValueOnce({
			id: "xxxx-123456",
			email: "test@gmail.com",
		})
		await expect(getGreet()).resolves.toBe("Hello, anonymous user!")
	})

	test("data fetch successed: exists name", async () => {
		// 데이터 취득이 성공했을때 `resolve` 응답을 기대하는 객체를 
		// mockResolvedValueOnce 에 지정
		jest.spyOn(Fetchers, "getMyProfile").mockResolvedValueOnce({
			id: "xxxx-123456",
			email: "test@gmail.com",
			name: "test"
		})
		await expect(getGreet()).resolves.toBe("Hello, test")
	})

	test("data fetch failed", async () => {
		// 데이터 취득 실패시, `reject` 응답을 기대하는 객체를
		// mockRejectedValueOnce 에 지정 	
		jest.spyOn(Fetchers, "getMyProfile").mockRejectedValueOnce(httpError)
		try {
			await getGreet()
		} catch (err) {
			expect(err).toMatchObject(httpError)
		}
	})
})
```

여기서 `"data fetch failed"` 에서 `try...catch` 문을 사용했지만 다음처럼도 사용가능하다

```ts
test("data fetch failed", async () => {
	jest.spyOn(Fetchers, "getMyProfile").mockRejectedValueOnce(httpError)
	// try...catch 대신 rejects 를 사용한 방법
	await expect(getGreet()).rejects.toMatchObject({
		err: { message: "Internal server error" }
	})
})
```

예외 발생하는지 확인하기를 원한다면, `expect.assertions` 를 사용한다
이는 `expect` 가 $1$ 번은 일어난다고 단언한다.

>[!info] `throw Error` 같은경우 함수에서 `return` 하는 값이 아니라 `expect` 에서 평가하지 않고 끝날수 있다. 이러한 경우를 보장하기 위해 `expect` 가 반드시 $1$ 번은 평가한다고 보장해야 `True` 가 된다.

```ts
test("data fetch failed", async () => {
	// 데이터 취득 실패시 오류가 발생한 데이터와 함께 예외를 throw
	expect.assertions(1)
	jest.spyOn(Fetchers, "getMyProfile").mockRejectedValueOnce(httpError)
	// try...catch 대신 rejects 를 사용한 방법
		try {
			await getGreet()
		} catch (err) {
			expect(err).toMatchObject(httpError)
		}
})
```