---
layout: post
title: 타입스크립트를 사용한 웹 프로젝트9
categories: [웹개발-타입스크립트 프로젝트]
tags: [java/typescript, firebase]
excerpt: 배운 것들을 응용하여 api 사용자 인증하기
---

## 지난 시간에

[지난번](http://kreator-kaebal.github.io/webproject8/)에 파이어베이스 클라우드 함수를 node express 형태로 만드는 것과, 이를 활용해 클라우드 함수를 라우팅하고 미들웨어 처리하는 방법을 배웠다.

이렇게 배운 기능들을 응용하여 게시판 api에 보안 기능을 추가해보자.  
가장 기본적이지만 중요한 보안으로, **파이어베이스에 등록된 사용자만** api 요청을 할 수 있도록 한다.  

## 파이어베이스 회원 등록

파이어베이스는 **Authentication**이라는 회원 관리 기능이 있다. 파이어베이스를 사용하는 앱(iOS,Web,Android 등...)의 회원 가입/로그인 기능을 구현할 때 요긴하게 쓰인다.  

이 Authentification도 파이어베이스 어드민 api로 제공되기 때문에 원래는 코드를 사용하여 회원가입 api를 구현해야 하겠지만, 시간상 회원가입 기능은 이 다음에 하기로 한다. 웹 콘솔을 사용하여 회원을 바로 등록할 수 있기 때문이다.

![wp9-img1](/images/posts/webproject9-img1.png)

웹 콘솔[^1]에 접속하여 게시판 앱 프로젝트를 선택한 뒤, 왼쪽에서 ```Authentication``` 을 클릭한다.

![wp9-img2](/images/posts/webproject9-img2.png)

```Sign-in method``` 메뉴에 들어가면 위와 같은 화면이 뜬다.  
'로그인 제공업체'는 이 앱에서 사용가능한 로그인 수단을 의미하는데,  
*이메일/비밀번호*는 이메일로 로그인하는 것, *Google*은 구글 계정으로 로그인하겠다는 것을 의미한다.

![wp9-img3](/images/posts/webproject9-img3.png)

이렇게 '새 제공업체 추가'로 로그인 수단을 추가해주어 우리가 흔히 접하는 '타 계정 연동 로그인'을 구현할 수 있다.  

우리는 구글 연동 로그인 따윈 필요없으므로 이메일/비밀번호 로그인만 활성화시키자.  
항목을 클릭하면

![wp9-img4](/images/posts/webproject9-img4.png)

여기에서 맨 위 ```이메일/비밀번호 사용 설정```을 켜준다.  
참고로 *이메일 링크*는 비밀번호 대신 매번 이메일 인증으로 로그인하는 기능이다.

![wp9-img5](/images/posts/webproject9-img5.png)

그 다음 ```Users``` 메뉴로 들어가면 ```사용자 추가``` 버튼이 생기는데, 이제 사용자를 등록할 수 있다. 이때 이메일과 비밀번호를 잘 기억해두자.  
등록하면 Users 목록에 사용자가 추가된다.

### 클라이언트에서 사용자 토큰 생성

이 내용은 [fcm 메세지 전송](http://kreator-kaebal.github.io/webproject3/) 때의 토큰 생성 방식과 유사하다. 토큰(Token)이라는 것은 클라이언트 구분을 위한 **임시 고유값** 이기 때문이다.[^2]

클라이언트에서 매번 웹페이지 초기화 때마다 토큰을 받는 useEffect 함수를 추가해주자.

```javascript
(_app.tsx같이 페이지 로딩될때마다 실행되는 함수에)
...
import {getAuth, getIdToken, onAuthStateChanged, signInWithEmailAndPassword} from 'firebase/auth';
...
useEffect(function () {
    const auth = getAuth();
    signInWithEmailAndPassword(
        auth,
        '(아까 이메일)',
        '(아까 비밀번호)',
    )
        .then((userCredential) => {
            console.log(userCredential.user);
        })
        .catch((err) => {
            console.log(err);
        });

    onAuthStateChanged(auth, async (user) => {
        try {
            const result = await getIdToken(user);
            console.log(`user token is ${result}`);
        } catch (err) {
            console.log(err);
        }
    });
}, []);
...
```

이렇게 함수를 만들면 된다.  
간단히 말해 파이어베이스 현재 권한을 받아(getAuth) 파이어베이스에 로그인하여(signInEmailAndPassword) 권한을 변경하고, 그것이 감지되면(onAuthStateChanged) 토큰을 받아(getIdToken) 출력하는 것이다.

![wp9-img6](/images/posts/webproject9-img6.png)

실행해보면 이렇게 콘솔창에 user token is ...라고 사용자 토큰이 출력된다.

---
[^1]: console.firebase.google.com/...으로 시작하는 사이트
[^2]: 세부적으로 fcm 토큰은 **파이어베이스 앱을 구분**, 여기서 만들 토큰은 **파이어베이스 앱 내의 사용자를 구분**한다는 차이점이 있다.
