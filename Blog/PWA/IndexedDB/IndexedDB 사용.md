
`IndexedDB` 가 다소 헷갈리기로 악명이 높다.
핵심 원리를 재빨리 이해하고, 짧은 시간내에 정확한 결과를 얻으려면 직접 사용해보는것이 가장 좋은 방법이다.

## Open Database Connection 

```js
const request = window.indexedDB.open("my-database", 1);

request.onerror = (event) => {
	console.log(`Database error: ${event.target.error}`);
};

request.onsuccess = (event) => {
	const db = event.target.result;
	console.log(`Database ${db}`);
	console.log(`Object store names:${db.objectStoreNames}`);
};
```

`window.indexedDB.open` 호출은 `database connection` 을 반환하지 않으며, **커넥션을 열기 위한 `IDBRequest` 객체를 반환한다.**

이 객체를 통해, 해당 요청에 대한 `event` (`success` 혹은 `error` 이벤트) 를 수신할수 있다.

## 데이터베이스 버전 번호 관리/객체저장소 변경

`service worker` 와 마찬가지로, `IndexedDB` 도 `DB` 버전을 가진다.
`Object Store` 의 추가, 변경, 삭제와 같이 데이터베이스 구조를 변경할때마다 새로운 버전을 생성해야 한다.

`indexedDB.open()` 의 두번째 인수로 전달되는 `version number` 를 증가시켜, 새 데이터베이스 버전을 만들 수 있다.

이를 처리할수 있도록 현재 자신의 `version number` 보다 크다면, `upgrade needed` 이벤트가 발생한다.

이때, 기존의 `database` 를 수정하려면 이 `upgrade needed` 이벤트를 활용한다.

```js
request.onupgradeneeded = (event) => {
	const db = event.target.result;

	db.createObjectStore("customers", {
		keyPath: "passport_number",
	});
};
```

