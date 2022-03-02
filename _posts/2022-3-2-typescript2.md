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
* 본인이 VSCode를 사용중이고 ESLint[^1]를 사용하고 싶으면  
  ```npm install -D eslint```
  을 사용해 eslint를 ```package.json```의 ```devDependencies:{}``` 목록에 추가한다.
* ```package.json```은 일종의 프로젝트 환경 설정인데, ```dependencies:{}```는 이 프로젝트를 **실행하기 위해 필요**한 모듈들, ```devDependencies:{}```는 **개발할 때만 사용**하는 npm 모듈을 모아놓는 곳이다.
  * ```npm install```로 모듈을 설치하면 기본적으로 ```dependencies:{}```에 추가된다. 만약 배포에는 필요 없고 개발할때만 사용할 모듈이 필요하면 ```npm install -D```로 해주면 된다.
  * 그 다음 ```npx eslint --init```으로 프로젝트 eslint의 설정을 한다.  
  이 때  
  ```√ Does your project use TypeScript? · No / Yes```,  
  ```√ Would you like to install them now with npm?```  
  에는 반드시 **Yes**를 체크해줘야 한다.  
  프로젝트가 타입스크립트 사용하냐는 질문과, 이전 질문에서 설정한 eslint 포맷을 npm -D 설치할지 물어보는 질문이다.  
  * 이렇게 하면 프로젝트 폴더에 ```eslintrc.json```이 생성된다.  
  ```√ What format do you want your config file to be in?```질문에 어떻게 답변했는가에 따라 확장자가 달라질 수 있다.  
  어쨌든 이 파일은 eslint 설정을 담는 파일이다.
* tsconfig.json을 이렇게 수정해준다.
  ```
  {
    "compilerOptions": {
        "target": "es6",
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
  * ```"target": "es6"```은 **어떤 자바스크립트 문법으로 컴파일**할 것인가를 설정한다. 기본 es5를 최신 문법인 es6으로 바꾸었다.

## NextJS 설치

[여기를 한번 참조해보자](https://nextjs.org/docs/api-reference/create-next-app)

만약 본인이 Node.js 대신 **Next.js**를 사용하고 싶다면...  

* 프로젝트를 새로 생성한다.
  * 이 때 ```npx create-next-app pseudo-webside --typescript``` 이렇게 생성하자.
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
* 마찬가지로 tsconfig.json도 create-next-app 시 자동 설정되었다.
  * ```"target": "es5"``` 항목만 es6으로 바꿔주자.

앞으로 프로젝트는 next.js 환경을 전제로 개발해보겠다.
![tsc2-img4](/images/posts/typescript2-img4.png)  
다음 시간에 계속...

[^1]: 자바스크립트 코드 문법을 체크해주는 확장 기능이다.