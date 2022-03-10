---
layout: post
title: 타입스크립트를 사용한 Next.js 웹실습5
tags: [java/typescript, react]
excerpt: 메모장 서비스 만들기-node.express로 서버구현
---

## 서버 만들기

이전 시간에 푸쉬 알림구현을 만들어보았다.  
이번에는 node.express 서버를 만들어 api 형태로 푸쉬알림을 전송하는 기능을 만들어보겠다.

## 기본적인 원리

이전 시간에 그렇게 강조했듯이 *푸쉬 알림은 서버단에서 작동*하는 것이 원칙이다.  
푸쉬 알림을 보내려면 **서버 키**, **fcm 토큰**이 필요하다는 것도 배웠다.  
이를 도식화해서 표현하면

![tsc6-img1](/images/posts/typescript6-img1.png)

대충 이런 식으로 작동된다고 한다.  
참고로 인터넷 검색으로 나오는 **프론트/백 엔드**는 클라이언트와 서버의 유식한 표현이라고 생각하면 좋다.

## node.express 서버 구축

### 소개

express.js라고도 하는 node.express는 node.js 개발을 쉽게 해주는 확장 프로그램 같은거라 보면 된다.

우선 새 폴더를 하나 만들어야 한다.  
**프로젝트 폴더 밖에** 만들어야 한다!

폴더 안에 들어가서

```
npm init
npm i express
```

로 node.js 프로젝트임을 밝힌 후 express를 설치한다.  
서버의 핵심인 **server.ts** 파일을 하나 만들자.

![tsc6-img2](/images/posts/typescript6-img2.png)

```javascript
const express = require("express");
const path = require("path");
const app = express();

const http = require("http").createServer(app);
http.listen(8080, () => {
  console.log("8080 포트에서 실행중");
});
```

다음과 같이 입력한다. 서버를 실행하기 위한 최소 환경설정을 하는 것이다.  
이러고

```
node server/server.ts
```

로 서버를 실행해본다.

```
localhost:8080
```

위 주소로 접속하면 아래 화면과같은 에러가 날텐데  
![tsc6-img3](/images/posts/typescript6-img3.png)  
이는 서버에 올라간 html 페이지가 없기에 발생한 일이다.

서버를 켜면 페이지를 불러오도록 만들어보겠다.

우선 **public** 폴더에 html 파일을 하나 만들고 아무렇게나 입력한다.

```
<h1>tosso is tissi</h1>
```

본인은 main.html이라는 이름으로 다음과 같이 만들었다.

```javascript
app.use(express.static(path.join(__dirname, "./public")));

app.get("/", (request, response) => {
  response.sendFile(path.join(__dirname, "./public/main.html"));
});
```

그 다음 server.ts 아래줄에 다음을 추가한다.

서버를 다시 켜고 웹창을 새로고침해보면  
![tsc6-img4](/images/posts/typescript6-img4.png)  
이제 정상적으로 페이지가 뜨는 것을 볼 수 있다.

### 리액트 연동

이전에 만든 리액트 프로젝트를 이 서버로 열어보자.  
이전에는 npm run dev로 리액트를 열었는데,  
이는 create-next-app(또는 create-react-app)으로 프로젝트 만들 시 제공하는 기능으로  
쉽게 말해 개발용 임시 서버를 알아서 생성해 연동해주는 기능이다.  
서버를 굳이 구축하지 않아도 되므로 빠르게 개발 현황을 확인할 수 있는 장점이 있지만  
이렇게 생성한 서버는 *화면을 보여주기만 하는 것*만 기능하므로  
추가 기능(api제작, DB관리 등)을 구현하고, 배포하려면 서버를 만들 수밖에 없다.

![tsc6-img5](/images/posts/typescript6-img5.png)  
리액트 프로젝트의 package.json 중 scripts 항목이다.  
npm run dev를 하면 next dev가 실행되고, run build를 하면 next build가 실행되는 식이다.

우리가 생성한 서버에 리액트 프로젝트를 올리려면 **next build** 기능을 활용하면 된다.  
리액트 프로젝트를 배포한다는 뜻인데, 서버에 올릴 수 있도록 변환해준다는 말이다.

```
npm run build
```
