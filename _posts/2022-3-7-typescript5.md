---
layout: post
title: 타입스크립트를 사용한 Next.js 웹실습4
tags: [java/typescript, react]
excerpt: 메모장 서비스 만들기-fcm으로 푸쉬알림 보내기
---

## fcm으로 푸쉬알림 보내기

지금까지 우리는 파이어스토어를 활용해서 초간단 메모장 서비스를 구현해보았다.  
여기에 부가 기능을 넣어서 알림 메세지가 오도록 하는 서비스를 만들어보자.

## fcm 활성화하기

파이어베이스에 있는 FCM[^1]을 활용하면 알림 서비스를 구현할 수 있다.  
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

그 다음 messaging_get_token.ts 파일을 수정한다. 설명은 주석문으로 대신하겠다.

```javascript
import { getMessaging, getToken } from "firebase/messaging";
import { app, clikey } from "../../firebase/firebaseConfig";

// 토큰 생성
const initToken = async () => {
  // firebaseConfig에서 생성한 initApp 객체로 메세징 객체-messaging 생성
  const messaging = getMessaging(app);
  // 메세징 객체와 파이어베이스 키페어(clikey)로 토큰받기
  // getToken이 함수이므로 token 객체로 리턴값 저장
  // 참고로 getToken은 프로미스 객체를 리턴하므로 값을 받기 위해 await 써주기
  const token = await getToken(messaging, { vapidKey: clikey })
    .then((currentToken) => {
      if (currentToken) {
        return currentToken;
      } else {
        console.log(
          "No registration token available. Request permission to generate one."
        );
        return null;
      }
    })
    .catch((err) => {
      console.log("An error occurred while retrieving token. ", err);
      return null;
    });
  // 받아온 토큰인 token객체 반환
  return token;
};

export default initToken;
```

이 코드는 **fcm 메세지 전송에 필요한 토큰**을 생성하는 코드이다.  
토큰은 각 클라이언트가 고유한 값을 가지며, 토큰을 바탕으로 어디에 메세지를 보낼지 결정할 수 있다.  
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

### 앱 설정

**\_app.tsx**[^2]의 MyApp 함수 안에 다음과 같은 코드를 써주자.  
물론 return 앞에 써야한다.

```javascript
// 아래 임포트틀 추가
import { useEffect } from "react";
import { database } from "../firebase/firebaseConfig";
import { collection, doc, setDoc, getDoc, updateDoc } from "firebase/firestore";
import initToken from "./fcm/messaging_get_token";

// 파이어스토어의 tokens 콜렉션에 토큰 넣기
// 토큰을 굳이 파이어스토어에 넣는 이유는 조금 있다 설명
const dbTokenData = collection(database, "tokens");
/* 앱 실행시 토큰받는 함수(messaging_get_token 내 initToken)
initToken의 리턴값을 그냥 출력하면 프로미스 객체 반환(getToken()이 프로미스 함수)initToken이 끝날 때까지 기다리지 않고
바로 처리 안된 getToken값을 token에 담아 반환해버리므로 발생하는 현상
따라서 감싸는 함수에 async/await를 사용해서 getToken의 then 구문까지 실행되도록 대기 */
const initTokenWrapper = async () => {
  let token = await initToken();

  // 일단 'my' 이름의 문서 객체 만들기
  const docprofile = doc(dbTokenData, "my");
  // getDoc도 프로미스 함수이므로 마찬가지로 리턴값 받기위해 await 사용
  const data = await getDoc(docprofile);
  // 해당 문서가 콜렉션에 있으면 불러오고 타임스탬프만 업데이트
  if (data.exists()) {
    // 토큰 출력
    console.log(data.data().token);
    updateDoc(doc(dbTokenData, "my"), {
      timestamp: Date.now(),
    });
    // 없으면 문서 생성(토큰값과 타임스탬프를 필드로 하여)
    // setDoc은 문서 객체(doc)을 사용하여 문서 이름을 정해 생성할 수 있게 하는 기능(addDoc은 임의의 이름으로)
  } else {
    setDoc(doc(dbTokenData, "my"), {
      token: token,
      timestamp: Date.now(),
    });
  }
};

// 매번 페이지가 초기화될 때마다 위 함수를 실행하도록
// useEffect 내에서는 async/await를 사용 못하므로 감싸는 함수(..Wrapper) 생성
useEffect(function () {
  initTokenWrapper();
}, []);
```

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

백그라운드에서 알림이 오는지 확인하는 테스트이므로 *페이지가 열린 창을 최소화*해 놓자.

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

## 백그라운드 아니여도 메세지 받는법

fcm 메세지는 기본적으로 백그라운드 상태에서 알림을 받기 위한 기법이다.  
대표적으로 휴대폰의 푸쉬 알림이 있다.  
앱을 켜놓지 않아도 백그라운드 상태에서 돌아가면 알림을 받을 수 있는 것이다.  
마찬가지로 웹도 창을 열어놓지 않은 상태에서 알림 메세지를 받는 것이다.

다만 웹창을 연 상태에서도 굳이 메세지를 확인하려면 다음과 같이 하면 된다.

fcm 폴더에 **messaging_get_message.ts**를 생성한 뒤 다음과 같이 작성하자.

```javascript
import { getMessaging, onMessage } from "firebase/messaging";
import { app } from "../../firebase/firebaseConfig";

const initMessage = () => {
  const messaging = getMessaging(app);
  onMessage(messaging, (payload) => {
    console.log(payload.notification.title);
    console.log(payload.notification.body);
    alert(payload.notification.body + "이다");
  });
};

export default initMessage;
```

그 다음 **\_app.tsx**에 다음 코드를 추가하자.

```javascript
// 임포트 추가
import initMessage from "./fcm/messaging_get_message";
// initTokenWrapper 아래에 추가
useEffect(initMessage, []);
```

이제 마찬가지로 알림을 보내보면  
![tsc5-img10](/images/posts/typescript5-img10.png)  
이렇게 알림창과 콘솔창에 알림 메세지가 뜨는 것을 볼 수 있다.  
물론 적절히 가공하여 모달 창[^3] 형태로 만들 수도 있다.

## 직접 요청 보내기

이러한 알림 메세지는 원래 **서버**가 보내줘야 한다.  
즉 아까 만든 토큰 등도 원래 서버에게 보내고 서버가 그걸 저장한 후, 서버가 토큰을 바탕으로 각 클라이언트에게 메세지를 보내는 것이다.  
하지만 우리는 서버를 따로 만들지 않았으므로 프론트에서 이러한 알림 메세지를 보내보도록 하겠다. 혼자 북치고 장구치는 격?

우선 http 요청을 보내기 위해 *axios*라는 모듈을 설치해야 한다.

```
npm i axios
```

글 제목을 클릭하면 알림이 오도록 설정해보자.  
![tsc5-img11](/images/posts/typescript5-img11.png)  
여기를 클릭

우선 다시 파이어베이스 콘솔에 접속하여 _톱니바퀴-프로젝트 설정-클라우드 메세징_ 순으로 클릭한다.  
![tsc5-img12](/images/posts/typescript5-img12.png)  
**서버 키**라는 것이 보일 텐데, 이를 복사해 *firebaseConfig.ts*에 저장하자.

```javascript
var firebaseServerKey = 서버키(스트링 형태로);
export var serkey = firebaseServerKey;
```

아까 말했듯이 fcm 메세지는 서버가 보내는 것이다.  
이 키는 일종의 인증키로, 서버에서 이 키를 사용해 알림 요청을 보내지 않으면 인증 오류(401)이 뜬다.  
그 다음 TossoDetails.tsx로 이동하여

```html
<h2 onClick="{getAlert}">{singleTosso.title}</h2>
```

onClick 옵션을 추가하고 getAlert 함수를 구현해야 한다.  
return 앞에

```javascript
// 임포트 추가
import { app, database, serkey } from "../../firebase/firebaseConfig";
import axios from "axios";

//_app.tsx에서 만든 tokens 콜렉션 불러오기
const dbTokenData = collection(database, "tokens");

const getAlert = async () => {
  // 아까 _app.tsx에서 토큰 저장한 'my' 이름의 문서 볼러오기
  const data = await getDoc(doc(dbTokenData, "my"));
  // 여기서 토큰값 빼내기
  let token = data.data().token;
  // axios 요청-아래 그대로 써주도록
  axios({
    url: "https://fcm.googleapis.com/fcm/send",
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `key=${serkey}`,
    },
    data: {
      to: `${token}`,
      notification: {
        title: "Tosso",
        body: `${singleTosso.desc}`,
      },
    },
  });
};
```

이러고 한번 실행해보자.

![tsc5-img13](/images/posts/typescript5-img13.png)  
빨간색 체크된 부분(제목)을 클릭하면...  
![tsc5-img14](/images/posts/typescript5-img14.png)  
메세지가 오고, 콘솔창에도 메세지 내용이 출력된다!

물론 굳이 axios를 사용하지 않아도 Postman 같이 https 요청을 보낼 수 있는 프로그램이라면 메세지를 보낼 수 있다.  
다만 다시 한 번 말하겠지만, **fcm 메세지 전송은 서버에서** 이뤄진다는 것을 명심하라.  
**클라이언트는 토큰만 생성**하여 서버에게 보내주는 역할이라는 것도 잊지 말라.

---

[^1]: **F**irebase **C**lient **M**essaging. 말 그대로 사용자(클라이언트)에게 메세지를 보내는 기능이다.
[^2]: 리액트 프로젝트 실행시 가장 먼저 실행되는 코드이다.
[^3]: 화면 안의 화면이라는 뜻으로 팝업 창 등을 직접 구현하고 싶을 때 사용되는 기법이다.
