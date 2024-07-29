
`CacheStorage` 는 개발자가 완전히 제어할 수 있는 새로운 형태의 **캐싱 레이어**(`caching layer`) 이다.

`CacheStrage` 는 캐시 생서 및 관리를 위한 기본적인 `API` 를 직접 제공한다.

브라우저 설저엥 따라 어떻게 작동할지 모르는 `cache policy` 에 신경 쓰는 대신, 개발자는 원하는 수 만큼 캐시를 생성하거나 열수 있고, 응답 정보를 캐시에 저장, 검색, 삭제할수 있다.

>[!info] 이전 브라우저들은 `cache`  시, `HTTP` 헤더를 사용하여 `contents` 에 대한 `hint` 를 브러우저에게 주는 방식으로 구성되었다.<br><br>이는 기존 개발자의 제어를 벗어난 형태의 방식이다.

이러한 `CacheStorage` 와 `ServiceWorker` 를 결합하여 캐시에서 삭제, 반환, 제어처리하기에 좋다.

## 캐시 결정

`Service Worker` 의 생명주기는 다음과 같다.

```sh
|    설치중    | --> |   활성화 중   | --> |   활성화 됨   |
   installing          activating          activated
```

`service worker` 의 `install` 이벤트는 `service worker` 가 처음 등록된 직후, `event` 가 활성화 되기 전에 단 한번만 발생하는 `event` 이다.

이때가 `service worker` 가 `page` 를 제어하고 `fetch` `event` 수신을 시작하기 전에, `offline` 화 가능한 모든 파일을 캐싱할수 있는 상황이다.

`install` 이벤트 내에서 `service worker` 설치 자체를 취소할수도 있다.
이후 사용자가 사이트에 재방문 하면 `service worker` 설치가 다시 시작된다.

이를 활용하면, `service worker` 의 의존성을 효과적으로 관리할수있다.
이를 `install` 이벤트시 처리하는 방법을 다음처럼 정리한다.

- 캐시 저장 
- 파일 미리 다운
- 문제발생시 `install` 취소

```js
self.addEventListener("install", (event) => {
	event.waitUntil(
		caches.open("gih-cache").then((cache) => {
			return cache.add("/index-offline.html");
		});
	);
});
```

>[!info] [event.waitUntil](https://developer.mozilla.org/en-US/docs/Web/API/ExtendableEvent/waitUntil)
>
> 이 `method` 는 `event` `dispatcher` 에게 작업중이라고 말한다.<br>이는 작업이 `successful` 되었는지 아닌지도 감지하는데 사용된다.<br><br> `service worder` 에서, `waitUntil` 메서드는 `promise` `settled`  될때까지 작업중임을 알려주며, 해당 작업을 완료하려면, `service worker` 를 종료해서는 안된다.
> --- 
> **`Installing` 상태**
> `waitUntil`  은 `task` 들이 완료될때까지, `installing` 상태 에서 멈춘다.
> 만약, `promise` `reject` 가 `waitUntil` 에 전달된다면, `install` 은 `failure` 로 간주되어 `service worker` 가 삭제된다.
> ---
> **`Activate` 상태**
> `activate`  이벤트는 `waitUntil` 메서드를 사용하여, `promise` `settled` 될때까지 `fetch` 그리고 `push` 같은 `event` 를 `buffer` 한다.
> 
> 이렇게 하면, `database` `schema` 업데이트하고, 오래된 `cache` 를 삭제할 시간을 확보하며, 다른 `event`  는 완전히 업그레이드된 상태에 의존할수 있다.

이 상태에서는 `index-offline.html` 에 의존적이다.

이는 성공적으로 설치가 진행되고, 새 `service worker` 가 활성화 되었다고 이야기하기 전에 성공적으로 `caching` 되었는지 확인해야 한다.

>[!info] `Promise` `settled` 상태는, 비동기 처리를 위해 `background` 로 넘겼다는 뜻이다.<br>이후, `callback` 함수가 이상없이 수행되면 `fulfilled`, 오류가 발생했다면 `rejected` 된다. 

>[!info] 앞에서 말했듯, `waitUntil` 은 `Promise` `settled` 되었을때, `install` `event` 를 연기한다.

위의 상황은 `caches.open` 이후, `promise` 가 `resolve` 될때까지, 작업을 진행한다.
만약, 문제가 생긴다면 `reject` 함으로써 설치를 중단할수 있다.

다음은, `index-offline.html` 에 `match`  되는 `html` `file` 을 `cache` 에서 가져오는 로직이다.

```js
self.addEventListener("fetch", (event) => {
	event.respondWith(
		fetch(event.request).catch(() => {
			return caches.match("/index-offline.html");
		});
	);
});
```

이때, `server` 에서 가져오는것이 아닌, `cache` 안에 `request` 에 대한 `response` 가 정말 있는지 확인하고, 바로 `response` 를 받는다.

>[!note] `CacheStorage` 는 `cors` (`Cross Origin Resource Sharing`) 보안 정책을 따른다.<br>`cache.match()` 혹은 `caches.open()` 을 사용해도, 현재 `Origin` 에서 생성된 `cahce` 에만 접근 가능하다.

>[!info] `cache.match`
>`match` 메서드는 주어진 `request` 에 대해서 `cache` 로 부터 `response` 객체를 반환한다.
>
>`match` 메서드는 모든 `cache` 에서 `match` 검색을 하는 `cache` 객체에서 호출되거나, 특정 `cache` 객체에서 호출될 수 있다.
>
>`match` 를 통하여 반환된 `promise` 는 `response` 를 찾지 못해도 `reject` 되지 않는다.
>단지, `response` 값이 `undefined` 로 반환할뿐이다.
```js
// 모든 캐시에서 일치하는 `request` 검색
caches.match("logo.png");

// 특정 캐시에서 일치하는 `request` 검색
cache.open("my-cache").then((cache) => {
	return cache.match("logo.png");
});

// 만약 `cache` 가 없는 경우
caches.match("not-exists.png").then((res) => {
	if (res) { // res 가 `undefined` 인지 확인
		return res
	} else {
		// res 가 `undefined` 이면 처리할 로직	
	}
});
```

다음은 모든 `response` 데이터를 `cache` 한다.

>[!info] `cache` 의 `add` 메서드는 `request` 객체를 받아 처리한다.<br>이는 매개변수로 사용되며, `URL` 은 해당 생성자와 동일한 규칙을 따른다.

```js
self.addEventListener("install", (event) => {
	event.waitUntil(
		caches.open("gih-cache", async (cache) => {
			await cache.add("gih-offline.html");
			await cache.add( 
			"https://maxcdn.boostrapcdn.com/boostrap/3.3.6/css/boostrap.min.css"
			);
			await cache.add("/css/gih-offline.css");
			await cache.add("/img/background-sm.jpg");
			await cache.add("/img/logo-header.png");	
		});
	);
});
```

`addAll` 을 사용하여, 이보다 더 간단한 방법으로 처리할수도 있다.

```js
const CACHE_NAME = 'gih-cache';
const CACHED_URLS = [
	"/index-offline.html",
	"https://maxcdn.boostrapcdn.com/boostrap/3.3.6/css/boostrap.min.css",
	"/css/gih-offline.css",
	"/img/background-sm.jpg",
	"/img/logo-header.png"
];

self.addEventListener("install", (event) => {
	event.waitUntil(
		caches.open(CACHE_NAME, async (cache) => {
			return await cache.addAll(CACHED_URLS);
		});
	);
});
```

그리고 `serviceworker.js` 에서 다음처럼 변경한다.

>[!info] `respondWith` 는 `browser` 의 기본 `fetch` 동작을 막으며, `response` 에 대한 `promise` 를 직접 수정 및 추가하여 제공할수 있는 메서드이다. 

>[!info] `fetch` 이벤트는 `browser` 가 `server` 에 `fetch` 할때  발생하는 이벤트이다. 

>[!info] `index-offline.html` 은 `index.html` 일수 있고, `/` 일수도 있다.<br> 그러므로, `headers` 의 `accept` 객체를 가져와 `text/html` 인 값이 있는지 확인한다.
```js
self.addEventListener("fetch", (event) => {
	event.respondWith(
		// event.request 를 `fetch`
		fetch(event.request)
			// fetch 에서 오류가 나왔다면 offline 이다.
			// offline 시 실행
			.catch(await () => {
				// cache 가 있는지 확인
				const res = await caches.match(event.request)
				// 있다면 반환값 반환
				if (res) {
					return res;
				// 없다면, 요청 `header` 에 `text/html` 이 있는지 확인
				} else if (
					event.request.headers.get("accept").includes("text/html")
				) {
					// 있다면, `index-offline.html` 반환
					return caches.match("/index-offline.html");	
				}
		});
	);
});
```

>[!note] `caches.match` 사용시, `event.request` 를 사용하여 처리도 가능하다.<br>하지만, 주의 해야 할점은 `query string` 을 같이 포함했을때, `query string` 도 같이 읽는다는 것이다.<br><br>만약, 이러한 `query string` 이 포함되어도 `contents` 에 아무런 영향을 주지 않는다면, 다음처럼 `ignoreSearch` 에 `match` 가 `query string` 을 무시하도록 할수 있다.<br><br>`caches.match("/promote.html", { ignoreSearch: true });`<br><br>이는 `/promote.html`, `/promote.html?utm_source=urchin`, `/promote.html?utm_medium=social` 모두 `match` 된다.

## HTTP 캐싱과 HTTP 헤더

`CacheStorage` 는 `HTTP` 캐시를 대체하지 않는다.

`HTTP` `header` 에 `Cache-Control: max-age=31536000` 헤더를 사용하여 파일을 제공하는경우, `browser cache` 에 `1년동안` 저장한다.

이와 동시에 `service worker` 는 `fetch` 이벤트를 받아, `cache storage` 에 파일을 저장할 것이다.

그럼, `1년 이전에` 이후 해당 파일을 수정했다고 가정하자.

그리고, `client`  는 새로 접속시 해당 파일을 요청할것이지만, `browser cache` 에 저장된 파일을 가져오고, `service worker` 의 `fetch` 이벤트 역시 `browser cache` 에 저장된 파일을 가져와 `cache storage` 에 저장할 것이다.

>[!info] 이는 간단하게, `Network` 통신이 아닌 `browser cache` 에서 파일을 가져온다는 것이다.

이는 `HTTP` `caching` 의 작동개념이며, 이를 이해하고 올바르게 수행하는것이 중요하다.

## 결론

`ServiceWorker`  `proxy` 를 통한 `Cache` 는 매우 강력하며, 빠르고, `offline` 시에도 `content` 를 반환할수 있도록 만든다.

이는 사이트의 `loading` 시간을 획기적으로 줄여주며, `server` 비용을 절약하는데 도움이 된다.









