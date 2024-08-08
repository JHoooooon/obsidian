
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

```
```





