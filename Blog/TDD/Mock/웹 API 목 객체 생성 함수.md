
코드는 다음과 같다

>[!info] /fixtures.ts
```ts
import { Articles } from 'fetchers'

export const httpError: HttpError = {
	err: {
		message: "Internal server error",
	},
};

export const getMyArticlesData: Articles = [
	{
		id: "1",
		createdAt: "2022-07-19T22:38:41.005Z",
		tags: ["testing"],
		title: "타입스크립트를 사용한 테스트 작성법",
		body: "테스트 작성 시 타입스크립트를 사용하면 테스트의 유지 보수가 쉬워진다",
    },
    {
		id: "nextjs-link-component",
		createdAt: "2022-07-19T22:38:41.005Z",
		tags: ["nextjs"],
		title: "Next.js의 링크 컴포넌트",
		body: "Next.js는 화면을 이동할 때 링크 컴포넌트를 사용한다",
    },
    {
	    id: "react-component-testing-with-jest",
	    createdAt: "2022-07-19T22:38:41.005Z",
	    tags: ["testing", "react"],
	    title: "제스트로 시작하는 리액트 컴포넌트 테스트",
	    body: "제스트는 단위 테스트처럼 UI 컴포넌트를 테스트할 수 있다",
    },
]
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

export type Article = {
	id: string;
	createdAt: string;
	tags: string[];
	title: string;
	body: string;
}

export type Articles = {
	articles: Article[];
}

export const getMyArticles = async (): Promise<Profile> => {
	const res = await fetch("https://myapi.test.com/my/profile")
	const data = await res.json();
	if (!res.ok) {
		throw data
	}
	return data
}
```

>[!info] greet.ts
```ts
import { getMyProfile, getMyArticle } from "./fetchers"

export const getGreet = async () => {
	const data = await getMyProfile();
	if (!data.name) {
		return "Hello, annoymous user!"
	}
	
	return `Hello, ${data.name}`
}

export const getMyArticleLinksByCategory = async (category: string) {
	const data = await getMyArticle()
	const articles = data.filter(article => article.tag.includes(category));
	if (!articles.length) {
		return null;
	}
	return articles.map(article => ({
		title: article.title,
		link: `/article/${article.id}`
	}))
}
```

여기에서 `getMyArticleLinksByCategory` 함수를 테스트한다.

>[!info] greet.ts
```ts
import { getMyArticleLinksByCategory } from './greet'
import * as fetchers from "./fetchres"
import { getMyArticlesData, httpError } from "./fixture"

// mocking 객체를 생성하는 함수
const mockGetArticles = (status = 200) => {
	// jestOn 으로 spyobject 를 생성
	const getMyArticles = jest.jestOn(fetchres, "getMyArticles");

	// status 가 299 보다 크면 reject
	if (status > 299) {
		return getMyArticles.mockRejectedValueOnce(httpError)
	}

	// status 가 299 보다 작으면 resolve
	return getMyArticles.mockResolvedValueOnce(getMyArticlesData)
}

describe("getMyArticleLinksByCategory", () => {
	// tag 가 없으면 null
	test("article is null, if tags not includes tag", async () => {
		// spyon 객체 생성 함수 호출
		mockGetMyArticles()
		// playwright 인 tag 를 가진 articles 반환
		const data = await getMyArticleLinksByCategory("playwright")
		// playwright 라는 tag 를 가진 article 이 없어 null
		expect(data).toBeNull()
	})

	// tag 가 있으면 article 은 not null
	test("article is not null, if tags includes tag", async () => {
		// spyon 객체 생성 함수 호출
		mockGetMyArticles()
		// testing 인 tag 를 가진 articles 반환
		const data = await getMyArticleLinksByCategory("testing")
		// data 는 다음의 배열과 일치
		expect(data).toMatchObject([
			{
				link: "/articles/howto-testing-with-typescript",
				title: "타입스크립트를 사용한 테스트 작성법",
			},
			{
				link: "/articles/react-component-testing-with-jest",
				title:  "제스트로 시작하는 리액트 컴포넌트 테스트",
			}
		])
	})

	// 데이터취득 실패로 reject 테스트
	test("fetch failed", async () => {
		// spyon 객체 생성 함수 호출
		// reject
		mockgetMyArticles(500)
		// error 단언문 생성
		await getMyArticleLinksByCategory("testing").catch(err => {
			// reject 된 error object 테스트
			expect(err).toMatchObject({
				err: {
					message: "Internal server error"
				}
			})
		})
	})

})
```

