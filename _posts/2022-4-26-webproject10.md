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

이렇게 하면 ```(주소)/login``` 으로 접속하였을 때 로그인 콤포넌트가 뜬다.

![wp10-img1](/images/posts/webproject10-img1.jpeg)

이렇게

이제 로그인 버튼(*제출*)을 눌렀을 때 회원정보가 Authentication에 있는지 확인하고 있으면 유저(uid) 토큰을 발급받는 함수를 작성한다.  
이 과정은 서버까지 갈 필요 없이 클라이언트단에서 처리해야 한다.  

```LoginComponent.tsx```에 보면 **getToken(id,pw)**이라는 미완성 함수가 있는데 이걸 구현한다.

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

