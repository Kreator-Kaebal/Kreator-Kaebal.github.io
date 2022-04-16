---
layout: post
title: 타입스크립트를 사용한 웹 프로젝트8
categories: [웹개발-타입스크립트 프로젝트]
tags: [java/typescript, firebase]
excerpt: 파이어베이스 클라우드 함수 응용
---

## 클라우드 응용

지난 [웹프로젝트 7번](http://kreator-kaebal.github.io/webproject7/)번에서는 파이어베이스 클라우드 함수를 사용하여 서버 구축 없이 api 서버를 제작하는 방법을 배웠봤다.
이 api 서버를 조금 더 체계적으로 만들어보자.

## node express와 통합

파이어베이스 클라우드 함수는 다양한 언어를 지원하는데, 만약 이 블로그처럼 Node.js(자바스크립트)를 사용하여 구현한다면 node express 문법으로도 제작할 수 있다.

node express가 무엇인지는 [여기](http://kreator-kaebal.github.io/webproject4)로  
한 줄 요약하면 자바스크립트로 서버도 만든다는 것이다.

파이어베이스 클라우드 함수 프로젝트의 functions 폴더에 들어간 뒤[^1], npm으로 express를 설치해주자.

```
npm install express
```

그 다음, functions 폴더의 **index.ts** 폴더에 접근해서 express 셋팅을 해줘야 한다.

```javascript
(import 구문 위치에)
import * as express from "express"
import * as cors from "cors"

const app = express();
const mycors = cors({ origin: true });

(initializeApp 위에)
app.use(mycors);
```

app.use(cors)는 CORS 에러[^2]를 해결하기 위해 express에서 제공해주는 장치로, 위 코드대로 하면 모든 주소에서 오는 요청을 받기 때문에 보안상 좋지 않지만 이 프로젝트는 배포할 게 아니므로 일단은 이대로 진행한다.

### 라우팅

그 다음, **라우팅**을 해 줘야 한다. 라우팅은 *api의 주소를 지정해주는 것*을 의미한다.  
예를 들어 tosso()라는 api 함수가 있고 이를 tissi로 라우팅하면,  
(서버 주소)/tissi 로 요청했을 때 tosso() 함수가 실행된다는 것이다.  

라우팅을 하기 위해서는 기존 api 함수를 수정해줘야 한다.

일단 함수에 있는 ```functions.https.onRequest...``` 등 firebase-functions와 관련된 요소들을 전부 제거해준 뒤 평범한 함수로 바꾼다.
[웹프로젝트 7번](http://kreator-kaebal.github.io/webproject7/)에서 만든 datastore 함수를 예로 들면,

```javascript
export const datastore = functions.https.onRequest((request,response) => ...)
```

를

```javascript
const datastore = async (request, response) => ...
```

으로 바꾸는 식이다.

이렇게 파이어베이스 api 함수를 평범한 함수로 바꾸면, express 변수를 사용해 이 함수를 실행할 주소를 지정해준다.  

```javascript
(함수 아래에)

app.post("/datastore", datastore);
```

이렇게 하면 기존 클라우드 함수 문법으로는 지정해주지 못했던 주소 엔드포인트와 요청 방식(get,post,...) 등도 마음대로 변경할 수 있다.

마지막으로, 이 함수를 파이어베이스 클라우드 함수로 감싸줘야 한다.

```javascript
(app.post 아래에)

module.exports = {
    v1: functions.https.onRequest(app)
}
```

이는 (app.get, app.post 등으로) app에 연결된 모든 함수를 **v1**이라는 엔드포인트에 묶어 파이어베이스 클라우드 함수에 넣겠다는 뜻이다.  
v1이라는 이름은 각자 원하는 것으로 교체해도 무방하다.

즉 datastore 함수를 요청하기 위해서는  
(서버 주소)/v1/datastore 이렇게 신청해야 하는 것이다.  

이렇게 한 뒤 배포해보고 엔드포인트에 맞춰 요청해본 뒤 정상적으로 응답이 오면 연동 성공이다!

### 미들웨어

node express는 **미들웨어**라는 독특한 기능을 제공한다.  
미들웨어는 간략하게 말해 api 요청과 응답 사이(middle)에서 작동하는 함수를 의미한다. 요청을 처리하기 이전 미들웨어 함수가 실행되기 때문에 미들웨어의 활용 가능성은 무궁무진하다.  
대표적으로 헤더 인증 같은 보안 처리를 들 수 있겠다. 미들웨어 함수가 api 실행하기 전 요청 변수를 가로채 거기에 있는 사용자 토큰(보통 헤더에 있다)을 인증한 다음 인증에 실패하면 바로 에러 응답을 날려버리는 식이다. 이러면 요청을 처리하지 않고도 사용자 인증을 할 수 있기 때문에 보안성과 재사용성[^3]이 향상된다.

미들웨어는 보통 index.js에서

```javascript
app.use(함수 형태)
```

로 만들 수 있다. 여기서는 간편하게 요청 보디에 타임스탬프[^4] 키를 추가하는 기능을 만들어보겠다.  

app.use(mycors) 아래에 또다른 app.use 함수를 추가해본다.  

```javascript
app.use((request, response, next) => {

    const { body } = req;

    const timestamp = Date.now();

    body.timestamp = timestamp;

    return next();

});
```

request, response 변수는 api 요청 시의 요청변수(request), 응답변수(response)를 가로챈다. 따라서 위와 같은 미들웨어를 만들면 어떤 api를 요청하던 요청 보디에 timestamp라는 키 값이 추가될 것이다.  

위 함수에는 요청, 응답 외에도 **next**라는 값이 눈에 띄는데, 이는 express에서 사용하는 **다음 단계로 이동한다**라는 뜻이다.  

무슨 말인지 잘 알아듣지 못하겠다. 예시를 들어 설명하자면

index.js가

```javascript
...
app.use(함수1)
app.use(함수2)

app.post(api함수)
...
```

이런 구조라고 하자. 함수 1과 함수 2는 정상 통과시에 모두 ```return next();```를 한다.

api함수 요청을 보내면(post) 함수1 미들웨어가 먼저 실행되어 요청을 가로챈다.  
만일 함수1이 정상적으로 통과되면 next()가 실행되는데, 이는 함수 1의 **아래**, 즉 **함수2 미들웨어**로 요청을 전달하겠다는 뜻이다.  

이렇게 함수 1에서 통과된 요청은 함수 2에서 한번 더 처리한다. 마찬가지로 정상적으로 통과되면 다시 next()가 되는데, app.use(함수2)의 아래는 app.post(api함수)이므로 그때서야 요청이 처리되는 것이다.  

즉 **위에서 아래로** 요청이 전달되는 것이라 보면 된다. 이는 응답도 마찬가지이다.

---
[^1]: 참고로 클라우드 함수 관련 터미널 조작(npm 등)은 무조건 **functions 폴더 내**에서 실행해주자. functions 폴더 내에 package.json이 있기 때문이다.
[^2]: CORS(Cross-Origin Resource Sharing)은 브라우저 단의 보안 장치로, 서버 외부(포트 포함)에서 요청이 오면 차단해버리는 기능을 뜻한다. 즉 CORS가 켜진 상태에서는 같은 포트의 서버 주소 안에서만 통신이 가능하다.
[^3]: 매 api 함수마다 인증 절차를 넣지 않아도 되니
[^4]: 요청 시간을 기록하는 변수