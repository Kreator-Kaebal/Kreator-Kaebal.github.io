---
layout: post
title: 타입스크립트를 사용한 웹 프로젝트2
categories: [웹개발-타입스크립트 프로젝트]
tags: [java/typescript, react/nextjs, firebase]
excerpt: 게시판 서비스 만들기-기본 설정과 firebase 연동까지
---

## Next.js와 파이어베이스 연동

이번 시간에는 구글에서 제공하는 모바일 및 앱 개발 api[^1] 서비스인 파이어베이스를 Next.js에 연동하여 간단한 메모장 서비스를 구현해보겠다.

[이 사이트를 참고하여 제작했음을 밝힌다](https://www.freecodecamp.org/news/nextjs-firebase-tutorial-build-an-evernote-clone/)

## 준비물

typescript 템플릿으로 설치한 next.js 프로젝트

```
npx create-next-app 프로젝트이름 --typescript
```

eslint 설정

```
npx eslint --init
```

> ? How would you like to use ESLint?  
> **To check syntax, find problems, and enforce code style**  
> ? What type of modules does your project use?  
> **JavaScript modules (import/export)**  
> ? Which framework does your project use?  
> **React**  
> ? Does your project use TypeScript?  
> **Yes**  
> ? Where does your code run?  
> **Browser**  
> ? How would you like to define a style for your project?  
> **Use a popular style guide**  
> ? Which style guide do you want to follow?  
> **Airbnb: <https://github.com/airbnb/javascript>**  
> (이거는 취향 차이긴 한데, 보통 Airbnb를 가장 많이 사용한다.)  
> ? What format do you want your config file to be in?  
> **JSON**  
> ? Would you like to install them now with npm?  
> **Yes**

초기설정 후

```javascript
module.exports = {
  env: {
    browser: true,
    es2021: true,
  },
  extends: ['plugin:react/recommended', 'airbnb', 'airbnb-typescript', 'plugin:prettier/recommended'],
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: ['./tsconfig.eslint.json', './tsconfig.json'],
    tsconfigRootDir: __dirname,
    ecmaFeatures: {
      jsx: true,
    },
    ecmaVersion: 'latest',
    sourceType: 'module',
  },
  plugins: ['react', '@typescript-eslint'],
  rules: {
    'arrow-body-style': ['error', 'always'],
    'jsx-a11y/anchor-is-valid': 0,
    'react/button-has-type': 0,
    'react/function-component-definition': ['off'],
    'react/react-in-jsx-scope': 0,
    'react/prefer-stateless-function': 0,
    'react/jsx-no-bind': 0,
    'react/jsx-no-useless-fragment': 0,
    'react/jsx-one-expression-per-line': 0,
    'react/jsx-props-no-spreading': 0,
    'no-nested-ternary': 0,
    'no-shadow': 'off',
    '@typescript-eslint/no-shadow': 'off',
    'no-use-before-define': ['off'],
    'react/jsx-filename-extension': [
      2,
      {
        extensions: ['.js', '.jsx', '.ts', '.tsx'],
      },
    ],
    'import/extensions': [
      2,
      'ignorePackages',
      {
        js: 'never',
        jsx: 'never',
        ts: 'never',
        tsx: 'never',
      },
    ],
    'prettier/prettier': 'error',
  },
  settings: {
    'import/resolver': {
      node: {
        extensions: ['.js', '.jsx', '.ts', '.tsx'],
        moduleDirectory: ['node_modules', '@types'],
      },
      typescript: {},
    },
  },
};
```

으로 .eslintrc.js 설정  
대충 eslint 에라가 발생하지 않도록 규칙을 설정한것이다.  
참고로 eslint가 귀찮으면 이 과정을 빼고 해도 된다. 물론 아래 prettier도 마찬가지

프로젝트 폴더에 tsconfig.eslint.json 추가

```json
{
  "include": [".eslintrc.js"]
}
```

tsconfig.json의 include 항목에 `.eslintrc.js` 추가

prettier 설정

```
npm -i prettier eslint-config-airbnb-typescript eslint-config-prettier eslint-plugin-prettier -D
```

후 .prettier 작성

```json
{
  "singleQuote": true,
  "semi": true,
  "useTabs": true,
  "tabWidth": 4,
  "trailingComma": "all",
  "printWidth": 80,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

settings.json 설정(VSCode)

```json
...
"editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "eslint.alwaysShowStatus": true,
  "[javascript]": {
    "editor.formatOnSave": false
  },
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "typescript.suggest.paths": false,
  "javascript.suggest.paths": false,
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ]
}
```

맨 아래에 다음과 같은 줄 추가

프로젝트에서 사용할 모듈 설치

```
npm i firebase react-quill@beta
```

firebase는 당연히 파이어베이스 사용에 필요하고, react-quill은 텍스트 에디터이다.  
리액트 17.0.2 환경에서 그냥 react-quill는 설치 에라가 나기 때문에 @beta를 붙여주자.

따라서 package.json의 디펜던시가

```json
  "dependencies": {
    "firebase": "^9.6.7",
    "next": "12.1.0",
    "react": "17.0.2",
    "react-dom": "17.0.2",
    "react-quill": "^2.0.0-beta.4",
    "sass": "^1.49.9"
  },
  "devDependencies": {
    "@types/node": "17.0.21",
    "@types/react": "17.0.39",
    "@typescript-eslint/eslint-plugin": "^5.13.0",
    "@typescript-eslint/parser": "^5.13.0",
    "eslint": "^8.10.0",
    "eslint-config-airbnb": "^19.0.4",
    "eslint-config-airbnb-typescript": "^16.1.0",
    "eslint-config-next": "12.1.0",
    "eslint-config-prettier": "^8.5.0",
    "eslint-plugin-import": "^2.25.4",
    "eslint-plugin-jsx-a11y": "^6.5.1",
    "eslint-plugin-prettier": "^4.0.0",
    "eslint-plugin-react": "^7.29.3",
    "eslint-plugin-react-hooks": "^4.3.0",
    "prettier": "^2.5.1",
    "typescript": "4.6.2"
  }
```

이렇게 되어있으면 정상이다.

## 메인페이지 수정

프로젝트 폴더의 **pages/index.tsx**를 다음과 같이 수정한다.

```javascript
import Head from 'next/head';
import styles from '../styles/Home.module.css';

const Home = () => {
  return (
    <div className={styles.container}>
      <Head>
        <title>Create Tosso App</title>
        <meta name="description" content="Generated by create tosso app" />
        <link rel="icon" href="/favicon.ico" />
      </Head>

      <main>
        <div className={styles.container}>
          <div className={styles.left}>left</div>
          <div className={styles.right}>right</div>
        </div>
      </main>
    </div>
  );
};
```

리액트나 뷰 같은 웹 프레임웤는 **콤포넌트** 단위로 페이지를 관리한다.  
정확히는 페이지보다는 한 페이지에서 표시하는 html들을 **객체**처럼 묶어 관리하는 것이다.  
네이버 메인페이지의 검색창, 뉴스창, 광고창 등이 다 각각 콤포넌트로 구성되어 있다 생각하면 된다.  
콤포넌트의 리턴값은 html[^2]이고, 클래스 불러오듯이 다른 콤포넌트를 불러와 **태그** 형태로 넣어줄 수 있다.  
위의 index.tsx도 하나의 콤포넌트이며, *메인 페이지*에 해당한다.

### 메인페이지 css

이 페이지의 css를 지정해주자. **styles/Home.module.css**를 불러와

```css
.container {
  display: flex;
}

.left {
  width: 20rem;
}
.right {
}
```

전부 지우고 위 것들만 입력해주자.

### 테스트

npm run dev로 실행해보면  
![wp2-img1](/images/posts/webproject2-img1.png)  
이렇게 뜨면 정상이다!

## 새 콤포넌트 생성

위 사진에서 *left*와 *right*부분에 콤포넌트를 만들어 넣어주려고 한다.  
우선 left 부분의 콤포넌트를 만들어보자.

프로젝트 루트에 있는 **pages** 폴더 내에 *components*라는 폴더를 만들어주자.  
해당 폴더에 들어가 *TossoOperations.tsx*라는 파일을 생성한다.  
![wp2-img2](/images/posts/webproject2-img2.png)  
이렇게 생성되면 정상(Dynamic,tossoDetails는 일단 무시하자)

### 기본 양식 작성

TossoOperations.tsx를

```javascript
import styles from '../../styles/Tosso.module.css';
import ReactQuill from 'react-quill';

const TossoOperations = () => {
  return (
    <>
      <div className={styles.btnContainer}>
        {/*클릭시 inputToggle 트리거 발동-->*/}
        <button className={styles.button}>Add a New Tosso</button>
      </div>

      <div className={styles.inputContainer}>
        <input className={styles.input} placeholder="Enter the tosso.." />
        <div className={styles.ReactQuill}>
          <ReactQuill />
        </div>
      </div>

      <button className={styles.saveBtn}>Save Tosso</button>
    </>
  );
};

export default TossoOperations;
```

Tosso.module.css에

```css
.button {
  width: 15rem;
  height: 2rem;
  cursor: pointer;
  background-color: black;
  color: whiteSmoke;
  border: black;
  font-family: sans-serif;
}

.input {
  width: 15rem;
  height: 2rem;
  outline: none;
  border-radius: 5px;
  border: 1px solid gray;
  margin: 5px 0;
}

.saveBtn {
  width: 15rem;
  height: 2rem;
  cursor: pointer;
  background-color: rgb(119, 27, 27);
  color: whiteSmoke;
  border: rgb(119, 27, 27);
  font-family: sans-serif;
}

.ReactQuill {
  width: 15rem;
}
```

이 항목을 추가해준다.  
그리고 **index.tsx**에서  
`<div className{styles.left}>left</div>` 의 left를  
`<TossoOperations />` 으로 바꿔준다.

이후 실행해보면...  
![wp2-img3](/images/posts/webproject2-img3.png)  
왼쪽 화면이 이렇게 바뀐것을 볼 수 있다!

### 파이어베이스 추가

[이 사이트에 접속해 프로젝트 추가 버튼을 누른다.](https://console.firebase.google.com/u/0/)

프로젝트를 생성한 후 왼쪽의 **Firestore Database**를 클릭한다.  
(생성시 google analystics는 사용하지 않을 것이므로 체크 해제하자)  
데이터베이스를 추가하라고 나올 것인데, 적절한 리전을 선택해 추가해준다.  
(보통은 ap-northeast-x 를 많이 선택한다)  
![wp2-img4](/images/posts/webproject2-img4.png)  
데이터베이스 생성이 완료되면 위 사진과 같이 나올 것이다.  
**컬렉션 시작**을 클릭해 새 콜렉션을 만들어준다.  
![wp2-img5](/images/posts/webproject2-img5.png)  
*첫 번째 문서 추가*는 아무렇게나 대충 만들어주자.

그 다음 이를 자바스크립트에서 쓰기 위해 **앱** 생성을 해야 한다.  
왼쪽 *프로젝트 개요*를 클릭해 프로젝트 홈으로 이동한 뒤  
![wp2-img6](/images/posts/webproject2-img6.png)  
위 그림같은 것에서 **</>** 모양을 클릭한다.

*웹 앱에 Firebase 추가*라는 화면이 나올 텐데,  
앱 닉네임을 입력 후 Firebase 호스팅 체크는 하지 말자.

다음 단계로 넘어가면 *Firebase SDK 추가*라는 화면이 나온다.  
![wp2-img7](/images/posts/webproject2-img7.png)  
*npm 사용*을 체크한 뒤, npm install firebase는 아까 했으니까 패쓰하고  
아래 코드가 중요하다.  
아래 코드를 통째로 복사한 뒤 프로젝트 루트 폴더에 **firebaseConfig.ts** 파일을 생성하여 통째로 붙여넣자.  
![wp2-img8](/images/posts/webproject2-img8.png)  
당직은 firebase 폴더를 생성하여 그 아래에 넣었다.

### 막간을 틈탄 firestore란

firestore는 **NoSQL 데이터베이스**이다.  
NoSQL 데이터베이스는 말 그대로 SQL 명령어를 사용하지 않는 데이터베이스로,  
데이터가 _JSON_ 형태로 저장되어 복잡한 쿼리나 스키마를 만들지 않아도 손쉽게 CRUD[^3] 작업을 할 수 있다.  
그 특성상 비정형화[^4]된 데이터 처리에 강점이 있어 mongoDB, documentDB 같은 빅 데이터 서비스에 많이 사용된다.  
파이어스토어는 이 NoSQL 형식의 DB를 따로 구축할 필요 없이 이름이랑 리전만 입력하면 생성뿐만 아니라 관리/분석까지 알아서 해 준다. 쉽게 말해 AWS 같은거라 보면 된다!

## 콤포넌트에 파이어베이스 연동하기

다시 **TossoOperations.tsx**로 넘어와서...  
맨 위에

```javascript
import { useState, useEffect } from 'react';
import { database } from '../../firebase/firebaseConfig';
import { collection, addDoc, getDocs } from 'firebase/firestore';
```

을 추가한다.  
대충 database는 파이어베이스를 연동하기 위해 필요한 것이고,
collection,addDoc,getDocs는 파이어스토어의 콜렉션을 불러오고, 콜렉션 내 문서 CRUD를 위해 필요하다 보면 된다.

useState, useEffect 는 리액트의 핵심 기능으로, 코드를 보며 설명하겠다.

### 문서 추가하기

우선 문서 추가 기능을 넣어보겠다.

return 구문을 다음과 같이 수정해보자. 설명은 주석문으로 넣어놓았다.

```javascript
return (
  <>
    {/* 입력창 열기 버튼 디브 */}
    <div className={styles.btnContainer}>
      {/*클릭시 inputToggle 트리거 발동-->*/}
      <button className={styles.button} onClick={inputToggle}>
        Add a New Tosso
      </button>
    </div>

    {/* 입력창 디브 */}
    {/* 삼항연산자를 사용하여 isInputVisible 값이 true일 경우에만 입력창 보여주기*/}
    {isInputVisible ? (
      <div className={styles.inputContainer}>
        {/* 입력값 들어올시 tossoTitle 값을 그걸로 변경 */}
        <input className={styles.input} placeholder="Enter the tosso.." onChange={(e) => setTossoTitle(e.target.value)} />
        {/* 입력값 들어올시 tossoDesc 값을 그걸로 변경 */}
        <div className={styles.ReactQuill}>
          <ReactQuill theme="snow" onChange={addDesc} />
        </div>
      </div>
    ) : (
      <></>
    )}

    {/* 저장 버튼 디브 */}
    {/* 클릭시 saveTosso 트리거 발동 */}
    <button className={styles.saveBtn} onClick={saveTosso}>
      Save Tosso
    </button>
  </>
);
```

문제는 이것만 있으면 안 된다. return 위에 다음과 같은 코드를 입력해주자.  
마찬가지로 설명은 주석문으로 대신하겠다.

```javascript
//isInputVisible 기본값은 false
const [isInputVisible, setInputVisible] = useState(false);
//isInputVisible 값 변경시키는 트리거
function inputToggle() {
  setInputVisible(!isInputVisible);
}

// tossoTitle 기본값은 비어있는
const [tossoTitle, setTossoTitle] = useState("");
// firestore에서 해당 콜렉션 가져오기
const dbInstance = collection(database, 파이어스토어 콜렉션 이름);
// addDoc의 트리거함수
function saveTosso() {
  //addDoc은 콜렉션(dbInstance)에 문서 추가
  addDoc(dbInstance, {
    // title 필드 생성후 값을 tossoTitle로, desc 필드 생성후 값을 tossoDesc로
    title: tossoTitle,
    desc: tossoDesc,
    // 문서 추가가 완료되면 콜백작동
  }).then(() => {
    // 입력창 초기화
    setTossoTitle("");
    setTossoDesc("");
    // 새로고침
    window.location.reload();
  });
}

//마찬가지로 기본값이 비어있는 tossoDesc 생성, setTossoDesc로 값 변경
const [tossoDesc, setTossoDesc] = useState("");
//setTossoDesc의 트리거함수(value는 onChange가 알아서 결정)
function addDesc(value) {
  setTossoDesc(value);
}
```

useState는 **리액트 훅**의 일종이다.  
리액트 훅이 무엇이냐면, 사용자 조작에 따른 변동사항을 만드는 것을 뜻한다.
즉 사용자가 화면에 버튼을 클릭하거나, 문자를 입력하면 버튼값이 바뀌거나 문자열값이 바뀌는 등의 동작을 useState로 처리할 수 있는 것이다.

```javascript
const [isInputVisible, setInputVisible] = useState(false);
```

위 useState 중 하나인 isInputVisible을 예로 들자.
isInputVisible은 useState 변수,  
setInputVisible은 isInputVisible 값을 변경하는 함수이다.  
그리고 useState() 내의 false는 isInputVisible의 기본값이다.  
useState 구문의 형식은 이렇게 구성되어야 한다.

```javascript
function inputToggle() {
  setInputVisible(!isInputVisible);
}
```

setInputVisible 함수를 발동시키는 inputToggle을 함수를 보면,  
setInputVisible() 내에 !isInputVisible 인자를 넣어주었다.  
즉 isInputVisible 값을 !isInputVisible 값으로 바꾼다는 것을 뜻한다.  
따라서 setInputVisible()의 인자로는 변경할 isInputVisible 값을 사용하면 된다.

```javascript
{
  isInputVisible ? <div className={styles.inputContainer}>...</div> : <></>;
}
```

위 삼항연산자 항목은 isInputVibiels 값에 따라 <div>...</div> 또는 <></>(비어있는) 값을 표시하는데,

```javascript
<button className={styles.button} onClick={inputToggle}>
  Add a New Tosso
</button>
```

위'Add a New Tosso' 버튼을 클릭시 isInputVisible 값을 바꾸는 inputToggle 함수가 발동되므로  
이 버튼을 누르면 <div>...</div> 항목이 나오거나 나오지 않거나 를 구현하는 것이다.

이 <div> 내 항목을 보면

```javascript
<input className={styles.input} placeholder="Enter the tosso.." onChange={(e) => setTossoTitle(e.target.value)} />
```

여기서 onChange에 주목해보자.  
즉 input 값이 바뀔때마다 해당 값(e.target.value)으로 tossoTitle을 변경한다는 것이다.

이런 식으로 useState() 기능을 사용하는 것이다.  
참고로 useState()는 **리액트 16.8**부터 지원한다는 것을 기억하자.

### 테스트

![wp2-img9](/images/posts/webproject2-img9.gif)  
Add a New Tosso 버튼을 누르면 창이 나온다.  
제목과 내용을 입력하고 Save Tosso를 누르면 창이 새로고침될 뿐 아무것도 일어나지 않는다.  
하지만 아까 생성한 *파이어스토어 콜렉션*에 들어가보면...?  
새로운 문서가 생긴 것을 볼 수 있다!

## 저장된 문서 표시

그 다음 파이어스토어 내 문서들을 표기하고 수정/삭제도 가능하게 하는 콤포넌트 **TossoDetails.tsx**를 ```components``` 폴더에 작성한다.

```javascript
import { useState, useEffect } from 'react';
import { app, database } from '../../firebase/firebaseConfig';
import {
  doc,
  getDoc,
  getDocs,
  collection,
  updateDoc,
  deleteDoc,
} from 'firebase/firestore';
import styles from '../../styles/Evernote.module.scss';
import QuillWrapper from './Dynamic';
import 'react-quill/dist/quill.snow.css';

const TossoDetails = ({ id }) => {
  const [singleTosso, setSingleTosso] = useState({});
  //프롭스로 받은 id에 해당하는 firestore 문서 정보를 singleTosso에 저장
  //firebase에 요청하므로 비동기 처리사용
  async function getSingleTosso() {
    if (id) {
      const singleTosso = doc(database, 'tosso', id);
      //getDoc은 특정 문서(doc객체)만 받아오는 것
      const data = await getDoc(singleTosso);
      setSingleTosso({ ...data.data(), id: data.id });
    }
  }
  //id가 바뀔 때마다 getSingleTosso 다시 실행
  //[]이면 페이지가 새로 렌더링(새로고침등)될때마다, [값]이면 값이 바뀔 때마다, 없으면 처음 한번만 수행
  useEffect(() => {
    getSingleTosso();
  }, [id]);

  // 새로고침시 기본 표시할 문서(맨 처음 문서를 기본으로)
  function getTosso() {
    const dbInstance = collection(database, 'tosso');
    getDocs(dbInstance).then((data) => {
      setSingleTosso(
        data.docs.map((item) => {
          return { ...item.data(), id: item.id };
        })[0],
      );
    });
  }
  useEffect(() => {
    getTosso();
  }, []);

  // 수정버튼 눌렀는지 관련 스테이트
  const [isEdit, setIsEdit] = useState(false);

  // 수정버튼 누르면 수정창이 나오도록
  const getEditData = () => {
    setIsEdit(true);
    setTossoTitle(singleTosso.title);
    setTossoDesc(singleTosso.desc);
  };

  // 수정할 제목과 내용 관련 스테이트
  const [tossoTitle, setTossoTitle] = useState('');
  const [tossoDesc, setTossoDesc] = useState('');

  const editTosso = (id) => {
    const collectionById = doc(database, 'tosso', id);

    updateDoc(collectionById, {
      title: tossoTitle,
      desc: tossoDesc,
    }).then(() => {
      window.location.reload();
    });
  };

  const deleteTosso = (id) => {
    const collectionById = doc(database, 'tosso', id);

    deleteDoc(collectionById).then(() => {
      window.location.reload();
    });
  };

  return (
    <>
      <h2>{singleTosso.title}</h2>
      <div dangerouslySetInnerHTML={{ __html: singleTosso.desc }}></div>

      <div>
        <button className={styles.editBtn} onClick={getEditData}>
          Edit
        </button>
        <button
          className={styles.deleteBtn}
          onClick={() => deleteTosso(singleTosso.id)}>
          Delete
        </button>
      </div>

      {isEdit ? (
        <div className={styles.inputContainer}>
          <input
            className={styles.input}
            placeholder="Enter the Title.."
            onChange={(e) => setTossoTitle(e.target.value)}
            value={tossoTitle}
          />
          <div className={styles.ReactQuill}>
            <QuillWrapper
              theme="snow"
              onChange={setTossoDesc}
              value={tossoDesc}
            />
          </div>
          <button
            className={styles.saveBtn}
            onClick={() => editTosso(singleTosso.id)}>
            Update Note
          </button>
        </div>
      ) : (
        <></>
      )}
    </>
  );
};

export default TossoDetails;
```

그 다음 ```index.tsx```에 이 콤포넌트를 표시해줘야겠지?
```<div className{styles.right}>right</div>``` 를  
```<TossoDetails />``` 로 바꿔주면 된다. 물론 당연히 TossoDetails import는 필수이고.

## 다음 시간에 계속...

다음 시간에는 게시물 표시, 수정과 삭제 기능을 구현해보겠다.

---

[^1]: 개인이나 회사 등이 개발한 기능을 다른 사람들이 편리하게 사용할 수 있도록 제공하는 기능이다. 예로 웹사이트에 지도를 표시하고 싶은데, 지도를 직접 만드는 것보다 구글에서 제공하는 지도 API를 사용하면 훨~씬 빠르고 간단하게 구현할 수 있다.
[^2]: 정확히는 jsx라는, 자바스크립트에서 지원하는 html 비스무리한 문법이다. 리액트에서는 콤포넌트에 return(<jsx 문법>) 을 사용하여 html을 구현한다.
[^3]: 생성(**C**reate), 조회(**R**ead), 수정(**U**pdate), 삭제(**D**elete)로, 데이터 처리의 기본 기능을 말한다.
[^4]: 쉽게 말해 DB 내 데이터들의 형식이 일정하지 않는다는 것이다.
