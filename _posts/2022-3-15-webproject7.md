---
layout: post
title: 타입스크립트를 사용한 웹 프로젝트7
tags: [java/typescript, firebase]
excerpt: 구글 클라우드 함수로 손쉽게 api구현
---

## 구글 클라우드 함수

구글 클라우드 함수를 사용하면 서버를 구축하지 않고도 함수 형태로 api를 구현할 수 있다.

아는 사람은 다 알겠지만, aws의 람다함수 같은 것이다. 클라우드 상에서 서버가 돌아가는 원리이다.

구글 클라우드 함수로 api를 구현해보자.

## 구글 클라우드 프로젝트 생성

[이 공식문서를 참조](https://firebase.google.com/docs/functions/get-started?hl=ko)

### 구글 CLI 설치

구글 클라우드 함수를 사용하기 위해서는 먼저 파이어베이스 프로젝트 생성이 필요하다.  
파이어베이스 프로젝트는 **구글 CLI**라는 콘솔을 사용하여 설치할 수 있다.

콘솔창에

```
npm i -g firebase-tools
```

을 입력해 구글 CLI를 설치한다.  
-g 옵션을 추가해 전역 설치하여 어디서든 콘솔을 불러올 수 있도록 하자.

```
npm i firebase-functions
```

```
npm i firebase-admin
```

파이어베이스 어드민과 함수도 설치해 놓자.

### 파이어베이스 클라우드 함수 프로젝트 생성

그 다음 파이어베이스 클라우드 함수 프로젝트를 생성해야 한다.

먼저 파이어베이스 프로젝트(웹 콘솔로 만들었던)를 만든 계정과 연동하기 위해

```
firebase login
```

을 실행시킨다.

![wp7-img1](/images/posts/webproject7-img1.png)  
이런 창이 뜨는데,  
(위의 Update Available... 네모박스는 무시한다.)

_Allow Firebase..._(대충 구글측에게 통계수집용 정보제공 하겠느냐는 말)는 Yes/No 아무거나 하면 뜬금없이 인터넷이 열리면서 구글 로그인 창이 뜬다.  
파이어베이스 프로젝트를 만들었던 계정으로 로그인하면
_+ Success! Logged in as (계정)_ 이렇게 뜨고 끝난다.

그 다음 프로젝트로 사용할 폴더를 하나 만들고 콘솔창 디렉토리를 그곳으로 이동한 뒤,

```
firebase init firestore
```

를 실행시킨다. 클라우드 함수 프로젝트가 파이어스토어를 사용할 수 있도록 하는 작업이다.

![wp7-img2](/images/posts/webproject7-img2.png)  
위와 같은 창이 뜰 텐데, 첫번째 질문은 해당 디렉토리(모자이크된)를 프로젝트 폴더로 사용할 것인가,  
두 번째 질문은 계정의 파이어베이스 프로젝트와 이 프로젝트를 연동할 것인가를 물어보는 것이다.

우리는 이미 만들었으므로
_Use an existing project_  
를 선택한 후 해당 프로젝트 이름을 선택하면 된다.

![wp7-img3](/images/posts/webproject7-img3.png)  
나머지는 다 엔터쳐주면 프로젝트 폴더에 무엇인가가 생성되고 끝난다. 파이어스토어 설정 파일이라 보면 된다.

이제 이 폴더에 또

```
firebase init functions
```

를 실행시켜준다. 이게 진짜 함수 프로젝트를 생성하는 일이다.

![wp7-img4](/images/posts/webproject7-img4.png)

참고로 두 번째 질문에서 **Typescript**를 선택하면 그 다음 질문에 ESLint를 사용할 것인지 물어보자.  
여기서는 귀찮으므로 No로 하였다.  
그 다음 질문인 **Dependency**는 꼭 Yes로 하여 작동에 필요한 npm 패키지들을 설치해주도록 하자.

![wp7-img5](/images/posts/webproject7-img5.png)

설치가 완료되면 폴더에 **functions**라는 폴더가 생성되고, 안에 들어가보면 이렇다.  
(eslint Yes시 .eslintrc.json 파일도 생긴다)

## 함수 만들기

우리는 functions 폴더 내 파일들만 신경쓰면 된다.

**src** 폴더에 들어가보면 **index.ts**(자바스크립트 선택시 js) 파일이 있을 텐데, 이 파일에서 함수를 만들면 된다.

### 간단한 예제

```javascript
import * as functions from "firebase-functions";
import * as admin from "firebase-admin";

import Ajv from "ajv";
import { custmemberschema } from "./interfaces/custmemberschema";

admin.initializeApp();
```

index.ts 파일은 서버를 실행시키면 가장 먼저 실행되는 파일이다.  
물론 이 프로젝트는 클라우드 함수이기 때문에 실제 배포시에는 클라우드상에서 서버가 알아서 돌아가지만,  
테스트시에는 서버가 필요하기 때문이다.

어쨌든 functions와 admin을 불러와서 admin.initializeApp()으로 서버를 초기화한다.

Ajv는 요청 body 형식을 확인하는 npm 모듈로, 일단은 신경쓰지 말자.  
custmemberschema는 사전에 정의해놓은 body 형식이다.

그 다음 아래에서 함수를 생성하는데...

```javascript
export const datastore = functions.https.onRequest(
  async (request, response) => {
    //const memkeys: Array<string> = Object.getOwnPropertyNames(request.body);
    try {
      const data = Object(request.body);
      let check = ajv.validate(custmemberschema, data);

      if (check) {
        await admin.firestore().collection("test").add(data);
        response
          .status(200)
          .json({ message: "파이어스토어 저장이 완료되었습니다." });
      } else {
        response
          .status(400)
          .json({ message: "요청 body 형식을 다시 확인하세요." });
      }
    } catch (error) {
      response.status(500).json({
        message: "서버와의 연결이 끊어졌습니다.",
        error: error,
      });
    }
  }
);
```

대충 request body에 담긴 내용을 test라는 파이어스토어 콜렉션 문서로 추가한다는 것이다.

ajv 모듈을 사용해 리퀘스트 보디가 요구되는 형식(custmemberschema)과 맞지 않으면 false를, 맞으면 true를 반환한다(check)

만일 check가 true이면 그대로 파이어베이스에 저장 후 200 메세지를, false이면 400 메세지를 띄운다.

응답을 보낼 때는 response[^1].status(HTTP 응답코드).send((json 형식의 응답메세지)) 이렇게 하면 된다.  
.send 대신 .json을 사용할 수도 있다.

파이어스토어 문서 추가는 네트워크 통신이므로 **async-await** 구문을 사용해서 파이어베이스 측에서 네트워크 응답(추가되었다거나, 에러났다거나...)을 줄 때까지 기다려주자.

여기서 api를 제작할 때 항상 눈치보여야 할 것이,  
이렇게 네트워크 통신이 사용되는 구문에서는 **try-catch** 구문을 사용하여 에러가 났을 때 붙잡아서 500메세지를 전송해줘야 한다.(에러 핸들링)  
api는 네트워크 통신이라 언제 어디서 에러가 날지 모르기 때문이다.

여기서는 에러 내용도 같이 전송해줬지만, 원래 에러 내용은 서버 콘솔로만 띄우고 클라이언트는 에러 내용을 몰라야 하는 것이 보안상 좋다.

아무튼 이렇게 해서 api를 만들었고, 테스트해 보자.

### 테스트

functions 폴더에서 콘솔창을 켠 뒤

```
firebase emulators:start
```

를 입력하면 에뮬레이터가 켜진다.

이 에뮬레이터라는 것이, 클라우드에 배포하기 전 임시 서버를 열어 테스트해보는 용도이다.

![wp7-img6](/images/posts/webproject7-img6.png)

에뮬레이터가 정상 작동되면 콘솔창에 이런 주소가 나오는데,  
맨 위의 `http://localhost:4000`에 접속하면 에뮬레이터 콘솔에 들어갈 수 있다.

![wp7-img7](/images/posts/webproject7-img7.png)

에뮬레이터 콘솔에 접속하면 이렇게 나오는데,  
firestore/functions emulator 이외에는 off로 되어있다.  
이는 아까 firebase init 할 때 firestore와 functions만 했기 때문이다.  
다른 서비스를 사용하고 싶으면 firebase init (명칭) 하면 된다.

파이어베이스 프로젝트와는 독립된 하나의 임시 프로젝트가 생성되었다고 보면 된다.  
따라서 여기서의 파이어스토어 서비스는 프로젝트에서 만들었던 파이어스토어와 일절 관련이 없다.

api를 보내보자.

![wp7-img8](/images/posts/webproject7-img8.png)

콘솔창을 보면 Functions 주소가 localhost:5001로 나와있는데, 여기로 요청을 보내면 된다.

요청 형식은

```
http://localhost:5001/(파이어베이스 프로젝트 아이디)/us-central1/(함수 이름)
```

여기서는 요청 보디에 값을 담으므로

![wp7-img9](/images/posts/webproject7-img9.png)

이렇게 http 통신 툴을 사용해 요청을 보내고 200 메세지가 오면 성공이다!

한번 확인해보자.  
위 에뮬레이터 콘솔에서 Firestore emulator-**Go to emulator**을 클릭하면

![wp7-img10](/images/posts/webproject7-img10.png)

파이어스토어 화면이 나온다. 문서가 성공적으로 등록되었다!

## 배포 및 응용형은 다음 시간에...

---

[^1]: 위 (request,response) 인자에서 정의한 변수명과 맞게-첫번째 것이 요청, 두번째 것이 응답 변수이다
