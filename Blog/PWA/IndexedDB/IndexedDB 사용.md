
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

### GET 으로 단일객체 요청

다음은 `key` 를 사용하여 단일 객체를 읽어온다.

```js
const request = window.indexedDB.open("my-database", 2);
request.addEventListener("success", (event) => {
	const db = event.target.result;
	const customerTransaction = db.transaction("customers");
	const customerStore = customerTransaction.objectStore("customers");
	const request = customerStore.get("7727");
	request.addEventListener("success", (event) => {
		const customer = event.target.result;	
		console.log(`First name: ${customer.first_name}`);
		console.log(`last name: ${customer.last_name}`);
	});
});
```

이는 다음처럼 처리할수 있다.

```js
const request = window.indexedDB.open("my-database", 2);

request.addEventListener("success", (event) => {
	const db = event.target.result;

	db.transaction("customers")
		.objectStore("customers")
		.get("7721")
		.addEventListener("success", (event) => {
			const customer = event.target.result;		
			console.log(`First name: ${customer.first_name}`);
			console.log(`Last name: ${customer.last_name}`);
		});
});
```

메서드 체이닝을 통해 처리가 가능하다.

## IndexedDB 버전 관리

데이터베이스의 버전은 두가지 뿐이다.
첫 번째 버전 [[#Open Database Connection]] 에는 아무 객체도 저장되지 않고 비어있었으며, 두번째 버전에는 [[#데이터베이스 버전 번호 관리/객체저장소 변경]] 에서 하나의 객체 저장소가 추가되어 있다.

만약 `version number` 를 $3$ 으로 업데이트 하고 페이지를 새로고침하면, 에러가 발생한다.

이는 `이미 존재하는 object store 가 있어 실패했다` 는 내용이다.

`onupgradeneeded` 메서드를 추가하여 `customer` 객체 저장소를 생성하고 `version number`  가 $1 \rightarrow 2$  로 변경되면서, `db.createObjectStore` 를 실행하여, `cutomer` `object store` 를 생성한다.

하지만, $2 \rightarrow 3$ 으로 `version` 이 `update` 된다면, 이로인해 다시 `onupgradeneeded` 메서드가 실행되고 이미 존재하는 `customer` `object store` 를 생성하기에 발생하는 에러이다.

이로인해 $2 \rightarrow 3$ 가 되지 않고 , 그대로 $2$ 버전으로 남게된다. 

이는 문제가 있다. 이를 해결하기 위해서는 전통적인 데이터베이스 마이그레이션기법을 활용한다.

```js
request.addEventListener("upgradeneed", (event) => {
	const db = event.target.result;
	const oldVersion = event.oldVersion;
	if (oldVersion < 2) {
		db.createObjectStore("cutomers", {
			keyPath: "passport_number",
		});
	} 

	if (oldVersion < 3) {
		db.createObjectStore("employees", {
			keyPath: "employee_id",
		});
	}
});
```

이 메서드는 `DB` 의 이전 버전 번호를 확인하여, 모든 버전의 데이터베이스를 가장 최신 버전으로 가져오도록 수행할수 있다.

이는 처음 방문시, `oldVersion == 0`,  으로, $2$ 가지 마이그레이션이 작동하며,
이전 방문으로 $2$ 버전이었다면, `oldVersion < 3`  으로 이인해 $1$ 가지 마이그레이션이 된다.

하지만 이는 효율적이지 못하다.
다음처럼 처리할수도 있다.

```js
request.addEventListener("upgradeneed", (event) => {
	const db = event.target.result;
	if (!db.objectStoreNames.contains("customer")) {
		db.createObjectStore("customer", {
			keyPath: "passport_number",
		});
	}
	if (!db.objectStoreNames.contains("employees")) {
		db.createObjectStore("customer", {
			keyPath: "employee_id",
		});
	}
});
```

## Cursor 로 객체읽기

[[#GET 으로 단일객체 요청]] 의 `get`  을 사용하여 객체 저장소에서 단일 객체를 검색하는 방법을 살펴보았다. 이는 여러 객체를 한번에 쿼리하는 방식에는 적합하지 않다.

만약 정확한 키를 모르며, 범위를 지정하여 쿼리해야 한다면, `cursor` 를 사용해야 한다.

>[!info] Cursor 란?<br><br>`SQL` 기반 데이터베이스에 익숙하다면, `SELECT * FROM table` 쿼리를 실행시켜 `cursor` 를 오픈한다고 생각할것이다.<br><br>`WHERE` 절이나 `LIMIT` 으로 범위지정 할수 있는 것 처럼 `CURSOR`  도 `index` 나 `boundary` 로 범위지정될수 있다.<br><br>`SQL` 에서반환된 결과와 달리, `CURSOR` 가 결과를 포함하고 있지는 않다.<br>단순히 객체 저장소에 존재하는 실제 객체에 대한 포인트 목록이다.

```js
const request = window.indexedDB.open("my-datebase", 3);
request.addEventListener("success", (event) => {
	const db = event.target.result;
	const customerTransaction = 
	db.transaction("customers")
		.objectStore("customers")
		.openCursor()
		.addEventListener("success", (event) => {
			const cursor = evnet.target.result;
			if (!cursor) {
				return;
			}
			console.log(cursor.value.first_name);
			cursor.continue();
		});
});
```

- `objectStore` 는 [IDBObjectStore](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore) 객체를 반환한다.<br>[IDBObjeectStore.openCursor()](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore/openCursor) 는 비동기 메서드이며, `success` `event` 를 통해 `cursor` 를 받아 처리한다.

- `event.target.result` 를 통해 현재 `cursor`  를 가져오며, `continue()` 메서드를 호출하여, `cursor` 가 앞으로 이동할때마다, `success` `event` 가 발생한다.

- 마지막 데이터를 전달하거나, 객체 저장소가 비어있으면 `null` 을 가리킨다.<br>`cursor` 가 앞으로 이동할때마다 `success` `event` 가 발생하므로, `null` 을 가리키는지 확인해야 한다.

>[!info] `cursor`  는 [IDBCursor](https://developer.mozilla.org/en-US/docs/Web/API/IDBCursor) 객체이다.

## 인덱스 생성

`Object stroe` 에서 모든 객체에 대해 순회하여 이동하는 `cursor` 를 여는 방법에 대해서 살펴보았다.
특정 조건에 맞는 객체를 검색하려고 할때, 모든 객체 저장소를 살펴보아야 한다.

처음부터 끝까지 `scan` 한다면 이는 매우 비효율적이다.
여기에 `index` 를 사용하면, 객체 저장소를 `query` 할수 있고, `query` 와 매칭되는 `record` 만 순회하여 살펴보는 `cursor` 를 열수 있다.

```js
request.addEventListener('upgradeneeded', (event) => {
	const db = event.target.result;
	if (!db.objectStoreNames.contains('customers')) {
		db.createObjectStore('customers', {
			keyPath: 'passport_number',
		});
	}
	if (!db.objectStoreNames.contains('exchange_rates')) {
		// exchange_rates store 생성
		const exchangeStore = db.createObjectStore(
			'exchange_rates',
			{
				autoIncrement: true,
			}
		);

		// exchnageStore 의 from_idx index 생성 
		// exchange_from 을 사용하여, 검색
		exchangeStore.createIndex('from_idx', 'exchange_from', {
			unique: false,
		});
		// exchnageStore 의 to_idx index 생성 
		// exchange_to 를 사용하여, 검색
		exchangeStore.createIndex('to_idx', 'exchange_to', {
			unique: false,
		});

		exchangeStore.transaction.addEventListener(
			'complete',
			(event) => {
				const exchangeRates = [
					{
						exchange_from: 'CAD',
						exchange_to: 'USD',
						rage: 0.77,
					},
					{
						exchange_from: 'JPY',
						exchange_to: 'USD',
						rage: 0.009,
					},
					{
						exchange_from: 'USD',
						exchange_to: 'CAD',
						rage: 1.29,
					},
					{
						exchange_from: 'CAD',
						exchange_to: 'JPY',
						rage: 81.6,
					},
				];
				const exchangeStore = db
					.transaction('exchange_rates', 'readwrite')
					.objectStore('exchange_rates');

				for (
					const i = 0;
					i < exchangeRates.length;
					i += 1
				) {
					exchangeStore.add(exchangeRates);
				}
			}
		);
	}
});
```

>[!info] 인라인 키 VS `out of line key`<br><br>`autoIncrement` 키로 `exchange_rates` 저장소를 생성한다.<br>`autoIncrement` 를 `ture` 로 설정하면, `IndexedDB` 로 하여금 `unique index` 를 자동으로 생성하도록 할수 있다.<br><br>첫번째 객체는 `ID 1`, 두번재 객체는 `ID 2` 인 형식이다.<br>이렇게 값과 별도로 저장되는 키를 `out-of-line-key` 라 한다.<br><br>반면, `keyPath` 를 사용하는 키를 `inline-key` 라 한다.

- **`createIndex`**: 첫번째 인수는 `index` 명, 두번째 인수는 `key 경로`,  세번째 인수는 `옵션 객체` 를 받는다.

## Index 로 데이터 읽기

인덱스를 사용하면 특정 기준과 일치하는 결과만 순회하는 `cursor` 를 열수 있다.

```js
const request = window.indexedDB.open("my-database", 2);

request.addEventListener("success", (event) => {
	const db = event.target.result;
	const exchangeTransaction = db.transaction('exchange_rates');
	const exchangeStore = exchangeTransaction.objectStore('exchange_rates');

	exchangeStore
		.index('from_idx')
		.openCursor('CAD')
		.addEventListener('success', (event) => {
			const cursor = event.target.result;
			if (!cursor) {
				return;
			}
			const rate = cursor.value;
			console.log(
				`${rate.exchange_from} to ${rate.exchange_to}`
			);
			cursor.continue();
		});
	};
});
```

이 예시에서 `index` 를 통해 검색하기 위해 [IDBIndex](https://developer.mozilla.org/en-US/docs/Web/API/IDBIndex) 에서 `openCursor` 메서드를 사용한다.
`index` 는 `from_idx` 를 사용하며, `from_idx` `index` 는 `exchange_from` 값을 기준으로 `indexing` 되어 있다.

그러므로, 위의 예시는 `index` 내에서 `exchange_from` 이 `CAD` 인, 요소를 찾는다.

>[!info] [IDBObjectStore](https://developer.mozilla.org/en-US/docs/Web/API/IDBObjectStore) 에서의 `openCursor` 는 해당하는  `ObjectStore` 전체에서 검색하며, [IDBIndex](https://developer.mozilla.org/en-US/docs/Web/API/IDBIndex)에서의 `openCursor`  는 지정한 `Index` 에서 검색한다는것이 다르다

## Cursor 범위 제한

기본적으로 `cursor` 는 `object store` 의 모든 객체 혹은 `index` 로 부터 반환된 모든 객체를 순회한다.
이는 비효율적이다. 모든 객체를 순회할 필요없이, 범위를 지정하면 더 효율적으로 검색할수 있을것이다.

이를 위해 필요한것이 [IDBKeyRange](https://developer.mozilla.org/en-US/docs/Web/API/IDBKeyRange) 이다.
다음은 동일한 결과를 도출한다.

```js
exchangeIndex.openCursor("CAD");
exchangeIndex.openCursor(IDBKeyRange.only("CAD"));
```

`IDBKeyRange` 의 `only()`  는 물론 여러 메서드들이 존재한다.

- [IDBKeyRange.bound()](https://developer.mozilla.org/en-US/docs/Web/API/IDBKeyRange/bound_static): `bound()` 정적 메서드는 `key` 의 `upper`, `lower` 범위를 지정한다.<br>이는 `range` 의 끝지점이 포함될수도 있고, 포함되지 않을수도 있다.<br><br>`IDBKeyRange.bound(lower, upper, [lowerOpen], [upperOpen])`<br><br>👉 `lower`: 새로운 `key` 범위의 `lower`(`하한`) 을 지정<br><br>👉 `upper`: 새로운 `key` 범위의 `upper`(`상한`) 을 지정<br><br>👉 `lowerOpen`: `lower` 범위의 `endpoint` 값을 제외할지를 가리킨다. <br>`default` 는  `false`<br><br>👉 `upperOpen`:`upper` 범위의 `endpoint` 값을 제외할지를 가리킨다. <br>`default` 는  `false` <br>
```js
// "C"(포함) 와 "D"(미포함) 사이의 모든 키
exchangeIndex.openCursor(IDBKeyRange.bound("C", "D", false, true))
```

- [IDBKeyRange.only()](https://developer.mozilla.org/en-US/docs/Web/API/IDBKeyRange/only_static): `only()` 정적 메서드는 하나의 `value` 를 포함한 `key` 의 범위를 생성한다.<br><br>`IDBKeyRange.only(value)`<br><br>👉 `value`: `key` 의 범위에 대한 `value`

```js
exchangeIndex.openCursor("CAD"); // 같다
exchangeIndex.openCursor(IDBKeyRange.only("CAD")); // 같다
```

- [IDBKeyRange.lowerBound()](https://developer.mozilla.org/en-US/docs/Web/API/IDBKeyRange/lowerBound_static): `lowerBound()` 정적 메서드의 `IDBKeyRange` 인터페이스는 오직 `lower` 범위에 대한 `key` 를 생성한다.<br><br>`IDBKeyRange.lowerBound(lower, [open])`<br><br>👉 `lower`: 새로운 `key` 범위의 `lower`(`하한`) 을 지정<br><br>👉 `open`: `lower` 범위의 `endpoint` 값을 제외할지를 가리킨다.<br>`default` 는 `false` 이다.

```js
// "CAD" 를 포함하여, "CAD" 이상의 모든 키를 포함
// CAD, USD
IDBKeyRange.lowerBound("CAD", false);
```

- [IDBKeyRange.upperBound()](https://developer.mozilla.org/en-US/docs/Web/API/IDBKeyRange/upperBound_static): `upperBound()` 정적 메서드의 `IDBKeyRange` 인터페이스는 오직 `upper` 범위에 대한 `key` 를 생성한다.<br><br>`IDBKeyRange.upperBound(upper, [open])`<br><br>👉 `upper`: 새로운 `key` 범위의 `upper`(`상한`) 을 지정<br><br>👉 `open`: `upper` 범위의 `endpoint` 값을 제외할지를 가리킨다.<br>`default` 는 `false` 이다.

```js
// "CAD" 를 제외한 "CAD" 아래의 모든 키 포함
// AUD, BRL
IDBKeyRange.upperBound("CAD", ture);
```

- [IDBKeyRange.includes()](https://developer.mozilla.org/en-US/docs/Web/API/IDBKeyRange/includes): `include()` 메서드는 인스턴스 메서드이다.<br>이는 `key` 의 범위내에 포함된 `key` 인지 아닌지 `boolean`  으로 반환하여 알려준다.<br><br>`include(key)`<br><br>👉 `key`: `key` 범위안에 있는지 확인하길 원하는 `key` 를 지정

```js
// "A"(포함) 와 "K"(포함) 사이의 모든 값
const keyRangeValue = IDBKeyRange.bound("A", "K", false, false);

keyRangeValue.includes("F");
// Returns true

keyRangeValue.includes("W");
// Returns false
```

## Cursor 방향 설정

`cursor` 는 기본적으로 오름차순으로 정렬된다.
`cursor` 를 `open` 할때 두번째 인수를 `"prev"` 로 넘겨 객체를 내림차순으로 정렬할수 있다.

```js
const request = window.indexedDB.open("my-database", 4);

request.addEventListener("success", (event) => {
	const db = event.target.result;
	const exchangeTransaction = db.transaction("exchange_rates");
	exchangeTransaction
		.objectStore("exchange_rates)
		.openCursor(null, "prev")
		.addEventListener("success", (event) => {
			const cursor = event.target.result;
			if (!cursor) {
				return;
			}
			const rate = cursor.value;
			console.log(`${rate.exchange_from} to ${rate.exchnage_to}: ${rate.rate}`);

			cursor.continue();
		})
});
```

이는 `index` 가 아닌, `objectStore` 에서 `cursor` 를 열고 순회한다.
이때, $2$  번째 인자로, `"prev"` 를 넣어, 내림차순으로 정렬되게 하며, $1$ 번째 인자는 `IDBKeyrange` 객체 대신 `null`  을 사용한다.

이는 `index` 든, `object store` 이든, 여부에 관계없이 모든 `cursor` 는 `range` 및 방향에 관한 인자를 받을수 있고, 그 두가지를 받거나 받지 않을수 있다.

## 객체 저장소의 객체 업데이트

`put()` 메소드를 호출해 객체의 내용을 바로 업데이트할수 있다.

```js
const request = window.indexedDB.open("my-database", 4);

request.addEventListener("success", (event) => {
	const updateRate = {
		"exchange_from": "CAD",
		"exchange_to": "ILS",
		"rate": 1.2,
	};

	const db = event.target.result;
	db.transaction("exchange_rates", "readwrite")
		.objectStore("exchange_rates")
		.put(updateRate, 2)
		.addEventListener("success", (event) => {
			console.log("Updated")
		});
});
```

위는 `out-of-line-key` 를 사용했을때, 사용가능하다.
이는 `incrimentalNumber`  를 통해 자동적으로 `ID` 값을 할당하므로, 이 `ID` 를 식별하여 처리한다.

이 방식이 아닌, `inline key` 를 사용할때, `field` 의 `ID` 값을 알고 있어야 처리 가능하므로, 먼저 객체저장소에서 객체를 먼저 가져오는것이 좋다.

```js
const request = window.indexedDB.open("my-database", 4);

request.addEventListener("success", (event) => {
	const db = event.target.result;
	const customerStore = db
		.transaction("customers", "readwrite")
		.objectStore("customers");

	customerStore
		.openCursor()
		.addEventListener((event) => {
			const cursor = event.target.result;

			if (!cursor) {
				return;
			}

			const customer = cursor.value;
			if (customer.first_name === "Archie") {
				customer.first_name = "Archer";
				cursor.update(customer);
			} else {
				customer.first_name = "Tom";
				customerStore.put(customer);
			}

			cursor.continue();
		});
});
```

이 코드는 모든 고객 정보를 순회하는 `cursor` 를 연후에, 각 고객의 이름을 검사하고, `update` 한다.
하지만, `"Tom"` 은 `customerStore.put` 을 통해 `update` 한다.

이는 원하는 방식을 사용하여 처리 가능하다.
단, 두번째 인자로 `ID` 값을 주지 않는데, 이는 객체 자체에 `ID` 값이 포함되어 있기 때문이다.
굳이, `Primary Key` 를 명시할 필요는 없다.

## 객체 저장소에서 객체 삭제

객체 삭제는 `delete` 를 사용한다.

```js
const request = window.indexedDB.open("my-database", 4);

request.addEventListener("success", (event) => {
	const db = event.target.result;
	db.transaction("exchange_rates", "readwrite")
		.objectStore("exchange_rates)
		.delete(2);
});
```

이는 `ID` 가 $2$ 인 `exxhange_rates` 의 요소를 삭제한다.
이는 `cursor` 를 통해 처리도 가능하다.

```js
const request = window.indexedDB.open("my-database", 4);

request.addEvenetListener("success", (event) => {
	const db = event.target.result;
	db.transaction("customers", "readwrite")
		.objectStore("customers")
		.openCursor()
		.addEventListener((event) => {
			const cursor = event.target.cursor;	
			if (!cursor) {
				return;
			}
			const customer = cursor.value
			if (customer.first_name === "Stevens") {
				cursor.delete();
			}
			cursor.continue();
		});
});
```

## 객체 저장소에서 모든 객체 삭제

`clear()` 는 `success`  및 `error` 이벤트를 갖는 `request` 를 반환한다.
다음은 `customers` 객체 저장소를 삭제하고 삭제를 끝내는 즉시 콘솔에 메시지를 기록한다.

```js
const request = window.indexedDB.open("my-database", 4);

request.addEventListener("success", (event) => {
	const db = event.target.result;
	db.transaction("customers", "readwrite")
		.objectStore("customers")
		.clear()
		.addEventListener("success", (event) => {
			console.log(`Object store clear`);
		});
});
```

## 위로 전파되는 Bubbling IndexedDB 에러 처리

`IndexedDB` 의 `error` 는 `bubbling` 된다.
이말은, 만약 `openCursor` 에서 `error` 가 발생했는데, `error` `event` 가 없다면, 이는 `transaction` 의 `error` `event` 로 올라갈것이고, 이역시 없다면 `objectStore` 의 `error` `event` 로 넘어간다.

이렇게 위 단계로 `error` 가 전파되어 올라가는데, 이를 활용하여, 공통 에러 핸들러 정의가 가능하다.
이는 편하게 에러 처리를 할수 있는 기법중 하나이다.

## SQL 과 비교

- `Cursor` 는 `SELECT * FROM table;` 과 비슷하다.<br>단, `Cursor` 는 `SQL` 과 달리 객체의 `Pointer` 로 가리키고 있을 뿐이다.  

- `IDBKeyRange` 는 `SELECT` 문에서 `WHERE` 절의 관계와 같다.<br>`WHERE x >= y` 는 `IDBKeyRange.lowerBound(y, false)` 와 같다.

- `Index` 는 `SQL` 의`Index` 를 열별로 미리 만들어두어 사용하는 개념과 비슷하다.<br>`IndexedDB` 의 `Index` 는 저장된 하나의 객체 속성에대해서만 쿼리가 가능하다.

- `Cursor Direction` 은 `SQL` 의 `ORDER BY x DESC` 와 비슷하다.<br>`IndexedDB` 에서는 `prev` 를 사용하여, 내림차순으로 정렬한다.<br>`SQL` 과는 달리 `index key` 를 기준으로 결과 값을 정렬할수 있다.





