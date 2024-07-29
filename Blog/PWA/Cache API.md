
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








