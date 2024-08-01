
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
                        'complate',
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

