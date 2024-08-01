
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

이는 `upgrade needed` 이벤트를 발생시키는 즉시 실행된다.

`upgrade needed` 이벤트에서 `DB` 객체를 가져오고, `customers` 라는 객체 저장소를 생성한다.
이를 통해 `passport_number` 를 객체에 대한 고유 키로 정의한다.

## 객체 저장소에 데이터 추가

```js
const request = window.indexedDB.open("my-database", 2);

request.onsuccess = (event) => {
	const db = event.target.result;
	const customerData = [
		{
			"passport_number": "6651",
			"first_name": "Tal",
			"last_name": "Ater",
		},
		{
			"passport_number": "7721",
			"first_name": "Archie",
			"last_name": "Stevens",
		},
	];

	const customerTransaction = db.transaction("customers", "readwrite");

	customerTransaction.onerror = (event) => {
		console.log(`Error: ${event.target.error}`);
	};

	const customerStore = customerTransaction.objectStore("customres");
	for (const i = 0; i < customerStore; i+=1) {
		customerStore.add(customerData[i]);
	}
};
```

`transaction` 에 쓰기 위해  `readwrite`  `transaction` 을 생성하고, 작업의 범위를 `customers` 객체 저장소로 지정한다.

>[!info] **db.transaction**<br><br>객체 저장소에 데이터를 추가하기 전에 새 `transaction` 을 시작해야 한다.<br>첫번째 인자로 `transaction` 범위를 전달한다.<br>두번째 인자는 `transaction` 을 제어하는 인수를 선택적으로 적용한다.<br><br>:LiAlertOctagon: 두번째 인자는 `readonly` 가 `default` 이고, `readwrite` 로 쓰기 작업을 하도록 적용가능하다 <br><br>`transaction` `scope` 는 트랜잭션이 영향을 줄수 있는 `Object store` 이름 혹은 `여러개의 Object store` 이름을 포함한 배열이다.<br><br>`IndexedDB` 는 `transaction` 범위를 지정하여 서로 다른 `transaction` 사이의 `race condition` 을 방지할수 있다.<br><br>만약, 두개 혹은 그 이상의 `readwrite` 트랙잭션 범위가 겹치는 경우, 각 `transaction` 은 `queue` 에 들어가 순차적으로 실행된다.<br><br>만약 서로 다른 트랜잭션 범위라면, 병렬로 실행된다.

>[!info] 트랜잭션은 트랜잭션 내에서 실행하는 작업은 마치 하나의 작업인것처럼 모두 성공하거나 모두 실패하는 `원자성`(`atomicity`) 가 보장된다.

## 객체 저장소에서 데이터 읽기

다음은 객체저장소에서 추가한 객체를 가져오는 방법에 대해 살펴본다.

데이터를 읽는 것에는 세가지 방법이 있다.

- 키를 사용하여 단일 객체를 요청
- `cursor` 를 사용하여 저장소의 모든 객체를 순회
- `index` 를 사용하어 더 작은 데이터 그룹으로 검색 

다음은 `key` 를 사용하여 단일 객체를 읽어온다.

```js
const request = window.indexedDB.open("my-database", 2);
request.addEventListener("success", (event) => {
	const db = event.target.result;
	const customerTransaction = db.trasaction("customers");
	const customerStore = customerTransaction.objectStore("customers");
	const request = customerStore.get("7727");
	request.addEventListener("success", (event) => {
		const customer = event.target.result;	
		console.log(`First name: ${customer.first_name}`);
		console.log(`last name: ${customer.last_name}`);
	});
});
```


