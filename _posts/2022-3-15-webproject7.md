---
layout: post
title: 타입스크립트를 사용한 웹 프로젝트7
categories: [웹개발-타입스크립트 프로젝트]
tags: [java/typescript, firebase]
excerpt: 파이어베이스 클라우드 함수로 손쉽게 api서버 구현
---

## api 서버와 클라우드 함수

파이어베이스 클라우드 함수를 사용하면 서버를 구축하지 않고도 api를 구현할 수 있다.  
무슨 말이냐면, [지난번](https://kreator-kaebal.github.io/webproject4/)까지는 node express를 사용해 api 서버를 구현했다.  

이 서버는 클라이언트(여기서는 웹 페이지)에서 api 형태로 넘어오는 모든 요청을 꾸준히 처리해야 하니 24/7 내내 돌아가야 한다. 365일 24시간 내내 작동하니 이를 실행하는 컴퓨터는 보통 것보다 훨씬 높은 신뢰성이 필요하겠다. 간혹 어플 등에서 서버 점검 시간이라고 사용을 막아놓는 짓을 벌이는데, 이 점검 사항 중 하나가 바로 api 서버에 이상이 생겼는지, 즉 맛이 가서 요청을 안받는다던가 등을 체크하는 것이다.  
이렇게 api 서버를 남들이 요청할 수 있도록(로컬 환경이 아닌) 컴퓨터 상에서 실행하는 것을 **배포**라고 하며, 이 배포는 서버 전용 컴퓨터를 따로 장만하던가(**서버 컴퓨터**) 해야 한다. 물론 당직 데스크톱이나 노트북 등에서도 배포할 수는 있지만 위에서 말했듯이 계속 켜놓아야 하니까..보통은 전력효율 좋은 미니 컴퓨터를 사용한다. 초거대기업 등은 야예 서버 구동에만 최적화된 메인프레임이 있고...

서론이 길었다. 어쨌든 핵심은 **api 서버를 돌리는 컴퓨터를 어떻게든 장만해야** 한다! 하지만 당직 같은 개인들도 api 서버를 실행하려면 서버 컴퓨터가 필요하니, 어떻게 보면 금전 낭비이다. 그래서 현명한 사람들은 다음과 같은 잔머리를 굴렸다.

*우리 컴퓨터를 남들이 돈 내고 사용할 수 있도록 빌려주자*

그래서 나온 개념이 바로 **클라우드 컴퓨팅**이다. 회사에서 컴퓨터를 무지막지하게 많이 사들인 후, 이를 고객들이 원격으로 쓸 수 있도록 판매하는 것이다.  
요금 측정도 사용한 만큼만 이뤄지니, 고객들에게는 컴퓨터 사는 비용과 전기료던가 등등..을 아낄 수 있으니 상전벽해나 다름 없었다! 서버로 보낸 요청이 탈취되는(후킹) 등의 보안 문제도 회사가 다 책임져 주니 나는 다른거 신경쓸 필요 없이 요청만 만들면 된다인 것이다.  
놀라운 것은 이 개념이 불과 15여년 전에 나왔다. 당시 일개 전자상거래 벤처기업에 불과했던 아마존(Amazon) 사가 시가총액 5위로 급성장할 수 있었던 까닭도 바로 이 클라우드 컴퓨팅 시장을 선점해서 그렇다.  

또 서론이 길어지니...진짜 요점만 말하자면 이번에 설명할 **파이어베이스 클라우드 함수**도 클라우드 컴퓨팅의 일종이다. 정확히는 api 함수 전용 클라우드 서비스이다.  
이게 무엇이라 함은, *파이어베이스*(구글)의 컴퓨터를 빌린 다음, *함수 형태*로 제작한 api를 그곳에 배포해 서버 구축 없이도 api를 사용할 수 있게 하는 서비스이다.  

클라우드 컴퓨팅도 사실 여러 종류가 있는데, 이것도 따로 설명할 기회가 있으면 설명하겠다.  
이번 시간에는 파이어베이스 클라우드 함수로 api 서버를 구현해보자.

## 파이어베이스 클라우드 프로젝트 생성

[이 공식문서를 참조](https://firebase.google.com/docs/functions/get-started?hl=ko)

### 구글 CLI 설치

파이어베이스 클라우드 함수를 사용하기 위해서는 먼저 파이어베이스 프로젝트 생성이 필요하다.  
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

### 초기 설정

```javascript
import * as functions from 'firebase-functions';
import * as admin from 'firebase-admin';

import Ajv from 'ajv';
import { custmemberschema } from './interfaces/custmemberschema';

admin.initializeApp();
```

index.ts 파일은 서버를 실행시키면 가장 먼저 실행되는 파일이다.  
물론 이 프로젝트는 클라우드 함수이기 때문에 실제 배포시에는 클라우드상에서 서버가 알아서 돌아가지만,  
테스트시에는 서버가 필요하기 때문이다.

어쨌든 functions와 admin을 불러와서 admin.initializeApp()으로 서버를 초기화한다.

이 때, 파이어베이스 어드민(서버)의 initializeApp()은 그냥 파이어베이스(클라이언트)의 initializeApp()과 설정하는 법이 다르다.  

어드민의 initializeApp 설정을 가져오기 위해서는

![wp7-img12](/images/posts/webproject7-img12.png)

웹 콘솔에서 **톱니바퀴-프로젝트 설정-서비스 계정**으로 들어가야 한다.  
여기서 **Firebase Admin SDK**를 클릭한 다음 **새 비공개 키 생성**을 누르면 json 파일 하나가 다운로드 받아진다.  

이 파일을 서버 폴더 안에 넣은 다음 위의 코드를 복사해 붙여넣으면 된다.  
물론 firebase-admin 임포트는 했으니 빼고 "path/to/..."는 json 파일 경로로 바꿔주는 것은 당연하다.  
주의할 점은 이 파일은 보안파일이라 github 배포 등을 할때 같이 푸쉬하지 말도록 하자.

여기서는 타잎스크립트를 사용하므로 문법을 적절하게 바꿔

```javascript
...
import * as serviceAccount from "(json이 저장된 경로-파일명 확장자까지 포함하여)";
...

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount as admin.ServiceAccount),
  databaseURL: "https://(프로젝트 아이디).firebaseio.com"
});
...
```

현재는 설정에 databaseURL(파이어스토어 주소)만 들어가 있지만, 파이어베이스 스토리지같이 다른 기능들도 추가할 수 있다.  

참고로 위처럼 했는데 json 임포트가 오류나는 분은 **tsconfig.json**에  

```json
{
  "compilerOptions": {
    ...
    "resolveJsonModule": true,
  },
  ...
}
```

compilerOptions에 resolveJsonModule 옵션을 true로 추가해주면 된다.

### 간단한 예제

간단하게 요청 보디를 확인한 후 파이어스토어에 넣는 api를 제작하겠다.

위에서 Ajv와 custmemberschema를 임포트했는데, Ajv는 요청 body 형식을 확인하는 npm 모듈로, 일단은 신경쓰지 말자.  
custmemberschema는 사전에 정의해놓은 body 형식이다.

그 다음 아래에서 함수를 생성하는데...

```javascript
export const datastore = functions.https.onRequest(async (request, response) => {
  //const memkeys: Array<string> = Object.getOwnPropertyNames(request.body);
  try {
    const data = Object(request.body);
    let check = ajv.validate(custmemberschema, data);

    if (check) {
      await admin.firestore().collection('test').add(data);
      response.status(200).json({ message: '파이어스토어 저장이 완료되었습니다.' });
    } else {
      response.status(400).json({ message: '요청 body 형식을 다시 확인하세요.' });
    }
  } catch (error) {
    response.status(500).json({
      message: '서버와의 연결이 끊어졌습니다.',
      error: error,
    });
  }
});
```

대충 request body에 담긴 내용을 test라는 파이어스토어 콜렉션 문서로 추가한다는 것이다.  
ajv로 요청형식 확인하고 싶지 않으면 check구문을 과감히 빼버리자.

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

## 함수 배포

에뮬레이터는 언제까지 당직 컴퓨터(로컬)에서 임시로 사용하는 것이므로, 실제 api를 사용 가능하게 만드려면 **배포**를 해야 한다.  
배포는 프로젝트를 파이어베이스 클라우드 상에 올리는 것을 뜻한다. 이렇게 올라간 프로젝트는 클라우드 서버에 컨테이너 형태로 변환되어 자동으로 실행된다.  
잘 이해가 되지 않겠지만, 프로젝트 코드를 클라우드 서버에 있는 가상머신에 올려 24시간 돌린다고 보면 편하다.
한번 api를 배포해보자.

### 배포 방법

구글 CLI를 사용하면 편하다.  
클라우드 함수 프로젝트 폴더/functions에 들어가서 콘솔창을 연 다음,

```
firebase deploy --only functions
```

을 입력한다. 파이어베이스 프로젝트(웹상에 있는 그것)에 클라우드 함수 프로젝트를 올리겠다는 의미이다.

(입력 후 에러가 나오면 보통 ESLint 에러이다.)

```
npm install -d eslint-config-prettier
```

를 해주고, 그래도 에러가 나오면 에러 로그를 보며 ESLint 규정에 맞게 코드를 고치자.

그렇게 ESLint 에러를 해결했는데...

```
Error: Your project test-1017c must be on the Blaze (pay-as-you-go) plan to complete this command. Required API cloudbuild.googleapis.com can't be enabled until the upgrade is complete. To upgrade, visit the following URL:
```

최종 보스가 등장하였다.  
다름 아닌 요금제 문제이다. 즉 배포하려면 돈 내라 이말이다.  
당직은 테스트 프로젝트에 요금을 부과할 수는 없으므로 패쓰하겠다.

어쨌든 돈을 내던지 해서 배포가 성공하면...

![wp7-img11](/images/posts/webproject7-img11.png)

(위 사진은 요금 낸 프로젝트이다.)  
이렇게 프로젝트 웹 콘솔에서 왼쪽 Functions를 클릭하면 배포되어 클라우드에서 실행중인 파이어베이스 함수를 볼 수 있다.  
① 번 항목은 함수 종류, ②번 항목은 api 주소이다.  
이제 로컬호스트 대신 ②번의 주소로 요청하면 클라우드 서버와 통신하는 것이다!

---

[^1]: 위 (request,response) 인자에서 정의한 변수명과 맞게-첫번째 것이 요청, 두번째 것이 응답 변수이다
