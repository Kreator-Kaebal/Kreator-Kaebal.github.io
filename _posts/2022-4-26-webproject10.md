---
layout: post
title: 타입스크립트를 사용한 웹 프로젝트10
categories: [웹개발-타입스크립트 프로젝트]
tags: [java/typescript, firebase]
excerpt: 배운 것들을 응용하여 로그인 기능 구현하기
---

## 마지막 시간

타잎스크립트 웹프로젝트에만 너무 시간을 쏟는 듯 하여 최종적으로(?) 로그인 구현을 구현하고 본 게시판 프로젝트를 끝마치기로 한다.  
로그인-게시물-댓글 3가지 기능만 있어도 그럴싸하게 보이기 때문이다?

## 로그인

로그인은 저번에 말했던것처럼 파이어스토어의 Authentication 기능을 활용하면 된다.  
그러면 사용자를 어떻게 등록하느냐? 이것도 firebase-admin에 다 있다.  

클라이언트에서 pages 폴더에 **login.tsx**, components에 **LoginComponent.tsx**를 만들어주자. 각각 다음과 같이 작성한다.  

src/components/LoginComponent.tsx

```javascript
import { useState } from 'react';
import styles from '../../styles/Login.module.css';

function Login() {
 const [id, setId] = useState('');
 const [pw, setPw] = useState('');

 return (
  <div>
   <div className={styles.loginContainer}>
    <div className={styles.loginTitle}>로그인하세요</div>
    <table>
     <thead />
     <tbody>
      <tr>
       <td>
        <input
         placeholder="Email" onChange={(e) => {
          return setId(e.target.value);
         }} />
       </td>
       <td rowSpan={2}>
        <button onClick={() => {
          return getToken(id, pw);
        }}>
         제출
        </button>
       </td>
      </tr>
      <tr>
       <td>
        <input type="password" placeholder="PW" onChange={(e) => {
          return setPw(e.target.value);
         }}
        />
       </td>
      </tr>
     </tbody>
    </table>
   </div>
  </div>
 );
}

export default Login;
```

pages/login.tsx

```javascript
import LoginComponent from '../src/components/LoginComponent';

const Login = () => {
  return <LoginComponent />;
};

export default Login;

```

이렇게 하면 ```(주소)/login``` 으로 접속하였을 때 로그인 콤포넌트가 뜬다.[^1]

![wp10-img1](/images/posts/webproject10-img1.jpeg)

이렇게

이제 로그인 버튼(*제출*)을 눌렀을 때 회원정보가 Authentication에 있는지 확인하고 있으면 유저(uid) 토큰을 발급받는 함수를 작성한다.  
이 과정은 서버까지 갈 필요 없이 클라이언트단에서 처리하면 된다.  

```LoginComponent.tsx```에 보면 **onClick=...getToken(id,pw)**이라는 미완성 함수가 있는데 이걸 구현한다.

```javascript
(임포트 구문에)
import { getAuth, getIdToken, onAuthStateChanged, signInWithEmailAndPassword } from "firebase/auth";

...

const [token, setToken] = useState('');

const getToken = (id,pw) => {
    const auth = getAuth();
    
    signInWithEmailAndPassword(auth, id, pw)
    .then(() => {
        return Router.push({
            pathname: `/tosso`,
        });
    })
    .catch((err) => {
        console.log(err);
        alert('사용자가 등록되지 않았거나 로그인에 실패했습니다.');
    });
};
```

[지난 시간](http://kreator-kaebal.github.io/webproject8)에 했던 클라이언트에서 uid 토큰받기의 _app.tsx useEffect 함수를 적절히 고친 것이다. 대충 설명하자면

* ```getAuth()```로 파이어베이스 인증 상태 변수를 가져온다.
* ```signInWithEmailAndPassword()```에 이 인증변수,아이디,비밀번호를 집어넣어 로그인을 시도한다.
* 로그인이 성공하면 인증변수가 해당 로그인 상태로 변경될 것이다.
* 위 로그인 기능은 콜백함수를 지원하는데 따라서 성공시 /tosso 페이지로 이동하고, 실패시(아이디 틀렸다던가...) 에러 출력한다.

참고로

```javascript
.then((userCredential) => {
                const {user} = userCredential;
                alert('로그인에 성공하였습니다.');
                Router.push('/tosso');
            })
```

이렇게 콜백함수에서 리턴인자 하나를 받을 수 있는데 이 리턴값의 .user에는 uid토큰을 포함한 사용자의 Authentication정보들이 담겨 있다.

![wp10-img2](/images/posts/webproject10-img2.png)

이제 _app.tsx의 이건 지운다.

한번 콤포넌트 아래에 ```<div>{token}</div>```같은 식으로 토큰 출력을 해보기로 하자.

![wp10-img3](/images/posts/webproject10-img3.png)
![wp10-img4](/images/posts/webproject10-img4.png)

아이디와 비밀번호가 Authentication에 있는가에 따라 다음과 같이 나온다.

## 회원가입

로그인 하고 다음은 일단 미뤄두고(그전에 배워야할 게 있다) 로그인이 있으면 회원가입도 있어야 하니 회원가입도 만들자.  

로그인과 똑같이 pages에 회원가입 페이지를 만들지만, 회원가입은 굳이 다른 페이지에서 화면을 재사용할 일이 없으므로 콤포넌트를 만들지 않는다.

여기서는 pages/**signup.tsx**라는 이름으로 생성하겠다.

```javascript
import { useState } from 'react';
import { createUserWithEmailAndPassword, getAuth } from 'firebase/auth';
import Router from 'next/router';

const Signup = () => {
    const [id, setId] = useState('');
    const [pw, setPw] = useState('');

    const auth = getAuth();

    const signUp = (id, pw) => {
        createUserWithEmailAndPassword(auth, id, pw)
        .then(() => {
            alert('회원가입에 성공하였습니다.');
                Router.push('/login');
            })
        .catch((error) => {
            console.log(error);
            alert('회원가입에 실패하였습니다.');
        });
    };

    return (
        <div>
            <div>회가하세요</div>
            <table>
                <thead />
                <tbody>
                    <tr>
                        <td>
                            <input
                                placeholder="Email"
                                onChange={(e) => {
                                return setId(e.target.value);
                                }}
                            />
                        </td>
                        <td rowSpan={2}>
                            <button
                                onClick={() => {
                                return signUp(id, pw);
                                }}>
                                제출
                            </button>
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <input
                                type="password"
                                placeholder="PW"
                                onChange={(e) => {
                                return setPw(e.target.value);
                                }}
                            />
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    );
};

export default Signup;
```

이번에는 회원가입이기 때문에 firebase/auth에서 signInWithEmailAndPassword 대신 createUserWithEmailAndPassword를 사용하였다.

회원가입에 성공하면(.then() 구문) **Router.push()**를 하는데 이건 next/router에서 제공하는 라우터 객체를 사용해 다른 페이지로 이동하겠다는 말이다. 참고로 프롭스도 넘겨줄 수 있는데, 자세한 설명은 [여기에서](https://nextjs.org/docs/api-reference/next/router)

참고로 createUser...도 마찬가지로 콜백에서 사용자 인증정보를 받을 수 있는데...

![wp10-img5](/images/posts/webproject10-img5.png)

정보의 .user 구문은 이렇게 생겼다.(지운 계정이기 때문에 허튼생각 하시지 마길)

어쨌든 회원가입을 해보면

![wp10-img6](/images/posts/webproject10-img6.png)

이렇게 실패했다는 알림창과 함께 오류가 나는데, 콘솔 찍어보니 비밀번호를 6자 이상으로 하라고 한다.  
아까 .catch()문으로 잡은 에러이므로, 추후 에러 핸들링을 구현하는 데 쓰일 수 있다.

![wp10-img7](/images/posts/webproject10-img7.png)

어쨌든 비밀번호를 6자 넘기고 다시 누르면 이렇게 성공했다고 알림이 뜨고 로그인으로 넘어간다.

![wp10-img8](/images/posts/webproject10-img8.png)

파이어베이스 웹콘솔에 들어가보면 회원이 등록되었다.

## 코드 리팩토링과 페이지 라우팅

로그인과 회원가입을 구현하였는데, 문제가 하나 생겼다.

기존의 페이지는 메인페이지(index.tsx)에 게시판 서비스를 구현해놓아 로그인이 아무 의미 없어진다. 사용자가 로그인 없이 바로 게시판 페이지에 들어갈 수 있기 때문이다.

따라서 프로젝트 구조를 조금 뜯어고쳐서 메인→로그인→게시판 으로 가는 시스템을 구현해야 한다.  

우선 index.tsx 코드를 통째로 이관시킨다. 본인은 **pages/tosso.tsx**에 붙여두었다.

그 다음 index.tsx 코드를 지우고 다음과 같이 변경한다. 일단 사용할 임시 메인페이지이다.

```javascript
import Link from 'next/link';

const Home = () => {
    return (
        <div>
            <div>
                <h1>게시판 서비스에 오신 것을 환영합니다</h1>
            </div>
            <div>
                <Link href="/login">
                    <button>로그인하기</button>
                </Link>
            </div>
        </div>
    );
```

next.js에서는 **Link**라는 태그를 사용하여 페이지 이동을 할 수 있다. 리액트의 Route라고 보면 된다. 위처럼 하면 ```로그인하기```를 누를 시 ```(주소)/login```로 이동한다.

![wp10-img9](/images/posts/webproject10-img9.png)

직접 확인해보자

그다음 로그인 콤포넌트를 수정해주자.

```javascript
(useState 임포트 구문 아래에)
import Link from 'next/link';
...
(render구문의 비밀번호 input태그 구문 아래에)
<tr>
    <td>
        <input
            type="password"
            placeholder="PW"
            onChange={(e) => {
                return setPw(e.target.value);
            }}
        />
    </td>
    <td>
        <Link href="/signup">
            <button>회원가입하기</button>
        </Link>
    </td>
</tr>
...
```

이렇게 회원가입 페이지로 이동해주는 버턴을 만들어주고  
로그인 함수(getToken)에서 성공시 게시판이 있는 ```/tosso```로 이동해주면 된다...?

조금 색다르게 접근해보자. 만약 해커가 이 페이지 구조를 알고 있어 로그인 없이 이 프로젝트 구조를 파악하고 있다면 주소창에 /tosso를 쳐서 로그인을 건너뛸 수 있다!  
당연하게 보안상 매우 큰 취약점이다. 다행히 이를 해결하는 방법도 파이어베이스에서 제공해준다.

## 로그인 상태 처리

이전 index.tsx 코드를 이관시킨 페이지(여기서는 **tosso.tsx**)로 이동한 다음 아래와 같이 함수를 추가한다.

```javascript
...
import Router from 'next/router';
import { useEffect, useState } from 'react';
import { getAuth, onAuthStateChanged, signOut } from 'firebase/auth';
...

...
const [logined, setLogined] = useState(false);
const [email, setEmail] = useState('');

const auth = getAuth();

const logout = async () => {
    try {
        await signOut(auth);
        return Router.push({
            pathname: '/',
        });
    } catch (err) {
        return console.log(err);
    }
};

useEffect(() => {
    onAuthStateChanged(auth, (user) => {
        if (user) {
            setEmail(user.email);
            return setLogined(true);
        }
        return Router.push({
            pathname: '/login',
        });
    });
});
...
```

로그아웃 함수와 useEffect 함수 하나를 추가해줬다. 둘 다 firebase/auth에 있는 기능을 활용하였다.

로그아웃 과정은...

* 현재 파이어베이스 인증값을 받아온 다음(```getAuth()```)
* ```signOut()```에 인증값을 넣어 로그아웃하면
* 콜백(async/await을 사용하던가, .then()을 사용하던가...)으로 성공시 메인페이지('/')로 이동, 실패시 에러출력

그다음 useEffect() 구문이 중요한데, ```onAuthStateChanged()```는 위 인증변수를 인자로 받아 상태가 바뀌었을시(로그인/아웃 등) 콜백함수를 호출하는 함수이다.
따라서 useEffect를 사용해 인증상태가 바뀌면 onAuthStateChanged 콜백함수가 실행되고, 콜백함수에는 그떄의 사용자 정보(user)가 인자로 담겨온다.

만약 사용자 정보가 있다면(```if (user)```, 즉 **로그인된 상태**라면) 이메일값(email)과 로그인 상태(logined) state변수를 맞게 바꿔준다.  
그리고 없으면 Router push를 통해 로그인 페이지로 이동한다.

이러면 이제 ```/tosso```로 접근했을때 로그인되어있지 않은 상태라면 자동으로 로그인 페이지로 이동한다!

### 콤포넌트 변경

그럼에도 불구하고 tosso 페이지에서 로그인되어있지 않은 상태라면 게시판이 보인다.

```javascript
...
return(
    {logined ? (<div>(원래 jsx문)</div>) : (<div />)}
)
```

이렇게 logined state변수가 false이면 빈 페이지를 띄우도록 삼항연산자로 감싸주자.  
참고로 삼항연산자를 사용할 시 jsx문들은 **괄호**로 감싸줘야 하고 (원래 jsx문)은 ```<div>```같은 태그로 감싸줘야한다.

그럼 email state변수는 어디다 쓰이나?
tosso 페이지의 콤포넌트(```src/components/TossoDetails.tsx```와 ```TossoOperations.tsx```)를 수정하여 글 작성시 글쓴이가 나오도록 하자.

일단 ```tosso.tsx```에서

```javascript
...
<TossoOperations
    getSingleTosso={getSingleTosso}
    email={email}
/>
...
```

이렇게 email변수를 프롭스로 넣어준 다음

```TossoOperations.tsx```에서

```javascript
...
function TossoOperations({ getSingleTosso, email }) {
    ...
    function saveTosso() {
        ...
        // addDoc은 콜렉션(dbInstance)에 문서 추가
        return addDoc(dbInstance, {
            // title 필드 생성후 값을 tossoTitle로, desc 필드 생성후 값을 tossoDesc로
            title: tossoTitle,
            desc: tossoDesc,
            author: email,
            edited: Date.now(),
            // 문서 추가가 완료되면 콜백작동
        })
        ...
    }
...
```

이렇게 파이어스토어 저장시 이메일과 현재 작성일자도 문서에 넣어준다.

그 다음 ```TossoDetails.tsx```로 이동하여

```javascript
...
return (
    <>
        
        <h2>{(singleTosso as any).title}</h2>
        <div
            dangerouslySetInnerHTML={{
                __html: (singleTosso as any).desc,
            }}
        />
        <i>
            {(singleTosso as any).author}{' '}
            {(singleTosso as any).edited}
        </i>
        ...
        
)
```

이렇게 이메일과 작성일자도 보여지도록 하면 끝이다!

## 완성본

https://github.com/kaebalsaebal/pseudo-website

[여기에서 작성한 게시판 서비스](https://kreator-kaebal.github.io/webproject2)의 버그를 수정한 완성본이다.

* 문서 없을때 수정/삭제/댓글달기 비활성화(클릭시 에러알림)
* 제목이나 내용 둘중 하나라도 입력안하면 저장 못하도록

```npm install```로 필요 모듈 설치후 ```public/firebase-messaging-sw.js```와 ```src/firebase/firebaseConfig.ts``` 파일을 본인 파이어베이스 설정으로 바꿔 작성해주자. 어떻게 작성하는가는 본 블로그의 *웹프로젝트2*, *웹프로젝트3* 에 나와있다.

---
[^1]: next.js의 특징으로 pages 폴더 안에 있는 자바스크립트 파일들은 /(파일 이름)으로 라우팅 없이 바로 접근할 수 있다. 참고로 세부 라우팅((주소)/A/B 이런식)을 하려면 pages에다 A라는 폴더를 만든 다음, 그 안에 B 파일을 생성하면 된다.
