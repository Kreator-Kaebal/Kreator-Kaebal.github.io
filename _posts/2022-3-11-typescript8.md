---
layout: post
title: 타입스크립트를 사용한 Next.js 웹실습6
tags: [java/typescript, react]
excerpt: 메모장 서비스 만들기-fcm 메세지 보내는 다른 방법
---

## 파이어베이스 클라우드 함수

지난 시간에 api 서버를 만들고, fcm 메세지를 보내는 api를 제작해보았다.  
서버에서 토큰과 메세지를 받아 fcm 서버로 전송하는 원리였는데,  
이번에는 최신 fcm 메세지 보내는 방법을 알아보겠다.

## http v1 URI 사용하기

사실 지난 시간에 사용한 URI https://fcm.googleapis.com/fcm/send 는 _현재 구글에서 권장하는 방법이 아니다._  
구글에서는 이 URI를 레거시(Legacy)라 일컫으며, 이를 보완한 URI인 HTTP v1 방법을 사용할 것을 **강력히 추천**하고 있다.  
HTTP v1은 기존 레거시보다 보안성과 확장성이 늘어났다고 한다.(한 번 요청으로 iOS와 안드로이드 모두 보내기 가능)

다만 아쉽게도 HTTP v1은 레거시보다 사용법이 조금 복잡하다.  
우선 URI부터 이렇게 바뀌었다.

```
https://fcm.googleapis.com/v1/projects/(프로젝트 id)/messages:send
```

프로젝트 id는 프로젝트 콘솔 페이지에 접속하여 프로젝트 설정(톱니바퀴)->일반->프로젝트 ID에 나와있다.

그리고 헤더에 담기던 Authorization 키 형식이 바뀌었다.  
레거시의 Authorization이 `key=(서버 인증키)`였다면 v1은 `Bearer (OAuth2키)` 이다.  
OAuth2 키는 서버 인증키와는 매우 다른 키이므로, 이를 발급받으려면 계정 설정을 조금 건드려야 한다.

### OAuth2키 발급받기

서버는 fcm 서버에 요청을 보내므로 fcm 서버 입장에서는 *클라이언트*이다.  
이 OAuth2 키는 클라이언트가 구글 서비스(api)를 사용하기 위해 필요한 것인데, 한 계정당 하나가 만들어진다.  
위 URI에서 프로젝트 id를 구분한 것이 그 때문이다.  
[자세한 것은 여기로](https://developers.google.com/identity/protocols/oauth2)

각설하고, 키를 발급받으려면 https://developers.google.com/oauthplayground/으로 들어가야 한다.

![tsc8-img1](/images/posts/typescript8-img1.png)

수많은 무엇들이 나열되어 있는데, 이들은 OAuth2 키로 사용할 수 있는 기능(API)들을 나타낸 것이다. 여기서 사용할 API들을 선택해야 한다.
우리는 FCM 관련 기능만 필요하므로, **Firebase Cloud Messging API v1**을 클릭하자.

![tsc8-img2](/images/posts/typescript8-img2.png)

클릭하면 나오는 하위 항목 두 개에 모두 체크해주고 **Authorize APIs** 버튼을 누르자.  
로그인 또는 계정 선택 항목이 나오는데 **프로젝트를 만든 계정**을 선택하면 된다.

![tsc8-img3](/images/posts/typescript8-img3.png)

이런 창이 뜨면 두 항목에 모두 체크한 뒤 계속을 클릭한다.

![tsc8-img4](/images/posts/typescript8-img4.png)

여기서 파란 버튼을 누르면 아래 Refrest token과 Access token 창이 찬다.

우리가 필요한 것은 **Access token**이다. 이걸 복사하자.

참고로 이 Access token은 제한시간이 있다. 로그인이 풀리는 것처럼 일정 시간이 지나면 토큰이 유효해지지 않는 것이다.  
이럴 때는 *Refresh access token*을 클릭해 토큰을 다시 활성화시키면 된다.

어쨌든 Access token을 받았으니 이제 요청을 보낼 수 있다. 다음을 보고 요청을 알아서 작성하자.

```json
https://fcm.googleapis.com/v1/projects/(프로젝트 id)/messages:send
POST
header: {
    "Content-Type": "application/json",
    "Authorization": "Bearer (Access token)"
}
body: {
    {
        "message":{
            "token":(클라이언트 토큰),
            "notification":{
                "title":(메세지 제목),
                "body":(메세지 내용)
            }
        }
    }
}
```

예제-axios

```javascript
axios({
    url: https://fcm.googleapis.com/v1/projects/(프로젝트 id)/messages:send,
    method: "POST",
    headers: {
      Content-Type: "nodelication/json",
      Authorization: "Bearer (Access token)",
    },
    data:{
        message:{
            token:"(클라이언트 토큰)",
            notification:{
                title:"(메세지 제목)",
                body:"(메세지 내용)",
            }
        },
    },
})
```

이 요청을 보내면 메세지가 온다!

## 직접 메세지 처리하기

이렇게 URI를 사용하는 것은 결국 fcm 서버에 REST api를 보내는 것이다.  
메세지를 클라이언트에 보내는 역할은 fcm 서버가 담당하므로, 실제로는 서버도 단순 중개자에 불과한 것이다.  
fcm 서버에 전송하지 않고 서버에서 메세지를 보내는 방법도 있다.

서버에서 **firebase-admin**을 활성화시켜야 한다.

```
npm i firebase-admin
```

으로 firebase-admin을 서버에 설치한다.

그 다음 파이어베이스 웹 콘솔->프로젝트 설정(톱니바퀴)->서비스 계정으로 이동한다.

![tsc8-img5](/images/posts/typescript8-img5.png)

아래 "Admin SDK 구성 스니펫"을 복사한 다음 서버 폴더에 **firebase_admin_init.ts** 파일을 생성해 붙여넣는다.

이 파일은 firebase-admin을 활성화시키는 역할로, 비공개 키가 필요하다.  
**비공개 키 생성**을 클릭하면 json파일이 다운로드 받아지는데, *똑같은 파일은 다시 받지 못*하므로 관리에 조심하자.

이 json파일을 서버 폴더에 넣고, firebase_init.ts을 이렇게 수정해주자.

```javascript
...
var serviceAccount = require("(json파일의 상대경로)") // ex)require("./json파일.json")

export const app = admin.initializeApp({...})
```

그 다음 server.ts를 이렇게 수정한다.

```javascript
...
import { app } from "./firebase_admin_init"
import { getMessaging } from "firebase-admin/messaging"

...

node.post("/fcm", async(request, response) => {

    ...(const message까지)

    const datum = {
        data: {
        title: "(원하는 제목)",
        body: `${message}`,
        },
        token: `${token}`,
    };

    const messaging = getMessaging(app);
    messaging
        .send(datum)
        .then((response) => {
        console.log(response);
        })
        .catch((error) => {
        console.log(error);
    });

    ...(axios구문 삭제)

});
```

이제 클라이언트(localhost:3000)에서 [여기와 똑같이](https://kreator-kaebal.github.io/typescript6/) api 요청을 보내면 메세지가 온다!
