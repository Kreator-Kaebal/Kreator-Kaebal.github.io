---
layout: post
title: 타입스크립트를 사용한 웹 프로젝트 번외편
categories: [웹개발-타입스크립트 프로젝트]
tags: [java/typescript, firebase, analytics]
excerpt: 번외편-구글 애널래틱스
---

## 구글 애널래틱스로 사용자 분석

웹페이지에 구글 애널래틱스를 적용하면 페이지에 접속한 사용자들이 누가/얼마나 있는지 등을 볼 수 있다.  
물론 접속한 사람의 신상정보가 뜨는 것은 아니고, 어느 나라 어느 지역에서 들어왔는지 같은 기초적인 정보만 알려준다. 애초에 접속기록만으로 신상정보까지 알아내는 것은 해킹이다!

![wp11-img1](/images/posts/webproject11-img1.png)

구글 애널래틱스는 이러한 접속 통계를 자동으로 분석해주기 때문에 마케팅이나 UX 측면에서 매우 유용하다고 볼 수 있다.

구글 애널래틱스를 적용해보자. 다만 이번에 진행한 웹 프로젝트는 웹서버 배포가 되어있지 않아 실제로 적용할수는 없기 때문에(도메인이 있어야 한다) 하는 방법만 보여주겠다.

## 애널래틱스 생성

[공식 문서](https://support.google.com/analytics/answer/9306384?hl=ko)에 보면 잘 나와있긴 하지만 Next.js에 넣는것은 조금 다르다.

![wp11-img2](/images/posts/webproject11-img2.png)

일단 문서에서처럼 본 서비스를 사용할 (구글)계정으로 [여기](https://www.google.com/analytics/)에 접속하여 무료로 시작하기를 누른다.  

![wp11-img3](/images/posts/webproject11-img3.png)

이렇게 애널래틱스 계정 생성부터 해야하는데 아래 계정 데이터 공유 설정은 알아서 정하면 된다.

![wp11-img4](/images/posts/webproject11-img4.png)

속성은 일종의 분석 단위인데, 속성 하나당 하나의 앱(웹사이트 등)을 분석할 수 있다. 즉 하나의 계정에는 여러 속성이 있을 수 있다는 말이다. 때문에 이름은 구분하기 쉽도록 앱 이름(여기서는 pseudo-website)과 똑같이 맞춰주자. 보고 시간대 및 통화는 우리나라 시간으로 바꾸는 것도 잊지 말고.

![wp11-img5](/images/posts/webproject11-img5.png)

비지니스 정보. 이것도 알아서 해주자. 그리고 만들기 버턴을 클릭하면

![wp11-img6](/images/posts/webproject11-img6.png)

이렇게 애널래틱스 계정과 속성 하나가 만들어진다.  
여기서 잠깐!

### 계정, 속성, 보기

구글 애널래틱스는 **계정>속성>보기** 순의 세 가지 계층으로 구성된다.

* **계정**은 애널래틱스를 사용하고 속성을 거느리기 위한 말 그대로의 계정이다.
* **속성**은 웹사이트, 모바일 앱 또는 기기이다.
* **보기**는 속성에서 분석할 데이터를 정하는 것이다.

위에서 써논 말로는 이해가 잘 되지 않을텐데, 예시를 들자면 당직이 pseudo-website 사이트 외에도 tosso라는 사이트도 운영한다고 한다.  
만일 둘 다 애널래틱스를 사용하고 싶으면 당직 계정을 생성 후 그 계정에 두 개의 속성을 추가하고, 각 사이트마다 속성 하나를 붙여주면 되는 것이다.  
또한 tosso는 모바일 화면 접속만 분석하고 싶으면 해당 속성의 보기를 그렇게 설정해주면 된다.

속성과 보기는 이 외에도 많은 기능을 제공하고 있으므로 직접 찾아보며 익히면 된다.

### Next.js에 애널래틱스 넣기

애널래틱스를 갓 만들면 속성의 **데이터 스트림** 항목이 클릭되어 있다.  
이 데이타 스트림은 프로젝트에 애널래틱스를 연동시켜주는 기능이다.
따라서 생성해야 할 텐데, pseudo-website는 웹 앱이므로 **웹**을 클릭한다.

![wp11-img7](/images/posts/webproject11-img7.png)

생성하려면 웹사이트 URL이 필요한데, 안타깝게도 이 웹은 도메인으로 배포하지 않았기 때문에 URL이 없다...(참고로 로컬호스트는 안된다)  
다만 당직은 ngrok을 사용해 로컬호스트를 우회하는 식으로 임시 URL을 만들었다. 어차피 테스트용으로만 쓸 것이기 때문에 이렇게라도 대충 만들고 스트림 만들기를 클릭하자.

ngrok이 뭔지는 [여기에서](https://kreator-kaebal.github.io/webproject6)

![wp11-img8](/images/posts/webproject11-img8.png)

이런 식으로 어떻게든 스트림을 생성하면 이렇게 세부정보가 뜨고, *태그하기에 대한 안내* 항목->*새로운 온페이지 태그 추가* 메뉴->*전체 사이트 태그(gtag.js)...* 을 클릭하면 사진처럼 뭐라 하는 코드가 잔뜩 뜬다.

이걸 일단 복사해두자. 이제 프로젝트 코드를 수정해야 할 차례다.  
또한 위 사진의 **측정 ID**값도 기억해두자.

애널래틱스 처리(방문객, 각종 이벤트 등)를 위해 src 폴더 등에 모듈(여기서는 gtag.ts로 하였다)을 하나 만든 다음

```javascript
export const GA_TRACKING_ID = '(측정 ID값)';

export const pageview = (url:URL) => {
  window.gtag('config', GA_TRACKING_ID, {
    page_path: url,
  });
};

type GTagEvent = {
    action: string;
    category: string;
    label: string;
    value: number;
  };

export const event = ({ action, category, label, value }: GTagEvent) => {
  window.gtag('event', action, {
    event_category: category,
    event_label: label,
    value,
  });
};
```

이렇게 입력해주자.  
만약 ```gtag```가 빨간줄이 뜬다면  

```
npm install -D @types/gtag.js
```

을 해보자.  
그 다음 _document.jsx/tsx(없으면 만들라)에 들어가서

```javascript
import Document, { Html, Head, Main, NextScript } from 'next/document';
import React from 'react';

class MyDocument extends Document {
    ...

    render() {
        return (
            <Html>
                <Head />
                <body>
                    <Main />
                    <NextScript />
                </body>
            </Html>
        );
    }
}

export default MyDocument;
```

_document.jsx/tsx의 구조는 원래 이런 모양이다. 여기의 ```<Head>``` 태그 아래에 아까 복사한 코드를 붙여넣어야 한다.  

```javascript
import { GA_TRACKING_ID } from '(gtag.js/ts의 위치)'
...
<Head>
    <script
        async
        src={`https://www.googletagmanager.com/gtag/js?id=${GA_TRACKING_ID}`}
    />
    <script
       dangerouslySetInnerHTML={% raw %}{{
          __html: `
          window.dataLayer = window.dataLayer || [];
          function gtag(){dataLayer.push(arguments);}
          gtag('js', new Date());
          gtag('config', GA_TRACKING_ID, {
            page_path: window.location.pathname,
          });
          `,
       }}{% endraw %}
    />
</Head>
...
```

(raw/endraw 는 jekyll 컴파일 오류때문에 집어넣은 문자로 실제 적용에서는 삭제해주자)

다만 그대로 복사했다가는 React 계열의 특성상 jsx 언어에서는 주석문을 사용하지 못하고 ```<script>``` 태그로 자바스크립트를 바로 넣지 못하기 때문에(보안 때문이라고 한다)
*dangerouslySetInnerHTML* 등을 사용하여 위와 같이 jsx 형식에 맞게 수정해줘야 한다.

그리고 마지막으로, _app.jsx/tsx 에 들어가서 페이지에서 gtag.ts 처리를 위한 useEffect 문 하나를 추가해준다.

```javascript
...
import { useRouter } from 'next/router';
import * as gtag from '../stores/gtag';
...

const router = useRouter();
...
  useEffect(() => {
    const handleRouteChange = (url: URL) => {
      gtag.pageview(url);
    };
    router.events.on('routeChangeComplete', handleRouteChange);
    return () => {
      router.events.off('routeChangeComplete', handleRouteChange);
    };
  }, [router.events]);
...
```

### 실행해보기

![wp11-img9](/images/posts/webproject11-img9.png)

아까 애널래틱스 콘솔에 접속하여 집 모양 버턴을 누르면 통계를 볼 수 있다.

이제 페이지에 접속하면 당직이 접속한 기록이 나온다!
