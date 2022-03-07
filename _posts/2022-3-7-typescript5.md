---
layout: post
title: 타입스크립트를 사용한 Next.js 웹실습4
tags: [java/typescript, react]
excerpt: 메모장 서비스 만들기-fcm으로 푸쉬알림 보내기
---

## fcm으로 푸쉬알림 보내기

지금까지 우리는 파이어스토어를 활용해서 초간단 메모장 서비스를 구현해보았다.  
여기에 부가 기능을 넣어서 메모가 추가될 때 알림이 오도록 하는 서비스를 만들어보자.

## fcm 활성화하기

파이어베이스에 있는 FCM[^1] 활용하면 알림 서비스를 구현할 수 있다.  
다시 한번 파이어베이스 콘솔에 접속해보자.  
![tsc5-img1](/images/posts/typescript5-img1.png)  
프로젝트에 접속한 뒤, _프로젝트 개요_ 옆의 톱니바퀴를 눌러서 **프로젝트 설정**에 들어간다.  
설정창에 들어가고 **클라우드 메세징**을 클릭한다.  
![tsc5-img2](/images/posts/typescript5-img2.png)  
아래의 *웹 푸쉬 인증서*에서 **generate key pair**를 클릭한다.  
그러면 키 하나가 뜰 텐데, 이를 저장해 놓는다.

## 프로젝트 설정

[이 사이트에 친절하게 설명](https://firebase.google.com/docs/cloud-messaging/js/client?hl=ko)되어 있지만, 뭔가 설명이 복잡하다.  
본인이 친절하게 설명해주겠다.  
참고로 읽을 사람은 해당 사이트의 **자바스크립트 클라이언트 설정**, **테스트 메세지 보내기**, **메세지 수신** 항목을 보면 된다.

### 토큰 받기

먼저 프로젝트 폴더의 pages에 **fcm** 폴더를 생성하고, 거기에 messaging_get_token.ts 파일을 생성한다.  
![tsc5-img3](/images/posts/typescript5-img3.png)  
이 파일을 건드리기 전, firebase/firebaseConfig.ts에 다음과 같은 코드를 넣는다.

```javascript
var firebaseClientKey = (아까 클라우드 메세징에서 저장해놓은 키);
export var clikey = firebaseClientKey;
```

그 다음 messaging_get_token.ts 파일을 수정한다.

```javascript
import { getMessaging, getToken } from "firebase/messaging";
import { app, clikey } from "../../firebase/firebaseConfig";

// Get registration token. Initially this makes a network call, once retrieved
// subsequent calls to getToken will return from cache.
const initToken = () => {
  const messaging = getMessaging(app);
  getToken(messaging, { vapidKey: clikey })
    .then((currentToken) => {
      if (currentToken) {
        // Send the token to your server and update the UI if necessary
        // ...
        console.log(currentToken);
      } else {
        // Show permission request UI
        console.log(
          "No registration token available. Request permission to generate one."
        );
        // ...
      }
    })
    .catch((err) => {
      console.log("An error occurred while retrieving token. ", err);
      // ...
    });
};

export default initToken;
```

참고로 이 코드는 위 링크에 있는 걸 그대로 가져다가 Next.js에 맞게 적절히 수정한 것이다.  
원본 코드는 함수로 감싸지 않았지만 굳이 함수화한 이유는 Next.js의 특성과도 관련있는데,  
조금 있다 설명하겠다.

각설하고 이 코드는 **fcm 메세지 전송에 필요한 토큰**을 생성하는 코드이다.  
파이어베이스 api의 getMessaging() 객체와 아까 받아온 키를 getToken에 넣으면 토큰이 생성되는 원리이다!  
getMessaging()의 괄호 안에는 firebaseConfig에서 생성한 initializeApp 객체를 넣어주면 된다.

### 백그라운드 설정

백그라운드에서 알림을 받을 수 있도록 추가 설정이 필요하다.  
프로젝트의 **public** 폴더에 **firebase-messaging-sw.js** 파일을 생성한다.  
주의할 점은, **반드시** public 폴더에 위와 똑같은 이름으로 넣어야 한다. 타입스크립트가 아니라 자바스크립트이다!

이 파일은 _백그라운드에서 알림을 받기 위해_ 필요한 파일이다. 프로젝트 실행시 파이어베이스 모듈이 알아서 찾아 실행해준다.

```javascript
importScripts("https://www.gstatic.com/firebasejs/8.1.0/firebase-app.js");
importScripts("https://www.gstatic.com/firebasejs/8.1.0/firebase-messaging.js");

const config = {
  //firebaseConfig에 있는 것과 똑같이
};
firebase.initializeApp(config);

const messaging = firebase.messaging();
```

여기서 중요한 것은 import를 쓰지 않고 importScripts를 쓴다는 것인데,  
public 폴더는 프로젝트 모듈에 포함되지 않기 때문에  
`import initializeApp from 'firebase/app'` 등을 사용하면 실행시 오류가 난다.  
또한 타입스크립트는 importScripts를 읽기 못하기 때문에 자바스크립트로 사용해야 한다.  
importScripts를 보면 `firebasejs/8.1.0` 이렇게 나와있는데 **9 버전 이상**으로 설정하면 gstatic.com에 없다는 오류가 나온다. 8 버전으로 해주자.

### index.tsx 설정

웹 사이트를 열 때 토큰을 받도록,  
index.tsx에서 messaging_get_token.ts를 불러오는 설정을 해야 한다.  
보통 타 리액트 프로젝트는 \_app.tsx[^2]에서 불러오지만 **Next.js는 서버-사이드 렌더링**이기 때문에 해당 방법이 먹히지 않는다.  
서버 사이드 렌더링이 무엇이냐면, 페이지를 불러오는 방식의 차이이다.  
기존의 프레임워크들은 페이지에 접속할 때 본인 컴퓨터(클라이언트)에서 페이지를 만들어 불러온(렌더링)다.  
그런데 Next.js는 서버에서 페이지를 미리 전부 만들고, 본인 컴퓨터는 그냥 만들어진 페이지를 보기만 하면 된다. 이게 서버-사이드 렌더링이다.

따라서 약간의 편법을 써야 한다. 바로 **useEffect**를 사용하는 방법이다.  
index.tsx에

```javascript
import initToken from './fcm/messaging_get_token';
...

//함수 내 return 전에
useEffect(initToken, []);
```

를 넣으면 매번 페이지를 초기화할 때마다 토큰을 가져오는 함수가 실행될 것이다.  
위에서 함수로 감싼 이유가 바로 useEffect를 실행시키기 위함이었다.

## 테스트

실행 원리는 다음과 같다.

- 프로젝트를 실행하고 메인 페이지에 접속해 토큰을 받는다.
- 파이어베이스에서 제공하는 알림 테스트 기능에 메세지와 토큰을 넣어 보낸다.
- 메세지를 받는다!

### 토큰 받기

npm run dev으로 접속한다.  
fcm 토큰을 받으려면 **https** 연결이 필수이므로 [ngork](https://ngrok.com/) 등을 통해 실행한 서버 주소를 https로 재접속한다.

웹 콘솔창(`F12`)를 열어 _콘솔_ 항목에 접속해보면  
![tsc5-img4](/images/posts/typescript5-img4.png)

이런 식으로 토큰이 하나 나올 것이다.  
메세지를 보내기 위해서는 이 토큰이 필요하다.

### 메세지 보내기

이 테스트는 백그라운드에서 알림이 오는지 확인하는 테스트이므로 *페이지가 열린 창을 최소화*해 놓자.

파이어베이스 프로젝트 개요에 접속 후 화면을 아래로 내려
![tsc5-img5](/images/posts/typescript5-img5.png)  
**Cloud Messaging** 항목에 들어간다.

![tsc5-img6](/images/posts/typescript5-img6.png)  
들어간 뒤 **Send your first message**를 클릭한다.

![tsc5-img7](/images/posts/typescript5-img7.png)  
알림 제목과 알림 텍스트를 입력 후 **테스트 메세지 전송**을 클릭한다.

![tsc5-img8](/images/posts/typescript5-img8.png)  
**FCM 등록 토큰 추가**라는 항목에 콘솔창에서 받은 토큰을 입력 후, 테스트 버튼을 누른다.

![tsc5-img9](/images/posts/typescript5-img9.png)  
윈도우 알림창에 메세지가 뜨는 것을 볼 수 있다!

## 안 백그라운드에서도 알림 받기

이제 웹페이지 창이 열려진 상태에서도 알림을 받도록 설정해보자.

---

[^1]: **F**irebase **C**lient **M**essaging. 말 그대로 사용자(클라이언트)에게 메세지를 보내는 기능이다.
[^2]: 리액트 프로젝트 실행시 가장 먼저 실행되는 코드이다.
