---
layout: post
title: 타입스크립트를 사용한 Next.js 웹실습5
tags: [java/typescript, react]
excerpt: 메모장 서비스 만들기-node.express로 서버구현
---

## 서버 만들기

이전 시간에 푸쉬 알림구현을 만들어보았다.  
이번에는 node.express 서버를 만들어 api 형태로 푸쉬알림을 전송하는 기능을 만들어보겠다.

아참, 프로젝트 폴더 디렉토리 구조가 조금 바뀌었다.  
난잡한 디렉토리 구조를 정리하느라 조금 바뀌었는데,

![tsc6-img0](/images/posts/typescript6-img0.png)  
참고하길 바란다.  
![tsc6-img0](/images/posts/typescript6-img0-1.png)  
이해를 돕기위한 이전 대조군

[여기에 들어가면 코드 전문을 볼 수 있다](https://github.com/kaebalsaebal/pseudo-website)

## 기본적인 원리

이전 시간에 그렇게 강조했듯이 *푸쉬 알림은 서버단에서 작동*하는 것이 원칙이다.  
푸쉬 알림을 보내려면 **서버 키**, **fcm 토큰**이 필요하다는 것도 배웠다.  
이를 도식화해서 표현하면

![tsc6-img1](/images/posts/typescript6-img1.png)

대충 이런 식으로 작동된다고 한다.

## node.express 서버 구축

### 소개

express.js라고도 하는 node.express는 node.js 개발을 쉽게 해주는 확장 프로그램 같은거라 보면 된다.

우선 새 폴더를 하나 만들어야 한다.  
**프로젝트 폴더 밖에** 만들어야 한다!

폴더 안에 들어가서

```
npm init
npm i express
npm i -D @types/node @types/express ts-node nodemon typescript
```

를 설치한다. 차례로

- express - node.express
- @types/node - 타입스크리트에서 node.js를 사용해기 위해 node 타잎을 추가
- @types/express - 마찬가지로 express.js를위해 express타잎추가
- ts-node - 컴파일 작업 없이 타잎스크립트를 바로 실행
  - tsc 서버.ts -> node 서버.js 이렇게 안해도 ts-node 서버.ts 한방에 실행가능
- nodemon - 코드 변경 후 저장시 서버를 다시시작

ts-node와 nodemon을 써먹기 위해 package.json을 수정한다.

```json
"scripts": {
    "dev": "nodemon --watch \"**/*.ts\" --exec \"ts-node\" server.ts"
}
```

이렇게 하면 npm run dev시 위 코드가 실행된다.

`tsc --init`을 사용하여 tsconfig.json도 만든다.  
만들었으면 폴더에 서버의 핵심인 **server.ts** 파일을 하나 만들자.

![tsc6-img2](/images/posts/typescript6-img2.png)

```javascript
// 아까 @types/express로 불러온 express객체
import express from "express";

const app = express();

app.listen("8080", () => {
  console.log(`8080 포트에서 실행중...`);
});
```

다음과 같이 입력한다. 서버를 실행하기 위한 최소 환경설정을 하는 것이다.  
이러고

```
npm run dev
```

로 서버를 실행해본다.

```
localhost:8080
```

위 주소로 접속하면 아래 화면과같은 에러가 날텐데  
![tsc6-img3](/images/posts/typescript6-img3.png)  
이는 서버에 올라간 html 페이지가 없기에 발생한 일이다.

서버를 켜면 페이지를 불러오도록 만들어보겠다.

우선 **public** 폴더를 만들고 html 파일을 아무렇게나 하나 만든다.

```
<h1>tosso is tissi</h1>
```

본인은 tosso.html이라는 이름으로 다음과 같이 만들었다.

```javascript
//(도메인)/ 주소에 tosso.html을 표시
app.get("/", (request, response) => {
  //sendFile 내에는 tosso.html의 절대경로 넣어주기
  response.sendFile(__dirname + "/public/tosso.html");
});
```

그 다음 server.ts에 다음을 추가한다.

서버를 다시 켜고 웹창을 새로고침해보면  
![tsc6-img4](/images/posts/typescript6-img4.png)  
이제 정상적으로 페이지가 뜨는 것을 볼 수 있다.

### api 만들어보기

그런데 우리 문제는 페이지를 띄우고 이런 것이 아니다.  
그건 후순위이고, 리액트 페이지에서 서버랑 정보를 주고받을 방식이 필요하다.  
가장 대표적인 방법은 **REST API**로, 웹 주소(URI) 형태로 소통하는 방법이라고 보면 된다.  
서버에서 한번 REST API를 만들어보겠다.

아까 만든 app.get을 그대로 활용하면 된다. 사실 이게 rest api이다.

```javascript
app.get("/", (request, response) => {
  //sendFile 내에는 tosso.html의 절대경로 넣어주기
  response.sendFile(__dirname + "/public/tosso.html");
});
```

많이 김빠졌을 것 같으니 하나씩 알아보자.

- **app.get**은 _GET_ 요청을 뜻한다는 것이다.
  - REST API는 소통하는 방식으로 *HTTP*라는 규격[^1]을 사용한다.
  - 이 규격은 크게 GET(서버에서 받기),POST(서버로 보내기)가 있고, 그 아래에 header,body,... 등 여러 형식이 존재한다.
  - 이전시간에 배운 axios로 fcm 메세지 보내기의 코드를 보면 method:POST,header:{...} 이런 식으로 되어있는데 이게 다 HTTP 양식인 것이다.
  - 만일 POST 요청을 보내고 싶으면 app.post로 하면 되겠지?
- **"/"**은 웹 주소의 세부 위치를 가리킨다.
  - 현재 서버 주소가 "tosso.tissi"이라면, 이 api는 "tosso.tissi/"를 접속하였을 때 발동된다는 것이다.
- **(request,response)**는 _요청과 응답_ 객체을 말한다.
  - 요청은 이 서버로 들어오는 것으로, 아까 method:POST,header:... 이게 다 요청이다.
  - 요청에는 각종 파라메타나 보안 토큰 등이 들어갈 수 있으므로 이 객체를 뜯어 사용자 체크나 파라메타에 맞는 처리를 진행할 수 있다.
  - 응답은 클라이언트로 넘어가는 것으로, 아래에서 말하겠다.
- **response.sendFile** 등 response.(ㅁㄴㅇㄹ) 에는 응답 객체에 어떤 것을 넣어 보낼지 지정해줄 수 있다.
  - 위에서는 sendFile이므로 tosso.html이라는 파일이 클라이언트로 넘어간다는 말이다.
  - 따라서 아까 클라이언트에서 접속시 html 파일이 표시되는 거였다.

이를 응용하여 테스트삼아 하나의 api를 만들어보자. server.ts에 추가

```javascript
app.get("/tosso", (request, response) => {
  response.send({ message: "tosso" });
});
```

위 설명을 종합해보면, 이 api는  
(서버주소)/tosso에 GET 요청을 보낼 시 {message:'tosso'}라는 JSON이 클라이언트로 보낸다는 것이다.

npm run dev로 서버를 실행해보고 http 요청을 보낼 수 있는 프로그램을 하나 실행시켜본다.  
본인은 **POSTMAN**이라는 프로그램을 사용하였다. api 테스트시 아주 유용한 프로그램이다.  
[다운로드는 여기서](https://www.postman.com/)

![tsc6-img5](/images/posts/typescript6-img5.png)

이 사진에서 GET은 GET 요청, localhost:8080/tosso는 요청 주소,  
아래 params,auth,...들은 요청 객체에 들어가는 것들을 지정해주는 것이다.  
현재 api에는 딱히 그런 것들을 확인하지 않으므로 전부 비워둔다.  
Send 버튼을 눌렀더니 맨 아래 창의 Body에 JSON 하나가 오는 것을 볼 수 있다.  
응답 객체의 Body에 아까 만든 JSON이 담겨서 온 것이다.

### 리액트에서 요청보내기

이제 api를 어떻게 써먹는지 알게 되었으니, 리액트로 요청을 보내보자.  
리액트에서는 저번에도 쓴 **axios**라는 것으로 http 요청을 보낼 수 있다.

TossoDetails.txt의 getAlert 함수를 수정하여 직접 fcm 요청을 보내지 않고 서버를 거쳐 보내보도록 하자. 참고로 토큰은 클라이언트가 이미 가지고 있다 가정한다.

일단 getAlert 함수 내부를 통째로 서버로 옮긴다. 또한 firebaseConfig.ts에 있는 serkey도 서버로 옮긴다.

그 다음 getAlert 함수를 다음과 같이 수정한다.

```javascript
const getAlert = async () => {
  // 토큰을 파이어스토어에서 꺼내오기
  const data = await getDoc(doc(dbTokenData, "my"));
  const { token } = data.data();

  // 토큰과 받고싶은 메세지를 요청의 Body에 담아 POST전송
  axios({
    url: "http://localhost:8080/fcm",
    method: "POST",
    data: {
      token: `${token}`,
      message: "tosso",
    },
  });
};
```

이제 이게 서버(localhost:8000)로 갈 것이다.  
서버에서는 이걸 받아 처리해줘야 한다.  
server.ts에 옮겨놓은 함수를 적절히 가공하자.  
물론 서버에 axios 설치도 필수!

```javascript
import axios from "axios"

...

// express.json()이라는 JSON 파싱 미들웨어(기능)을 사용한다는 것
//요청은 JSON 형태로 오기 때문에 사용
app.use(express.json());

app.post("/fcm", (request, response) => {
  // 요청의 body 부분을 빼오기
  const data = request.body;
  // 이렇게 body에서 토큰과 메세지 빼기
  // express.json()을 사용하였으므로 점 하나로 간단하게 처리
  const token = data.token;
  const message = data.message;
  // fcm 서버로 요청
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
        body: `${message}`,
      },
    },
  });
  // response는 필요 없으므로 사용x
});
```

이제 리액트와 서버 각각 npm run dev를 하여 상황을 지켜보자.

```
Access to XMLHttpRequest at 'http://localhost:8080/fcm' from origin 'http://localhost:3000' has been blocked by CORS policy: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

버튼을 클릭했으나 아무 일도 일어나지 않고,  
혹시나 해서 콘솔창을 들어가보니 위와 같은 오류가 뜬다.

이는 *CORS 오류*라고 해서, 간단히 말해 **브라우저가 요청하는 주소가 달라서** 생기는 문제이다.
현재 서버는 localhost:8080 주소상에서 돌아가고 있다. 이 주소(포트 포함) 내의 항목들(localhost:8080/... 등)에는 자유로이 요청할 수 있다.  
하지만 다른 주소(포트 포함)에서 요청이 들어올 때는 브라우저가 이를 인식하고 CORS 오류를 내어 응답값 접근을 막아버린다.  
여기서는 리액트가 작동중인 localhost:3000 주소에서 요청이 들어오는데 포트 번호가 달라 다른 주소로 취급되어 오류가 난 것이다.

그러면 이 오류를 어떻게 해결할 것인가...  
임시방편책으로 아래와 같은 방법이 있다.

```
npm i cors
npm i -D @types/cors
```

node.express에서는 cors를 처리할 수 있는 미들웨어(부가기능)을 제공하므로
npm을 통해 설치한다.  
이 때 cors가 타잎스크립트 환경에서도 작동하도록 하는 @types/cors도 개발자 설치한다.

그 다음 server.ts에서

```javascript
import cors from "cors"
...
// app.post(...) 위에
app.use(cors());
```

이렇게 해 주면 서버에서 cors 오류를 무시한다는 말이 되어[^2] 어떤 주소에서 오든 서버가 받아들인다.  
문제는 그러면 해킹의 염려가 있을 것이다. 예를 들면 요청값에 바이러스 코드를 넣는다던지...  
즉 cors는 자기 주소만 허용해 그러한 문제를 일차적으로 원천 차단하는 방화벽 같은 것이다.  
cors에 특정 URL만 신뢰하는 설정을 넣어줄 수 있다.

```javascript
const mycors = {
  origin: "http://localhost:3000",
  credentials: true,
};

app.use(cors(mycors));
```

이런 식으로 하면 다른 주소는 다 차단하되, localhost:3000으로 시작하는 것들은 허용한다는 말이다.

서버에 이를 설정하고 실행해보면...

![tsc6-img6](/images/posts/typescript6-img6.png)

이제 잘 나온다!

---

[^1]: 유식한 말로 **프로토콜**(Protocol)이라고 한다.
[^2]: 정확히는 모든 주소를 허용하겠다는 말이다.
