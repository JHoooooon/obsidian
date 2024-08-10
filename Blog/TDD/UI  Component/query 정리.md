
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

### Testing-library/react 의 query 우선순위
 
`testing-library/react` 에서의 `query` 는 우선순위가 존재한다.
이는 요소를 얻기 위한 원칙인데, 웹 접근성과 연관이 깊다.

웹접근성은 청각, 시각, 인지, 신경, 신체 , 언어에 대한 장애를 겪는 사람들 역시 이러한 점을 극복하고 웹에 대한 접근성을 높히는데 있다. 

>[!info] 웹의 힘은 보편성에 있습니다.<br>장애에 상관없이 모두가 접근할 수 있다는 것이 가장 중요한 부분입니다.<br><br>`팀 버너스리, W3C 디렉터 및 Wrold Wide Web의 창시자`

`testing-library` 는 `모든 사용자 입력을 제약없이 재현` 이라는 목적을 가지고 설계되었다.

**이는 신체적, 정신적 특성에 따른 차이 없이 접근할수 있는 쿼리를 의미한다.**

이러한 접근성을 고려해서, 테스팅에 대한 쿼리를 웹 접근성을 기반으로 한 `query` 가 우선시되어야 한다.

다음은 이러한 우선순위를 보여준다.

1. **모두가 접근 가능한 쿼리**<br>신체적, 정신적 특성에 따른 차이 없이 접근할 수 있는 쿼리를 의미.<br>`screen reader` 등의 보조 기기로 인지한것과 동일한것을 증명한다.<br><br>[`ByRole`](https://testing-library.com/docs/queries/byrole): [HTML-Aria](https://www.w3.org/TR/html-aria/#docconformance)에 명시된 `ROLE` 로 `query`<br><br>[`ByLabelText`](https://testing-library.com/docs/queries/bylabeltext): 주어진 `TextMatch` 와 연관된 `label` 로 `element` `query`<br>`TextMatch` 는 `label` 에 주어진 이름이다.<br><br>[`ByPlaceholderText`](https://testing-library.com/docs/queries/byplaceholdertext): 주어진 `placeholder` 에 맞는 `element` `query`<br><br>[`ByDisplayValue`](https://testing-library.com/docs/queries/bydisplayvalue): `textfield`, `input`, `select` 요소에서 표시된 `value` 로 `query`<br>

2. **시맨틱 쿼리**<br>공식 표준에 기반한 속성을 사용하는 쿼리를 의미<br>시맨틱 쿼리를 사용할때는 브라우저나 보조 기기에따라 상당히 다른 결과가 나올수있다.<br><br>[`ByAltText`](https://testing-library.com/docs/queries/byalttext): `Alt` 에 주어진 `text` 를 기반으로 `element` `query`<br><br>[`ByTitle`](https://testing-library.com/docs/queries/bytitle): `element` 가 가진 `textContent` 를 기반으로 `element` `query`<br><br>[`ByText`](https://testing-library.com/docs/queries/bytext): 주어진 `TextMatch` 와 매칭되는 `Text Node` 를 가진 `element` 를 `query` 

3. **테스트 ID**<br><br>테스트용으로 할당된 식별자를 의미<br>역할이나 문자 컨텐츠를 활용한 쿼리를 사용할수 없거나, 의도적으로 의미 부여를 피하고 싶을때 사용<br><br>[`ByTextId`](https://testing-library.com/docs/queries/bytestid):  `container.querySelector("[data-testid="${yourId}"]` 와 같은 의미로, 식별자를 `query`

### query Type

중요한 부분은 `query` 에 대한 타입에 따라 붙는 `prefix` 가 달라진다.
이는 공통적으로 사용되는 `query` `convention` 으로 사용된다. 
`query type` 은 다음과 같다

>[!info] [type of query](https://testing-library.com/docs/queries/about#types-of-queries) 에서 확인할수 있다.

#### Single Element

**getBy...** : `matching` 되는 `element` 를 반환한다.<br>만약 `matching` 되는 `element` 가 없거나 $2$ 이상의 `element` 가 있다면,<br>`error` 를 `throw` 한다.

**queryBy...**: `matching` 되는 `element` 를 반환한다.<br>만약, `matching` 되는 `element` 가 없다면 `NULL` 을 반환한다.<br>하지만, $2$ 이상의 `element` 가 있다면, `error` 를 `throw` 한다.

**findBy...**: `Promise`  를 반환한다.<br> 주어진 `query` 에 `matching` 되는 `element` 가 있다면`resolve` 된다.<br> 주어진 `query` 에 `matching` 되는 `element` 가 없거나, `default` `timeout` 이 지나면, `reject` 된다.<br><br>`default` `timeout`  은 $1000ms$  이다. 

---
#### Multiple element

**getAllBy...**: `query` 에 `matching` 되는 모든 `element` 를 배열로 반환한다.<br>주어진 `query` 에 `matching` 되는 `element` 가 없다면 `error` 를 `throw` 한다.

**queryAllBy...**: `query` 에 `matching` 되는 모든 `element` 를 배열로 반환한다.<br>주어진 `query` 에 `matching` 되는 `element` 가 없다면 `[]`(`empty array`) 를 반환한다.

**findAllBy...**: `Primise` 를 반환한다.<br>주어진 `query` 에 `matching` 되는 `element`  가 있다면, `resolve` 되며 모든 `element` 를 담은 `Array` 를 반환한다.<br>주어진 `query` 에 `matching` 되는 `element` 가 없거나, `default` `timeout` 이 지나면, `reject` 된다.<br><br>`default` `timeout` 은 $1000ms$ 이다. <br><br>여기서 중요한 부분은 `reject` 된다는것은 `error` 를 `throw` 한다는 뜻이며, 빈배열을 반환하지 않는다.

---
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

## within 함수로 범위 좁히기

큰 컴포넌트를 다룰때, `test` 대상이 아닌 `listitem` 도 `getAllByRole` 의 반환값에 포함될수 있다.

>[!info] `listitem` 은 모든 `li` 를 선택한다. 그러므로 `test` 대상이 아닌 모든 `li` 를 선택하므로 이는 문제가된다.

이때 얻은 `list` 노드로 범위를 좁혀, 여기에 포함된 `listitem` 요소의 숫자를 검증해야 한다.
이때 사용하는 함수가 `within` 이다.

```tsx
test('itmes 수만큼 목록 표시', () => {
    render(<ArticleList items={articleList} />);
    const list = screen.getAllByRole('list');
    expect(list).toBeInTheDocument();
	expect(within(list)).getAllByRole('listitem').toHaveLength(3) // 3
});
```

##  목록에 표시할 내용이 없는 상황에서 테스트

목록을 표시하지 않으면, `"게시된 기사가 없습니다.` 라는 문구가 나온다.
이를 검증하는 테스트는 다음과 같다

```tsx
test('목록이 없음', () => {
	render(<ArticleList items={[]} />)
	const list = screen.getAllByRole('list')
	expect(list).toBeNull() // true
	expect(screen.getByText('게시된 기사가 없습니다.')).toBeInTheDocument() // true
})
```

## 개별 아이템 컴포넌트 테스트

만약 다음과 같은 개별 컴포넌트가 있다고 가정하자

>[!info] ArticleListItem.tsx
```tsx
export type ItemProp {
	id: string;
	title: string;
	body: string;
}

export const ArticleListItmem = ({ id, title, body }: ItemProps) => {
	return (
		<li>
			<h3>{title}</h3>
			<p>{body}</p>
			<a href={`/articles/${id}`}>더 알아보기</a>
		</li>
	)
}
```

그리고, 이러한 `listItem` 을 테스트한다.
테스트는 `id` `attrubute` 를 찾는다.

이때 해당 `attribute` 를 찾기위한 `toHaveAttribute` 메서드를 사용한다.

>[!info] ArticleListItem.test.tsx
```tsx
import {render, screen} from "@testing-library/react";
import { ArticleListItem } from "./ArticleListItem";

const item: ItemProp = {
	id: "howto-testing-with-typescript",
	title: "타입스크립트를 위한 테스트 방법",
	body: "테스트..."
};

test("링크에 id 로 만든 URL 표시", () => {
	render(<ArticleListItem {...item}/>);
	expect(screen.getRoleBy("link", { 
		name: "더 알아보기" 
	})).toHaveAttribute('href', `/articles/${item.id}`)
});
```

## 인터렉티브 UI 컴포넌트 테스트

다음은 `form` 컴포넌트를 테스트한다.
이는 이용약관에 동의하면 `button` 이 활성화 된다.

>[!info] Agreement.tsx
```tsx 
type Props = {
	onChange?: React.ChangeEventHandler<HTMLInputElement>;
};

export const Agreement = ({ onChange }: Props) => {
	return (
		<fieldset>
			<legend>이용 약관 동의</legend>
			<label>
				<label htmlFor="agreement">동의하기</label>
				<input id="agreement" type="checkbox" onChange={onChange} />
				서비스&nbsp;<a href="/terms">이용 약관</a>을 확인했으며 이에 동의합니다.
			</label>
		</fieldset>
	)
};
```

>[!info] InputAccount.tsx
```tsx
export const InputAccount= () => {
	return (
		<fieldset>
			<legend>계정정보 입력</legend>
			<div>
				<label>
					메일주소
					<input type="text" placeholder="example@test.com" />
				</label>
			</div>
			<div>
				비밀번호
				<input type="password" placeholder="8자 이상" />
			</div>
		</fieldset>
	)
}
```

>[!info] Form.tsx
```tsx

import { useState, useId } from 'react';
import { InputAccount } from './InputAccount'; 
import { Agreement } from './Agreement';

const [checked, setChecked] = useState(false)
const headingId = useId()

export const Form = ()=> {
	<form aria-labelledby={headingId}>
		<h2 id={headingId}>신규 계정 등록</h2>
		<InputAccount />
		<Agreement />
		<div>
			<button disabled={!checked}>회원가입</button>
		</div>
	</form>
}
```

이를 테스팅 한다.

### Agreement

>[!info] Agreement.tsx
```tsx 
type Props = {
	onChange?: React.ChangeEventHandler<HTMLInputElement>;
};

export const Agreement = ({ onChange }: Props) => {
	return (
		<fieldset>
			<legend>이용 약관 동의</legend>
			<label>
				<input type="checkbox" onChange={onChange} />
				서비스&nbsp;<a href="/terms">이용 약관</a>을 확인했으며 이에 동의합니다.
			</label>
		</fieldset>
	)
};
```

다음은 `fieldset`  이 존재하는지 확인하는 테스트이다

>[!info] Agreement.test.tsx
```tsx
import { render, screen } from '@testing-library/react';
import { Agreement } from './Agreement'

test("fieldset", () => {
	render(<Agreement />)
	expect(
		screen.getByRole('group', { name: "이용 약관 동의" })
	).toBeInTheDocument();
})
```

[group role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/group_role) 에서 보면, `fieldset`  의 `role` 은 `group` 이라고 명시한다.

>[!info] `fieldset` 요소는 `group` 이라는 암묵적 역할을 한다.<br>`legend` 는 `fieldset` 의 하위요소로서 그룹에 제목을 붙이는데 사용된다.

이는 기능적으로 연관된 `item`  들의 논리적인 모음으로도 사용된다.
`role` 은 `items` 를 묶는 `ul`, `tree` 구조로 이루어진 항목에 사용된다.

```html
<div id="tree1" role="tree" tabindex="-1">
  <div
    id="animals"
    class="groupHeader"
    role="presentation"
    aria-owns="animalGroup"
    aria-expanded="true">
    <img role="presentation" tabindex="-1" src="images/treeExpanded.gif" />
    <span role="treeitem" tabindex="0">Animals</span>
  </div>
  <div id="animalGroup" role="group">
    <div id="birds" role="treeitem">
      <span tabindex="-1">Birds</span>
    </div>
    <div
      id="cats"
      class="groupHeader"
      role="presentation"
      aria-owns="catGroup"
      aria-expanded="false">
      <img role="presentation" tabindex="-1" src="images/treeContracted.gif" />
      <span role="treeitem" tabindex="0">Cats</span>
    </div>
    <div id="catGroup" role="group">
      <div id="siamese" role="treeitem">
        <span tabindex="-1">Siamese</span>
      </div>
      <div id="tabby" role="treeitem">
        <span tabindex="-1">Tabby</span>
      </div>
    </div>
  </div>
</div>
```

```html
<div role="menu">
  <ul role="group">
    <li role="menuitem">Inbox</li>
    <li role="menuitem">Archive</li>
    <li role="menuitem">Trash</li>
  </ul>
  <ul role="group">
    <li role="menuitem">Custom Folder 1</li>
    <li role="menuitem">Custom Folder 2</li>
    <li role="menuitem">Custom Folder 3</li>
  </ul>
  <ul role="group">
    <li role="menuitem">New Folder</li>
  </ul>
</div>
```

책에서는 다음처럼 역할을 가지지 않은 `div` 로 마크업하지 않기를 강조한다.
이는 접근성에서 역할이 없으므로, 그룹으로 식별하기 어렵다.

>[!warning] `div` 로 마크업하는건 `role` 식별이 어렵다
```tsx
type Props = {
	onChange?: React.ChangeEventHandler<HTMLInputElement>;
};

export const Agreement = ({ onChange }: Props) => {
	return (
		<div>
			<legend>이용 약관 동의</legend>
			<label>
				<input type="checkbox" onChange={onChange} />
				서비스&nbsp;<a href="/terms">이용 약관</a>을 확인했으며 이에 동의합니다.
			</label>
		</div>
	)
};
```

>[!info] 이처럼 `UI 컴포넌트 테스트` 작성시 웹 접근성을 고려하며 테스팅하게 된다.

### 체크박스의 초기 상태 검증

다음은, `checkbox` 가 `checked` 되었는지 확인한다.

```tsx
test("체크 박스 체크되어 있지 않습니다.", () => {
	render(<Agreement />)
	expect(screen.getByRole("checkbox")).not.toBeChecked()
})
```

>[!info] [JEST DOM toBeChecked](https://github.com/testing-library/jest-dom?tab=readme-ov-file#tobechecked)
>
> 주어진 `element` 가 `checked` 되었는지 확인하는 메서드
> `checked` 되었다면 `ture`, 아니면 `false` 이다.

### 계정 정보 입력 컴포넌트 테스트

```tsx
export const InputAccount= () => {
	return (
		<fieldset>
			<legend>계정정보 입력</legend>
			<div>
				<label>
					메일주소
					<input type="text" placeholder="example@test.com" />
				</label>
			</div>
			<div>
				<label htmlFor="pass">비밀번호</label>
				<input id="pass" type="password" placeholder="8자이상" />
			</div>
		</fieldset>
	)
}
```

`Input` 에 입력을 하는 테스트를 진행한다
이를 위해 [user-event](https://testing-library.com/docs/user-event/intro) 를 `import` 해서 사용한다.

>[!info] `user-event` 는 `user` 와의 상호작용을 위해 만들어진 `libarary` 이다.

`userEvent.setup()` 으로 `API` 를 호출할 `user` 인스턴스를 생성하며,
`user` 로 입력하여 테스트를 생성한다.

```tsx
import { render, screen } from '@testing-libarary/react'
import userEvent from '@testing-libarary/user-event'
import { InputAccount } from './IntpuAccount'

const user = userEvent.setup();

test("메일 주소 입력", async () => {
	render(<InputAccount />)
	// 메일주소 input 쿼리
	const textBox = screen.getByRole('textbox', { name: '메일주소' })
	// 입력값
	const value = 'test@mail.com'

	// user.type 을 사용하여 textBox 에 value 값 입력
	await user.type(textBox, value)

	// screen 상에 value 값이 있는 요소가 있는지 검증
	expect(screen.getByDisplayValue(value)).toBeInTheDocument()
})
```

### 비밀번호 입력

이는 똑같이 처리 가능하지만, 약간 다르다.
`input type=password` 는 `textbox` 로 취급하지 않기 때문이다.

이는 몇가지 이유가 존재한다.

1. password 입력 필드는 일반 텍스트 상자와 다른 보안 특성을 가집니다. 입력된 문자가 asterisks(\*)나 dots(•)로 마스킹되어 표시된다.

2. 접근성: 스크린 리더와 같은 보조 기술이 이 필드를 일반 텍스트 상자와 다르게 처리할 수 있다 

>[!warning] ARIA  명세에서는 `password` `no-role` 인것을 볼수 있다. 

이렇게 명세가 되어있지 않기때문에, `browser` 마다 다른 접근법을 제시할수 있기에, 다른 방법으로 처리해야한다.

대략적으로 $2$ 가지 방법이 제시된다.

>[!info] InputAccount.test.tsx
```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import InputAccount from '.';

const user = userEvent.setup();

test('메일주소 입력', () => {
    render(<InputAccount />);
    const textbox = screen.getByRole('textbox', { name: '메일주소' });
    expect(textbox).toBeInTheDocument();
});

test('비밀번호 존재 확인(plaeholder 로 쿼리)', () => {
    render(<InputAccount />);
    const passwordBox = screen.getByPlaceholderText('8자이상');
    expect(textbox).toBeInTheDocument();
});

test('비밀번호 존재 확인(label text 로 쿼리)', () => {
    render(<InputAccount />);
    const passwordBox = screen.getByLabelText('비밀번호');
    expect(textbox).toBeInTheDocument();
});
```

이제 입력을 확인한다

```tsx
test('비밀번호 존재 확인(plaeholder 로 쿼리)', async () => {
    render(<InputAccount />);
    const passwordBox = screen.getByPlaceholderText('8자이상');
	const value = "abc12345"
    expect(textbox).toBeInTheDocument();

	await user.type(textbox, value)

	expect(screen.getByDisplayValue(value)).toBeInTheDocument()
});
```

제대로 처리되는것을 확인할수 있다.

### 회원가입 폼 테스트

>[!info] Form.tsx
```tsx

import { useState, useId } from 'react';
import { InputAccount } from './InputAccount'; 
import { Agreement } from './Agreement';

const [checked, setChecked] = useState(false)
const headingId = useId()

export const Form = ()=> {
	<form aria-labelledby={headingId}>
		<h2 id={headingId}>신규 계정 등록</h2>
		<InputAccount />
		<Agreement />
		<div>
			<button disabled={!checked}>회원가입하기</button>
		</div>
	</form>
}
```

```tsx
test('회원가입 버튼 비활성화', () => {
    render(<Form />);
    const button = screen.getByRole('button', { name: '회원가입하기' });

    // button 있는지 확인
    expect(button).toBeInTheDocument();

    // button 이 disabled 인지 확인
    expect(button).toBeDisabled();
});
```

```tsx
test('회원가입 버튼 활성화', async () => {
    render(<Form />);
    const button = screen.getByRole('button', { name: '회원가입하기' });
    const chkBox = screen.getByRole('checkbox', { name: '이용 약관' });

    // button 있는지 검증
    expect(button).toBeInTheDocument();
    // chkBox 있는지 검증
    expect(chkBox).toBeInTheDocument();

    // button disabled 검증
    expect(button).toBeDisabled();

    // 체크박스 클릭
    await user.click(chkBox);

    // button enabled 검증
    expect(button).toBeEnabled();
});
```

### Form 의 접근 가능한 이름

`Form` 의 이름역할을 하는 `heading` 이 존재한다.
이는 `<h2>` 요소로써  `aria-labelledby` 라는 속성으로 접근가능한 이름으로 사용할수 있다.

```tsx
import { useState, useId } from 'react';
import { InputAccount } from './InputAccount'; 
import { Agreement } from './Agreement';

const [checked, setChecked] = useState(false)
const headingId = useId()

export const Form = ()=> {
	<form aria-labelledby={headingId}>
		<h2 id={headingId}>신규 계정 등록</h2>
		...
	</form>
}
```

이는 다음처럼 테스트 가능하다.

```tsx
test("form 접근가능한 이름", () => {
	render(<Form />);
	expect(
		screen.getByRole("form", { name: "신규 계정 등록" })
	).toBeInTheDocument();
});
```

## 유틸리티 함수를 활용한 테스트

>[!info] DeliveryForm.tsx
```tsx
import { useState } from 'react';
import ContactNumber from './ContactNumber';
import RegisterDeliveryAddress from './RegisterDeliveryAddress';
import DeliveryAddress from './DeliveryAddress';
import PastDeliveryAddress from './PastDeliveryAddress';

export type AddressOption = React.ComponentProps<'option'> & { id: string };

export interface Props {
    deliveryAddresses?: AddressOption[];
    onSubmit?: (e: React.FormEvent<HTMLFormElement>) => void;
}

const DeliveryForm = (props: Props) => {
    const [registerNew, setRegisterNew] = useState<boolean | undefined>(
        undefined
    );

    return (
        <form onSubmit={props.onSubmit}>
            <h1>배송지 정보 입력</h1>
            <ContactNumber />
            {props.deliveryAddresses?.length ? (
                <>
                    <RegisterDeliveryAddress onChange={setRegisterNew} />
                    {registerNew ? (
                        <DeliveryAddress title="새로운 배송지" />
                    ) : (
                        <PastDeliveryAddress
                            disabled={registerNew === undefined}
                            options={props.deliveryAddresses}
                        />
                    )}
                </>
            ) : (
                <DeliveryAddress />
            )}
        </form>
    );
};

export default DeliveryForm;
```

위의 `DeliveryForm`  을 사용하여 `Testing` 한다.

### 폼 입력을 함수화

화면 분기가 있는경우 폼 입력 테스트는 여러번 동일한 인터렉션을 작성한다.
이렇게 반복적인 호출해야 하는 인터렉션을 하나의 함수로 처리하면 처리하기 편해진다.

>[!info] Form.text.tsx
```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import DeliveryForm from '.';

const user = userEvent.setup();

// contactNumber input 입력 함수
const inputContactNumber = async (
    inputValues = {
        name: '배연수',
        phoneNumber: '000-0000-0000',
    }
) => {
    const phone = screen.getByRole('textbox', { name: '전화번호' });
    const name = screen.getByRole('textbox', { name: '이름' });

    await user.type(phone, inputValues.phoneNumber);
    await user.type(name, inputValues.name);

    return inputValues;
};

// deliveryAddress input 입력 함수
const inputDeliveryAddress = async (
    inputValues = {
        postalCode: '16397',
        prefectures: '경기도',
        municipalities: '수원시 권선구',
        streetNumber: '매곡로 67',
    }
) => {
    const postalCode = screen.getByRole('textbox', { name: '우편번호' });
    const prefectures = screen.getByRole('textbox', { name: '시/도' });
    const municipalities = screen.getByRole('textbox', { name: '시/군/구' });
    const streetNumber = screen.getByRole('textbox', { name: '도로명' });

    await user.type(postalCode, inputValues.postalCode);
    await user.type(prefectures, inputValues.prefectures);
    await user.type(municipalities, inputValues.municipalities);
    await user.type(streetNumber, inputValues.streetNumber);

    return inputValues;
};

// mockFn 을 반환하는 함수
// onsubmit 함수실행시 mockFn 에 formData 를 넣어 호출한다.
const mockHandleSubmit = () => {
    const mockFn = jest.fn();
    const onSubmit = (e: React.FormEvent<HTMLFormElement>) => {
        const formData = new FormData(e.currentTarget);
        const data: { [k: string]: unknown } = {};
        formData.forEach((v, k) => {
            console.log(v, k);
            data[k] = v;
        });
        mockFn(data);
    };

    return [mockFn, onSubmit] as const;
};

// test
describe('이전 배송지가 없는경우', () => {
	// 배송지 입력란 존재하는지 검증
    test('배송지 입력란이 존재한다.', async () => {
        render(<DeliveryForm />);

        const contractNumber = screen.getByRole('group', { name: '연락처' });
        const deliveryAddress = screen.getByRole('group', { name: '배송지' });

        expect(contractNumber).toBeInTheDocument();
        expect(deliveryAddress).toBeInTheDocument();
    });

	// 폼 제출시 입력내용 확인 검증
    test('폼을 제출하면 입력 내용을 전달받는다', async () => {
		// mockHandleSubmit 함수에서 mockFn 과 onSubmit 구조분해할당
        const [mockFn, onSubmit] = mockHandleSubmit();
		// 구조분해 할당된 onSubmit 을 DeliveryForm 에 전달
        render(<DeliveryForm onSubmit={onSubmit} />);

		// inputCntactNumber 함수 호춯
        const contactNumberValue = await inputContactNumber();
		// deliveryAddressValue 함수 호춯
        const deliveryAddressValue = await inputDeliveryAddress();
		// button 쿼리
        const btn = screen.getByRole('button', { name: '주문내용 확인' });

		// button 클릭
        await user.click(btn);

		// mockFn 이 호출되었는지 검증
        expect(mockFn).toBeCalled();
		// mockFn 이 contactNumberValue 와 deliveryAddressValue 와
		// 함께 호출되었는지 검증
        expect(mockFn).toHaveBeenCalledWith({
            ...contactNumberValue,
            ...deliveryAddressValue,
        });
    });
});

```

### 이전 배송지가 있는 경우 테스트

```tsx
```