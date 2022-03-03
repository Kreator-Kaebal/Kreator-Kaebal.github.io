---
layout: post
title: 타입스크립트를 사용한 리액트 웹실습1
tags : [java/typescript,react]
excerpt : 타입스크립트 클론 코딩-사전준비
---

# 타입스크립트를 사용한 리액트 실습준비

앞전에 말했듯이 이제 타입스크립트를 사용하여 시중 웹 페이지를 모방해보는 리액트 프로젝트를 진행하겠다.  

참고로 리액트는 이전 포스트에서 말했듯이 웹 페이지 개발을 쉽게 해주는 도구라고 보면 된다.

## 리액트 프로젝트 생성

[여기를 한번 참조해보자](https://ko.reactjs.org/docs/getting-started.html)  

1. [노드 JS를 다운로드한다.](https://nodejs.org/ko/)
   * 컴퓨터에 노드 JS가 설치되어 이제 npm 및 npx 사용이 가능해졌다.
   * npm 및 npx는 파이썬의 pip와 같은 패키지 설치 관리자이다.
2. 콘솔창을 열고 ```npx create-react-app 프로젝트 이름 --template typescript```으로 리액트 프로젝트를 생성한다.
   * 뭔가 깔라는 페이지가 나오면 전부 y 누르고 엔터 친다.
   * ```--template typescript```를 붙여줬기 때문에 이 리액트 프로젝트는 타입스크립트 사용을 전제로 설정된다.
3. 리액트 프로젝트 폴더가 생성되었다. 해당 폴더 경로로 콘솔창을 켠 후 ```npm start```를 한다.
   * 콘솔창에 뭔가 좌라락 뜨고 주소가 ```localhost:3000```인 웹페이지가 생성된다.
   * ```npm start```는 본인 컴퓨터를 노드 JS 서버로 하여 리액트 프로젝트를 실행한 것이다.
4. ```localhost:3000``` 페이지는 리액트에서 기본으로 제공해주는 메인 페이지이다. 우리는 앞으로 이것을 수정하는 식으로 개발을 진행할 것이다.
   * 본인 컴퓨터를 서버로 사용하였으므로 주소가 로컬호스트로 표기된다.  
   ![tsc2-img1](/images/posts/typescript2-img1.png)  
   참고로 컴퓨터와 같은 네트워크 내 기기들은 콘솔창에 뜨는 ip 주소로도 해당 페이지에 접속할 수 있다.
   * 리액트 프로젝트의 기본 포트는 3000이다.

## 프로젝트 설정

[여기를 한번 참조해보자](https://create-react-app.dev/docs/adding-typescript/)
![tsc2-img2](/images/posts/typescript2-img2.png)  

* 프로젝트 폴더에 들어가면 아래와 같은 디렉토리가 나온다.
  * .git 및 readme.md는 본인이 이 프로젝트를 깃허브 레포지토리로 설정해놓아 생긴 것이므로, 없어도 무방하다.

### 개발 보조모듈 설치

[여기를 참고했습니다](https://velog.io/@junghyeonsu/React-create-react-app-Typescript-%EC%B4%88%EA%B8%B0-%EC%84%B8%ED%8C%85-%EC%99%84%EB%B2%BD-%EC%A0%95%EB%A6%AC)

난잡한 코드 형식을 통일화해주는 **Prettier**을 설치해보자.  

* 일단 VSCode에서 Prettier 확장 프로그램을 설치한다.
* 그 다음 프로젝트 디렉토리에서 콘솔창을 열고,  
  ```npm i prettier -D -E```  
  으로 프로젝트에 prettier 모듈을 설치한다.
  * -D 옵션은 **devDependencies 전용**을 뜻한다.
    * ```package.json``` 파일에는 ```dependencies```와 ```devDependencies```라는 JSON 키가 있는데, 이들은 파이썬의 requirements.txt 같이 이 프로젝트를 실행하기 위해 필요한 모듈 정보들을 담고 있다.
    * dependencies는 **실제 배포에 필요한**, devDependencies는 **개발 단계에만 필요한** 모듈을 놓는다.
    * 보통 모듈을 설치하면 dependencies에 들어가는데, -D 옵션을 주면 대신 devDependencies에 들어간다.
  * -E 옵션은 **버전 고정**을 뜻한다. 이렇게 하면 dependencies에 이 버전 이하도 이상도 아닌 정확한 버전으로 설치되도록 설정된다.
    * 참고로 dependencies를 보면 보통은 ^(버전명) 이렇게 되어 있는데, 이는 *이 버전 이상으로 설치*하라는 의미이다.
* ```create-react-app```으로 생성한 리액트 프로젝트에는 **ESLint**[^1]라는 모듈이 기본적으로 깔려 있다.
  * ESLint는 플러그인과 설정이라는 것으로 구성되어 있으며, 이 둘을 적절히 선택할 수 있다.
  * Prettier와 ESLint를 그대로 같이 사용하면 모듈 충돌이 일어나므로, 충돌을 막아주는 ESLint 플러그인과 설정인  
  ```npm i eslint-plugin-prettier eslint-config-prettier -D```  
  를 설치해준다.
  * 또한  
    ```npm i eslint -D```  
    로 eslint를 devDependencies로 보내야 한다.
  * package.json을 열어서 devDependencies가 위와 같으면 잘 따라왔다.  
    ![tsc2-img5](/images/posts/typescript2-img5.png)
* 이제 ESLint 설정을 해줘야 한다. 프로젝트 폴더에서  
  ```npx eslint --init```  
  을 실행시키고, 아래와 같이 선택한다.  
  
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

* 설정이 끝나면 폴더에 ```.eslintrc.json```이 생길 것이다. 이 파일이 ESLint 설정을 담고 있는 곳인데, 아래와 같이 바꿔준다.

    ```
    {
        "env": {
            "browser": true,
            "es2021": true,
            "node": true
        },
        "extends": [
            "plugin:react/recommended",
            "airbnb",
            "plugin:@typescript-eslint/recommended",
            "plugin:prettier/recommended"
        ],
        "parser": "@typescript-eslint/parser",
        "parserOptions": {
            "ecmaFeatures": {
                "jsx": true
            },
            "ecmaVersion": "latest",
            "sourceType": "module"
        },
        "plugins": [
            "react",
            "@typescript-eslint",
            "prettier"
        ],
        "rules": {
        }
    }

    ```

    rules는 세부 규칙인데, 일단은 비워두자. 보통 기본 규칙을 무시하고 싶을 때 사용한다.
* 이제 prettier 설정을 해줘야 한다.  
  프로젝트 폴더에 ```.prettierrc``` 파일을 생성한다. 확장자 없이!
  * 파일을 메모장으로 열어 다음과 같이 입력한다.
    
    ```
    {
        "endOfLine": "auto" //줄바꿈 처리방식
        "singleQuote": true, //작은따옴표 사용여부
        "semi": true, //세미콜론 사용여부
        "useTabs": false, //탭 사용여부
        "tabWidth": 2, //탭 길이
        "trailingComma": "all", //하나의 코드가 여러 줄을 먹을 때 줄바꿈 콤마 사용방식
        "printWidth": 160, //졸바꿈폭
        "arrowParens": "always", //화살표함수 괄호사용방식
        "bracketSpacing": true, //괄호에 공백
    }
    ```

  * 이 파일은 prettier 적용 규칙을 나타낸다. 자세한 설명은 [이곳에 친절하게 나와있다.](https://velog.io/@planethoon/.prettierrc-%EA%B8%B0%EB%B3%B8-%EC%84%A4%EC%A0%95)

* 그 다음 VSCode를 열어 ```Ctrl+,```으로 설정창에 들어간 후 ```Format On Save```를 검색한다.
  * 체크박스가 하나 보일텐데, 체크해준다.

* 이제 타입스크립트 파일을 작성할 때 문법 에라가 표시되고, 파일을 저장하면 위 설정대로 코드 양식이 통일화된다!
### 컴파일 환경설정

* tsconfig.json을 이렇게 수정해준다.
  ```
  {
    "compilerOptions": {
        "target": "es5",
        "lib": [
            "dom",
            "dom.iterable",
            "esnext"
        ],
        "allowJs": true,
        "skipLibCheck": true,
        "esModuleInterop": true,
        "allowSyntheticDefaultImports": true,
        "strict": true,
        "forceConsistentCasingInFileNames": true,
        "noFallthroughCasesInSwitch": true,
        "module": "esnext",
        "moduleResolution": "node",
        "resolveJsonModule": true,
        "isolatedModules": true,
        "noEmit": true,
        "jsx": "react-jsx"
    },
    "include": [
        "src/**/*"
    ]

  }
  ```

  * 해당 파일은 타입스크립트 컴파일 설정이다.  
  ```include:[]```는 **컴파일할 파일**을 설정한다. 여기서는 src 내의 모든 파일을 컴파일하도록 했다.
  * 마찬가지로 include에서 컴파일 제외할 파일을 설정해놓는 ```exclude:[]```도 있다. 따로 명시해놓지 않으면 ```exclude:["node_modules", "bower_components", "jspm_packages"]```으로 기본 설정된다.

## NextJS 설치

[여기를 한번 참조해보자](https://nextjs.org/docs/api-reference/create-next-app)

만약 본인이 Node.js 대신 **Next.js**를 사용하고 싶다면...  

* 프로젝트를 새로 생성한다.
  * 이 때 ```npx create-next-app 프로젝트명 --typescript``` 이렇게 생성하자.
* 프로젝트가 생성되었으면 디렉토리로 이동해 ```npm run dev```으로 실행한다.
  * ![tsc2-img3](/images/posts/typescript2-img3.png)  
    실행 후 프로젝트 디렉토리는 이렇게 생겼다.  
    .git 및 readme.md는 지워도 무방하다.
* 만약 본인 깃 레포지토리로 바꾸고 싶으면...(깃 레포지토리 생성 가정하에)
    1. .git과 readme.md를 본인 레포지토리의 것으로 덮어씌운다.
    2. ```git add .```으로 변경사항을 전부 추가한다.
    3. ```git commit```과 ```git push```로 레포지토리 반영한다.
* create-next-app 시 package.json의 dependencies와 devDependencies는 자동으로 설정되었다.
  * 즉 아까처럼 npm을 사용해 일일이 등록을 안해줘도 된다.
  * 따라서 eslint 설정은 그냥 ```npx eslint --init```해주고 적절히 셋팅해주자.
  * 다만 prettier는 수동으로 설치하고 설정해 줘야 한다. eslint-plugin-prettier 및 eslint-config-prettier도 마찬가지이다.
* 마찬가지로 tsconfig.json도 create-next-app 시 include/exclude 등이 자동 설정되었다.

앞으로 프로젝트는 next.js 환경을 전제로 개발해보겠다.
![tsc2-img4](/images/posts/typescript2-img4.png)  
다음 시간에 계속...

[^1]: 자바스크립트 코드 문법을 체크해주는 확장 기능이다.