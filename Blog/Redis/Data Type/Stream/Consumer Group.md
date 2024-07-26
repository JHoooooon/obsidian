
[[Stream]]의 모든 명령어는 `stream` 데이터를 한 개의 `consumers` 가 읽어가는 상황이었다.

이는 `fan-out` 방식으로 사용된다.

하지만, `fan-out` 하지 않고, 같은 데이터를 여러 `consumer` 가 나눠서 가져가기 원할수 있다.

![[Consumer Group.png]]

이는 소비자 그룹에 속한 이메일 서버가 각 `ID` 를 분산해서 처리하는것을 보여준다.

`Redis` `stream` 에서는 소비자 그룹에게 전달되는 순서를 고려할 필요없이, 그룹내의 소비자는 다른 소비자가 읽지 않는 데이터만을 가져가는 것을 볼수 있다.

>[!info] 다른 플랫폼같은 경우 전달되는 순서를 고려하기위한 처리를 따로 해주어야 한다.

만약 `Consumer Group` 중 하나의 `Consumer` 가 받은 `message` 가 어떠한 이유로 인해 성공하지 못햇다고 가정하자.

그럼 성공하지 못한 `message` 는 없어지게 된다.
이는 실패한 `message` 를 식별하고 다른 `Consumer` 에서 처리할수 있는 방법을 제공할 필요가 있다.

이러한 방식을 처리하기 위해 `Consumer` 는 `message` 를

`Consumer Group` 은 `Consumer` 에게 명시적으로 `message` 를 성공적으로 이행했다는 것을 요구 한다.

>[!info] 이를위해 [[#XACT]] 이라는 명령어를 제공한다

다음은 `Consumer Group`  을 사용할지 안할지를 선택하기 위한 기준이다.

1. `stream`  과 여러 `clients` 가 있고, 이 `client` 가 모든 메시지를 받기 원한다면, `Consumer Group` 이 필요없다. 

2. `stream` 과 여러 `clients` 가 있고, `stream` 을 분배(`partitioned`)하거나 `client` 들 사이에서 공유(`shared`) 하고, 그래서 각 `client` 는 `stream`으로 부터 전달된 `message` 의 하위 집합을 얻는다면 `Consumer group`  을 사용해야 한다.

## XGROUP 

[[#XGROUP]] 명령어는 여러개가 존재한다.
이는 다음과 같다.
### XGROUP CREATE

>[!info] XGROUP CREATE
```sh
XGROUP CREATE key group <id | $> [MKSTREAM]
  [ENTRIESREAD entries-read]
```

이는 새로운 `consumer group` 을 생성한다.
주어진 `group` 인자는, `stream key` 내의 고유 식별자로써 사용된다.

>[!warning] 만약, `stream key` 가 중복된다면, `BUSYGROUP` 에러가 발생한다.

- **key** : `stream` 의 `key`

- **group**: `stream key` 내의 생성 그룹의 고유 식별자 

- **<id | $>**: `id` 인자는 `group` 에 마지막 요소 `ID` 를 지정한다.<br>이는 마지막 요소 `ID` 이후의 요소의 메시지를 받는다는 의미이다.<br><br>\$ 을 지정하면, `stream` 의 현재 시점 이후의 데이터를 리스닝 하겠다는 의미이다.

>[!info] `$` 는 특별한 `ID` 로써, `stream` 내의 마지막 항목의 `ID` 를 가리킨다.<br> 그러니 마지막 항목의 `ID` 이후의 메시지를 받는다는 의미와 같다

만약, `ID` 를 $0$ 을 사용한다면, 이는 `Consumer Group` 에서 처음부터 전체 `stream` 을 받는다는 의미이다.

```sh
XGROUP CREATE mystream mygroup 0
```

- **MKSTREAM**: 이는 만약 `stream` 이 존재하지 않는다면, 해당 `stream` 을 생성하겠다는 것이다.

- **[ENTRIESREAD entries-read]**: 이 옵션은 새로 생성된 소비자 그룹이 이미 처리한 것으로 간주할 `stream` 항목의 수를 지정한다.<br><br>이는 `ENTRIESREAD`  다음에 들어가는 **숫자 값** 을 지정하여, 이미 읽은 것으로 처리할 항목 수를 지정하는 것이다.<br>
>[!info] 여기서 **이미 읽은 것으로 가주할 항목 수** 는, 지정된 `ID` 이전 항목 개수부터 시작하여, 그 이후의 모든 항목을 처리 대상으로 삼는다.

```sh
XGROUP CREATE mysteam mygroup $ ENTRIESREAD 1000;
```

이는 이 그룹이 마지막 `ID` 에서 $1000$ 개 이전 항목부터 시작하여, 처리 대상으로 삼는다.

>[!info] 헷갈릴수 있어 다시 정리한다!! 
>
>`stream` 에 $5000$ 개의 항목이 있다고 하자.
>`ENTRIESREAD 1000` 을 사용하면, 처음 `4000` 개 항목은 이미 읽은 것으로 처리되며,
>`Consumer Group` 은 `4001` 항목 부터 처리를 시작한다.

### XGROUP CREATECONSUMER

>[!info] [XGROUP CREATECONSUMER](https://redis.io/docs/latest/commands/xgroup-createconsumer/)
```sh
XGROUP CREATECONSUMER key group consumer
```

`Consumer GROUP` 의 `consumer`  를 생성하는 명령이다

- **key** : `stream` 의 `key`
- **group**: `stream key` 의 `group` 이름
- **consumer**: `stream group` 에 생성할 `consumer` 이름

>[!info] [[#XREADGROUP]] 에서, 해당하는 `Consumer` 가 없다면, 자동적으로 `Consumer` 를 생성한다.

### XGROUP DELCONSUMER

>[!info] XGROUP DELCONSUMER
```sh
XGROUP DELCONSUMER key group consumer
```

`Consumer GROUP` 의 `consumer`  를 삭제하는 명령이다
이는 더이상 사용되지 않는 `consumer` 가 있다면 유용한 명령어이다.

- **key** : `stream` 의 `key`
- **group**: `stream key` 의 `group` 이름
- **consumer**: `stream group` 에 생성할 `consumer` 이름

>[!note] `consumer`  가 보유하고 있던 `pending message` 는 삭제된 후에는 청구할수 없다.<br> 따라서 `group` 내의 `consumer` 를 삭제하기전에 보류중인 메시지를 요청하거나 승인하는것이 좋다.

### XGROUP DESTROY

>[!info] [XGROUP DESTROY](https://redis.io/docs/latest/commands/xgroup-destroy/)
```sh
XGROUP DESTROY key group
```

이 명령은 `consumer group` 을 완벽하게 제거한다.

`consumer` 가 활성화되어 있고, `message` 가 `pending` 중이라도, `consumer group` 을 삭제한다.
그래서 정말로 필요할때, 이 명령을 사용해야 한다.

- **key** : `stream` 의 `key`
- **group**: `stream key` 의 `group` 이름

### XGROUP SETID

>[!info] [XGROUP SETID](https://redis.io/docs/latest/commands/xgroup-setid/)
```sh
XGROUP SETID key group <id | $> [ENTRIESREAD entries-read]
```

`Consumer Group` 의 마지막으로 전달된 `ID` 를 설정한다.

[[#XGROUP CREATE]] 에서 `GROUP` 생성시, 마지막으로 전달된 `ID` 를 지정한다.
[[#XGROUP SETID]] 는 기존의 `Consumer Group` 을 삭제후 재생성 없이, `Consumer Group` 의 마지막으로 전달된 `ID` 를 수정한다.

- **key** : `stream` 의 `key`

- **group**: `stream key` 의 `group` 이름

- **<id | $>**: `id` 인자는 `group` 에 마지막 요소 `ID` 를 지정한다.<br>이는 마지막 요소 `ID` 이후의 요소의 메시지를 받는다는 의미이다.<br><br>\$ 을 지정하면, `stream` 의 현재 시점 이후의 데이터를 리스닝 하겠다는 의미이다.

- **[ENTRIESREAD entries-read]**: 이 옵션은 새로 생성된 소비자 그룹이 이미 처리한 것으로 간주할 `stream` 항목의 수를 지정한다.<br><br>이는 `ENTRIESREAD`  다음에 들어가는 **숫자 값** 을 지정하여, 이미 읽은 것으로 처리할 항목 수를 지정하는 것이다.<br>
>[!info] 여기서 **이미 읽은 것으로 가주할 항목 수** 는, 지정된 `ID` 이전 항목 개수부터 시작하여, 그 이후의 모든 항목을 처리 대상으로 삼는다.

## XREADGROUP

[[#XGROUP]] 은 생성 및 그룹 수정을 위한 명령어라면,
[[#XREADGROUP]] 은 읽기 위한 명령어이다.

>[!info] [XREADGROUP](https://redis.io/docs/latest/commands/xreadgroup/)
```sh
XREADGROUP GROUP group consumer [COUNT count] [BLOCK milliseconds]
  [NOACK] STREAMS key [key ...] id [id ...]
```

[[#XREADGROUP]] 은 [[#XREAD]] 의 특별한 버전의 명령어이다.

[[#XREADGROUP]] 은 `server` 에 기억된 주어진 `message` 를 받아 처리한다.
이를 이해하기 위해서는 `Consumer Group` 이 어떻게 처리하는지 살펴볼 필요가 있다.

#### PEL 

[[#XREADGROUP]] 을 읽을때, `message` 는 `PEL`(`Pending Entries List`) 라 불리는 `Consumer Group` 안에 저장된다.

이는 전달되었지만, [[#XACT]] 를 사용하지 않은 이상, 확인되지 않는 `message` 로 처리된다.
 [[#XACT]] 를 실행한다면, `PEL` 안의 대기중인 `message` 는 삭제되며, 이는 확인된 `message` 가 된다.
 
 >[!info] 만약, `PEL` 은 [[#XPENDING]] 명령을 사용하여 검사할수 있다.

#### XREADGROUP Command

`Consumer Group` 을 `READ` 하기 위한 명령어이다.

- **group**: `stream key` 의 `group` 이름

- **consumer**:  `consumer group` 의 `consumer` 식별자

- **[COUNT count]**: 반환 개수를 지정한다.

- **[BLOCK milliseconds]**: `Read` 시 다음 `message` 를 받을때까지, `block` 할 시간을 지정한다.<br> `BLOCK` 되는 동안, 대기 상태로 있다가, `message` 를 받으면 대기 상태가 풀린다.

- **[NOACT]**: `NOACT` 는 `PEL`(`Pending Entries List`) 로 메시지를 추가하는것을 방지한다.<br>[[#XREADGROUP]] 은 `message` 를 읽으면 `PEL` 로 `message` 를 저장한다.
 
- **STREAMS key [key...] id [id...]** : `Stream` 의 `key` 와 `id` 를 지정한다. <br>
 `STREAMS` 는 특별한 `ID` 가 필요하다.
 
- **>**: 이 특별한 `ID` 는 어떠한 다른 `consumer` 에 전달되지 않은 메시지만을 받기를 원한다는 특별한 `ID` 이다. <br><br>간단하게 새 메시지를 보내달라는 뜻이다.

- **0 또는 숫자 ID**: 이는 새로운 메시지를 확인하는 것이 아닌, 입력한 `ID` 보다 큰 `ID` 중 `Pending List` 에 속하던 메시지를 반환한다.

다음은 `XGROUP` 을 사용하여 처리하는 로직이다.
하나의 `Consumer Group` 에서 여러개의 `stream` 을 리스닝하는것도 가능하다

```sh
XGROUP CREAT Email BIGroup 0;
XGROUP CREAT Push BIGroup 0;

XREADGROUP GROUP BIGroup BI1 COUNT 2 STREAMS Email Push > >; 
```

`BiGroup` 은 `Email` 과 `Push` `stream` 을 동시에 리스닝한다.
여기에 더 많은 `group` 을 추가해본다.

```sh
XGROUP CREATE Email EmailServiceGroup $;

XREADGROUP GROUP EmailServiceGroup ES1 COUNT 2 STREAMS Email >;
```

```sh
XGROUP CREATE Push NotificationServiceGroup $;

XREADGROUP GROUP NotificationServiceGroup NS1 COUNT 2 STREAMS Push >;
```

이는 다음과 같은 그림으로 볼수 있다.

![[stream data process.png]]



