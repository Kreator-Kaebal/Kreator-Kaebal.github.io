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
이 과정은 서버까지 갈 필요 없이 클라이언트단에서 처리해야 한다.  

```LoginComponent.tsx```에 보면 **onClick=...getToken(id,pw)**이라는 미완성 함수가 있는데 이걸 구현한다.

```javascript
(임포트 구문에)
import { getAuth, getIdToken, onAuthStateChanged, signInWithEmailAndPassword } from "firebase/auth";

...

const [token, setToken] = useState('');

const getToken = (id,pw) => {
    const auth = getAuth();
    
    signInWithEmailAndPassword(auth, id, pw).then(() => {
    onAuthStateChanged(auth, async(currentUser) => {
        try{
        const token = await getIdToken(currentUser);
        navigate(`/admin/${currentUser.uid}`, {
            state: {
            userId: currentUser.email,
            token: token,
            },
        });
        return setToken(token);
        } catch(err){
        console.log(err);
        return setToken('토큰발급에 실패했습니다.')
        }
    });
    }).catch(err => {
    console.log(err);
    return setToken('사용자가 등록되지 않았거나 로그인에 실패했습니다.')
    });
}
```

[지난 시간](http://kreator-kaebal.github.io/webproject8)에 했던 클라이언트에서 uid 토큰받기의 _app.tsx useEffect 함수를 적절히 고친 것이다.

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

이번에는 회원가입이기 때문에 firebase/auth에서 signInWithEmailAndPassword 대신 CreateUserWithEmailAndPassword를 사용하였다.

회원가입에 성공하면(.then() 구문) **Router.push()**를 하는데 이건 next/router에서 제공하는 라우터 객체를 사용해 다른 페이지로 이동하겠다는 말이다. 참고로 프롭스도 넘겨줄 수 있는데, 자세한 설명은 [여기에서](https://nextjs.org/docs/api-reference/next/router)

참고로

```javascript
.then((userCredential) => {
                const {user} = userCredential;
                alert('회원가입에 성공하였습니다.');
                Router.push('/login');
            })
```

이렇게 then구문에서 리턴값 하나를 받을 수 있는데 이 리턴값의 .user에는 uid토큰을 포함한 사용자의 Authentication정보들이 담겨 있다.

![wp10-img5](/images/posts/webproject10-img5.png)

userCredential.user는 이렇게 생겼다.(지운 계정이기 때문에 허튼생각 하시지 마길)  
위에서는 eslint 때문에 const {user} = userCredential로 .user를 받았다.

어쨌든 회원가입을 해보면

![wp10-img6](/images/posts/webproject10-img6.png)

이렇게 실패했다는 알림창과 함께 오류가 나는데, 콘솔 찍어보니 비밀번호를 6자 이상으로 하라고 한다.  
아까 .catch()문으로 잡은 에러이므로, 추후 에러 핸들링을 구현하는 데 쓰일 수 있다.

![wp10-img7](/images/posts/webproject10-img7.png)

어쨌든 비밀번호를 6자 넘기고 다시 누르면 이렇게 성공했다고 알림이 뜨고 로그인으로 넘어간다.

![wp10-img8](/images/posts/webproject10-img8.png)

파이어베이스 웹콘솔에 들어가보면 회원이 등록되었다.

## 코드 리팩토링

로그인과 회원가입을 구현하였는데, 문제가 하나 생겼다.

기존의 페이지는 메인페이지(index.tsx)에 게시판 서비스를 구현해놓아 로그인이 아무 의미 없어진다. 사용자가 로그인 없이 바로 게시판 페이지에 들어갈 수 있기 때문이다.

따라서 프로젝트 구조를 조금 뜯어고쳐야 한다.

우선 index.tsx 코드를 통째로 이관시킨다. 본인은 **pages/tosso.tsx**에 붙여두었다.

그 다음 index.tsx를 지운다.

---
[^1]: next.js의 특징으로 pages 안에 있는 자바스크립트 파일들은 /(파일 이름)으로 바로 접근할 수 있다.
