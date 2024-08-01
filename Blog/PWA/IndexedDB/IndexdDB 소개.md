
모든 앱에 필요한 데이터를 `CacheStorage` 에 저장했다.
 `CacheStorage` 에서 데이터를 답은 `JSON` 파일을 포함하는 `HTTP` 응답을 통째로 캐싱한다.
예약목록에 접근하고 싶을때마다, `cache` 에서 `file` 을 가져오올수 있고 `parsing` 할수도 있었다.

`App` 이 사용자의 `internet` 연결 상태와 상관없이 완전하게 작동하지만, 사소한 데이터 수정이 있을때 마다 `server` 에 크게 의존해야 한다면, 이를 `offlien 우선` 이라고 할수 없을 것이다.

결국 `browser` 는 `data` 를 지속적으로 처리할 더 좋은 방법이 필요하다.
이렇게 `network` 에 의존하지 않고 `data` 를 `local` 에 저장하고, 읽고, 수정할수 있도록 해야 한다. 

이를 사용하기 위한 도구가 바로 `IndexdDB` 이다.

## IndexedDB 란?

`IndexedDB` 는 브라우저 내에서 제공된 `trasaction` 객체 저장소 데이터베이스이다.

### IndexedDB 는 트랜잭션 기반으로 작동한다.

`Trasaction` 은 수행하는 작업을 그룹화 한다.
`Trasaction` 내의 모든 작업이 성공하거나 혹은 실패한다.

- 질의 통장에 $7$ 달러를 인출
- 제이크의 통장에 $7$ 달러를 입금

만약 질의 통장에서 $7$ 달러를 인출했지만, 제이크의 통장에 $7$ 달러가 입금되지 않았다고 하면 분명 문제가 생긴다.

>[!info] 그 반대의 경우도 문제가 생기기는 마찬가지이다.

이를 해결하기 위한 방법은 모든 작업이 성공했을때, 작업을 이행하는것이다.
이러한 방법이 바로, `Transaction` 이다.

`IndexedDB`는 이러한 `Transaction` 으로 동작한다.

### IndexedDB 는 객체 저장소 데이터베이스이다.

기존의 `relational database` 와는 달리 `object based storage` 로 구성된다.
각 `database` 는 다수의 `object storage` 를 가질수 있고, 각각의 저장소는 다수의 객체를 가질수 있다. 

`object` 는 자바스크립트 `object`, `boolean`, `number`, `BLOB` (`Binary Large Object`) 및 자바스크립가 처리할수 있는 대부분의 데이터 포멧중 하나일수 있다.

### IndexedDB 는 인덱스된 데이터베이스이다.

`relational database` 처럼 `index` 를 사용한다.
그 어떠한 `object store` 라고 할지라도, `index` 만 추가하면 원하는 객체를 검색하고 가져오는데 사용할 수 있다.

은행 고객 객체 저장소는 사용자의 성을 `index` 로 들고 있다면, 성을 기반으로 쉽게 찾아 관련 정보를 불러올수 있다

최종 로그인 시간을 `index` 로 사용한다면, 이를 이용해 마지막 로그인한 $10$ 명의 사용자를 불러올 수 있다.

### IndexedDB 는 브라우저 기반이다.

`IndexedDB` 는 브라우저에서 완전하게 실행된다.
`IndexedDB` 에 저장된 모든 데이터는 사용자의 연결 상태에 상관없이 접근 및 조작 가능하다

`local database` 에 대한 변경사항은 서버에 자동으로 반영되지 않는다.
`local database` 의 변경 사항을 서버에 전달하거나, 서버의 변경 사항을 `local database` 에 `update` 하는것은 전적으로 개발자의 마음에 달렸다.

`local database` 와 `server` 간의 데이터 전달을 더욱 쉽게 만들어주는 여러 개의 `open soruce library` 가 있다. 

>[!info] 대표적으로 [idb](https://www.npmjs.com/package/idb) 가 있다.

### IndexedDB 의 주의할 사항

- 개발자는 여러 개의 `DB` 를 생성할수 있다.<br>(대부분의 앱에서는 한개의 데이터베이스를 생성한다.)

- 각 데이터베이스는 여러 개의 객체 저장소를 들고 있다.

- 각 객체 저장소에는 한가지 타입의 데이터가 들어간다.

- 객체 저장소에는 `key/value pair` 로된 `record` 가 들어있다.

- 객체, 숫자, boolean, string, 날짜, 배열, 정규식, undefined, null 등 자바스크립트로 표현 가능한 대부분의 정보가 값이 될 수 있다.

- 키는 객체 저장소의 개별 값을 참조하는데 사용된다.

- `IndexedDB` 는 `동일 출처 정책`(`same-origin policy`) 를 따른다.<br>작성된 데이터가 다른 사이트에 노출될 것을 걱정하지 않아도 다른 사이트에 방문할수 있다.<br><br>자신의 도메인 내에서는 데이터를 읽고 쓸수 있지만, 다른 도메인의 `IndexedDB` 에 기록된 데이터는 접근할 수 없다.

- `IndexedDB` 의 `DB` 는 `version` 을 갖는다.<br>객체 저장소를 생성하거나 구조를 수정하고 싶다면, 새로운 버전의 데이터베이스 커넥션을 열어야 한다.<br><br>이렇게 커넥션을 열 때 `upgrade-needed` 이벤트가 발생한다.<br>현재 버전과 이전 버전사이의 데이터베이스에 대한 변경 사항 반영은 이 이벤트중에 처리될수 있다.

- 대부분의 `IndexedDB` 작업은 비동기다.<br> `API` 는 값이 요청되었을때 해당 값을 반환하지 않는다.<br>해당 이벤트를 처리하는 콜백 함수를 정의해야 한다.

`IndexedDB` 를 사용하는 작업의 대부분은 하나의 기본 패턴으로 정리할 수 있다. 

1. `DB` 를 연다
2. 객체 저장소에 읽기 혹은 쓰기를 위해 `transaction` 을 시작한다.
3. 객체 저장소를 연다
4. 객체 저장소에서 필요한 작업을 수행한다.
5. `transaction` 을 완료한다.




