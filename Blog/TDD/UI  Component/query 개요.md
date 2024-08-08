
`testing-library/react` 를 사용하며, 이때 `react` 의 요소를 `rendering` 하려면, `render` 메서드를 사용한다.

렌더링된 요소중 특정 `DOM` 요소를 가져오려면 `screen.getByText` 를 사용한다
이는 `screen` 에서 `Text` 와 일치하는 값이 있다면 해당 요소를 가져오는 것이다.

다음을 보자query

>[!info] Form.tsx
```tsx
type Props = {
    name: string;
    onSubmit?: (e: React.FormEvent<HTMLFormElement>) => void;
};

export const Form = ({ name, onSubmit }: Props) => {
    return (
        <form
            onSubmit={(e) => {
                e.preventDefault();
                onSubmit?.(e);
            }}
        >
            <h2>계정 정보</h2>
            <p>{name}</p>
            <div>
                <button>수정</button>
            </div>
        </form>
    );
};
```

>[!info] Form.test.tsx
```tsx
import { render, screen } from '@testing-library/react';
import { Form } from './Form';

test('이름을 표시', () => {
    render(<Form name="taro" />);
    expect(screen.getByText('taro')).toBeInTheDocument();
});
```

이는 `taro` 라는 `text` 를 가진 요소를 가져온다.
이와 동시에 `expect` 단언문을 사용하여, `document` 안에 해당 요소가 있는지 검증한다.

## DOM 요소를 `Role` 로 가져오기

모든 요소마다 `Role` 이 존재한다.
이는 웹 접근성에서 식별하기 요소가 어떠한 역할을 하는지 알기위해 사용된다.

`<button>`  요소는 `"button"` 이라는 `Role` 이 부여되어 있다

```tsx
test("버튼을 표시", () => {
	render(<Form name="taro" />)
	expect(screen.getByRole("button").toBeInTheDocument())
})
```

이는 `<button>` 요소를 가져온다.

`<h2>` 요소가 있다면, 해당 요소를 가져오는 `Role` 은 `heading` 이다.
`heading` 은 `h1 - h6` 까지의 요소전부를 포함한다

```tsx
test("h2 heading 표시", () => {
	render(<Form name="taro") />)
	const heading = screen.getByRole('heading') // <h2> 요소를 가져온다.
	expect(heading).toBeInTheDocument() // true
})
```

만약, 문자가 포함되었는지 확인한다면, `toHaveTextContent` 를 사용한다.

```tsx
test("heading 을 포함", () => {
	render(<Form name="taro") />)
	const heading = screen.getByRole('heading')  // <h2> 요소를 가져온다.
	expect(heading).toBeInTheDocument() // true
	expect(heading).toHaveTextContent("계정 정보") // true
})
```

>[!info] `getByRole` 은 `button` 요소 $1$ 개를 반환하며, 여러 `button` 이 필요하다면, `getAllByRole` 을 사용한다.

이는 웹 접근성을 사용하여, 요소에 접근하는것으로 볼수 있다.
[html-aria role](https://www.w3.org/TR/html-aria/#docconformance) 에 모든 `role` 에 대한 설명이 나와있다.

## 아이템 목록 UI 컴포넌트 테스트

>[!info] ArticleList.ts
```ts
import { ItemProps } from './index.type';

export const articleList: ItemProps = [
    {
        id: 'howto-testing-with-typescript',
        title: '타입스크립트를 사용한 테스트 작성법',
        body: '테스트 작성 시 타입스크립트를 사용하면 테스트의 유지 보수가 쉬워진다',
    },
    {
        id: 'nextjs-link-component',
        title: 'Next.js의 링크 컴포넌트',
        body: 'Next.js는 화면을 이동할 때 링크 컴포넌트를 사용한다',
    },
    {
        id: 'react-component-testing-with-jest',
        title: '제스트로 시작하는 리액트 컴포넌트 테스트',
        body: '제스트는 단위 테스트처럼 UI 컴포넌트를 테스트할 수 있다',
    },
];
```

>[!info] ArticleList.tsx
```tsx
import { ItemProps } from '../../fixtures/index.type';
import ArticleListItem from './ArticleListItem';

type Props = {
    items: ItemProps;
};

const ArticleList = ({ items }: Props) => {
    return (
        <div>
            <h2>기사 목록</h2>
            {items.length ? (
                <ul>
                    {items.map((item) => (
                        <ArticleListItem {...item} key={item.id} />
                    ))}
                </ul>
            ) : (
                <p>게시된 기사가 없습니다.</p>
            )}
        </div>
    );
};

export default ArticleList;
```

>[!info] ArticleList.test.tsx
```tsx
import { render, screen } from '@testing-library/react';
import ArticleList from './ArticleList';
import { articleList } from '../../fixtures/index';

test('itmes 수만큼 목록 표시', () => {
    render(<ArticleList items={articleList} />);
    const listItmes = screen.getAllByRole('listitem');
    expect(listItmes).toHaveLength(3); // 3
});

test('목록을 표시', () => {
    render(<ArticleList items={articleList} />);
    const list = screen.getByRole('list');
    expect(list).toBeInTheDocument(); // true
});
```

위는 `listItmes` 의 `length` 가 $3$ 개임을 볼수 있다.

### within 함수로 범위 좁히기

큰 컴포넌트를 다룰때, `test` 대상이 아닌 `listitem` 도 `getAllByRole` 의 반환값에 포함될수 있다.

>[!info] `listitem` 은 모든 `li` 를 선택한다. 그러므로 `test` 대상이 아닌 모든 `li` 를 선택하므로 이는 문제가된다.

이때 얻은 `list` 노드로 범위를 좁혀, 여기에 포함된 `listitem` 요소의 숫자를 검증해야 한다.




