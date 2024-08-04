
## Push Notification 

사용자에게 알림을 보내는 기능만큼 네이티브 앱과 웹앱을 확연히 구분해주는 기능은 거의 없다.

일명 `Push Notification` 은 사용자에게 원하는 앱의 콘텐츠와 데이터를 적절한 시기에 업데이트 받을 수 있다.

개발자는 `push notification` 을 사용해 사용자 경험을 향상시키고, 이를 통해 사용자의 앱 사용량을 늘릴수 있다. 이는 사용자가 앱을 사용하도록 만들고, 앱을 성공으로 이끄는 가장 필수적인 요소이다.

사용자가 앱을 다시 사용하고, 계속 사용하게 만드는 것이야 말로 앱 설치를 통해 기대할수 잇는 수익을 높일 수 있는 핵심 방법이다.

이는 네이티브 앱을 성공으로 있는 가장 중요한 요소라고 해도 결코 과언이 아니다.
웹에서도 `push notification` 을 사용할수 있다.

사용자에게 알림을 보내는 기능만큼 웹 앱에 추가했을때 큰 효과가 있는 기능은 별로 없다.

## Push Notification Life cycle

`Push Notification` 은 `Push API` 를 사용하여  전송된 `메시지`와 `Notification API` 를 사용하여 보여지는 `알림` 의 두가지 기능으로 이루어진다

이러한 `Notification` 은 `browser` 밖에 표시되며, 단일 `browser` `window` 아 `tab context` 외부에 존재한다.

알림은 브라우저 윈도우나 탭과는 독립적으로 작동하기 때문에, 사용자가 사이트를 떠난 후에도 생성될수 있다.

```js
Notification.requestPermission().then((permission) => {
	if (permission === "granted") {
		new Notification("Shiny");
	} 
});
```

위는 `permission` 이 `"granted"`(`부여된`) 이라면, `"Shiny"` 라는 `Notification` 알람을 생성한다.
이부분에 대해서 어떠한 방식으로 처리가 이루어지는지 살펴본다.

### Push API

`Push API` 를 사용하면, 앱 사용자는 서버에서 보낸 `Push` 메시지를 구독하고, 서버에서는 언제든지 `browser` 로 메시지를 전송할 수 있다.

`Push` `message` 는 서비스워커에 의해 제어되며, 사용자가 앱을 떠난 후에도 `Push message` 를 받아 필요한 작업(`Task`)을 할수 있다.

>[!info] 여기서 `Task` 이란, 일반적으로 사용자에게 알림을 표시하는 것이다.

이를 나쁘게 사용한다면, 사용자 기기에 언제든지 메시지를 전송할수 있다면, `message` 를 끊임없이 보내 사용자를 괴롭힐수 있다. 이를통해 사용자 행동을 조용히 추적하는 것도 가능해진다.

이러한 나쁜 방식으로 남용되지 않도록, 모든 `Push message` 는 `중앙 메시지 서버`(`central messaging server`) 를 통해 전달된다.

이 `중앙 메시지 서버` 는 `message` 가 악용되거나, 사용자에게 너무 많은 메시지가 전송되지 않도록 방지한다.

또한, 사용자가 메시지를 받을수 없는 상태에서 메시지를 보낸 경우, 이후 메시지가 전될되도록 복잡한 작업을 처리한다.

이러한 단계를 총 $4$ 단계로 볼수있다.

#### 1단계: 푸시 메시지 구독

![[Send subscribe.png]]

먼저, `page` 는 `Push API` 의 `subscribe()` 메서드를 호출한다.
`method` 가 호출되면, 중앙 메시징 서버로 구독 요청이 전달되고, 중앙 서버는 신규 구독 상세 정보를 저장한후, `구독 상세 정보` 를 `page` 로 반환한다.

#### 2단계: 구독 세부내용 저장 

![[Save subscription info.png]]

`구독 상세 정보` 받은 `page`  는 이 정보를 나중에 사용할수 있도록 `app server` 로 전송한다.
`구독 상세 정보` 는 `사용자 상세 정보` 를 저장하는 테이블이나, 객체 저장소에 함께 저장하는 경우가 많다.

#### 3단계: 구독자에게 메시지 전송

![[Send message to subscriber.png]]

`App server` 는 저장해두었던, `구독 상세 정보` 를 가져와, `messaing server` 로 `message` 를 전송할때 사용한다.

#### 4단계:  서비스워커로 메시지 전달

![[Send Message To Service Worker.png]]

`Messaging server` 는 `App Server`  에게 받은 `message` 를 `Browser` 에게 전달한다.
이를 통해 `Browser` 의 `Service Worker` 는 `message` 를 수신하여 그 내용을 읽고 무엇을 할지 결정한다.

>[!info] [[#1단계 푸시 메시지 구독]] 단계의 `Push` 생성에는 사용자 권한이 필요하다.<br>보통의 `Browser` 는 알림 표시를 위한 권한을 한번만 요청한다.

### Push 알림 프로세스

`Push` 알림을 전송하는 전체 프로세스는 다음과 같다

1. `Page` 가 사용자에게 알림을 보여주기 위한 권한을 요청하면, 사용자가 그 권한을 부여
2. 






