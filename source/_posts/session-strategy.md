---
layout: title
title: 웹 서버 개발의 Session 전략 
date: 2019-04-20 23:32:01
description: 웹 서버 개발에 Session 의 문제점과 서비스 환경에 따른 전략에 대해 설명합니다.
tags:
 - session
 - cookie
categories:
 - Server
 - Web
thumbnailImage: thumbnail.png
thumbnailImagePosition: left
autoThumbnailImage: yes
coverSize: full
coverMeta: in
coverImage: cover.jpg
---

이번 포스팅은 웹 서버 개발에 Session 의 문제점과 서비스 환경에 따른 전략 대해 설명합니다.

<!-- excerpt -->

들어가기 앞서 코드의 설명은 nodejs + express 로 하겠습니다. 
전체 소스 코드는 [gitHub](https://github.com/devhaks/myNode) 의 session 브랜치를 참고하시면 됩니다.

## 목차

* [배경](#배경)

* [Session 이란](#Session-이란)

* [Session의 저장 방식](#Session의-저장-방식)
 * [In memory](#In-memory)
 * [File storage](#File-storage)
 * [Database storage](#Database-storage)

* [마치며](#마치며)

## 배경

이 글을 쓰게된 이유는 스스로의 궁금증을 풀고자 합니다. 저는 CMS 블로그 [Wordpress](https://ko.wordpress.org/) 개발을 3년 정도 했었고 현재는 [Parse](https://parseplatform.org/) 로 웹 서버 개발을 하고 있습니다. 

위 두 서비스의 DB 구조를 보면서 공통적인 부분을 발견했습니다.

바로 Session 이 DB 에 저장되어 있다는 것입니다. 저의 의문은 여기서 시작 되었고 왜 Session 을 DB 에 저장하는지 이유를 파헤쳐보려 합니다.

{% alert info %}
Wordpress 는 블로그로 알려진 세계적인 CMS(컨텐츠 관리 시스템) 입니다. 

Parse 는 백앤드 개발을 빠르고 쉽게 지원 하기 위해 각종 플랫폼(웹, 안드로이드, IOS) 에 맞는 api, sdk 를 제공합니다. 초기에 페이스북에서 BaaS(Backend as a Service)로 런칭했었고 현재는 오픈소스 입니다.
{% endalert %}

---

## Session 이란

Session 을 이해하기 위해서는 먼저 '왜 Session 이란 개념이 생긴 것일까?' 부터 이해 하셔야 합니다.
모든 웹 서비스는 http 통신을 기본으로 하고 있습니다. http 통신은 서버와 클라이언트 간의 소통을 하기 위한 길 입니다. 

그러면 이 길을 누가 유지 하나요? 클라이언트 or 서버 일까요? 정답은 둘 다 유지 하지 않습니다. 

이러한 상황 때문에 http 는 `stateless` 특성을 갖습니다. 즉, 누구도 상태를 유지하지 않는다는 것이죠. 상태를 유지 하지 않으면 어떠한 상황이 발생 될까요? 예를 들면, 사용자가 온라인 구매 사이트에 접속하여 물품을 장바구니에 넣어놓고 다음날 다시 접속해서 확인하니 장바구니에 물품이 없는 상황이 발생합니다.

위와 같은 상황이 발생하기 때문에 상태 유지를 위한 `statefull` 을 지원하기 위해 Session 개념이 생긴 것입니다.

Session 만 얘기하면 서운한 녀석이 바로 Cookie 입니다. 쉽게 설명하면 클라이언트가 서버에 요청 했을 때, 요청한 사용자에게 증거를 남기기 위한 증거물을 Cookie 라고 이해 하시면 됩니다. 

## Session의 저장 방식

Session의 저장 방식은 3가지로 나눌 수 있습니다. 아래의 순서는 의미가 있습니다. 각 저장 방식에 따른 문제점을 보완한 단계를 의미합니다. 어떤 문제점이 있는지 확인하는 과정을 통해 설명 하겠습니다.

1. [In memory](#In-memory)
 - app 을 구동하고 있는 서버의 메모리에 Session을 생성합니다.

2. [File storage](#File-storage)
 - app 을 구동하고 있는 서버의 특정 디렉토리에 파일 형태로 Session을 생성합니다.

3. [Database storage](#Database-storage)
 - file storage 대신에 DB 에 저장합니다. 

---

#### In memory

app.js
```js
const session = require("express-session");

app.use(
  session({
    secret: "yourPrivateKey", // Session을 hash 하기 위한 비밀키 입니다.
    resave: false,
    saveUninitialized: true,
    cookie: {
      secure: false,
      maxAge: 600 * 1000 // Cookie의 생명 주기를 설정합니다.
    }
  })
);
```

routes/index.js
```js
router.use("/session", (req, res) => {
  const { sessionID, cookies } = req;

  // 서버가 생성한 sessionID
  console.log('sessionID:', sessionID);
  // 클라이언트가 보유한 cookie
  console.log('cookies.yourCookieName', cookies.yourCookieName);

  res.json({
    sessionID,
    cookies
  });
});
```

http://localhost:3030/session 로 접속해서 메모리에 Session을 저장 하겠습니다.
 
첫번째 클라이언트가 요청 했을 때, Cookie는 없고 서버는 Cookie가 없음을 인지하고 메모리에 Session을 저장하고 클라이언트로 전달합니다. 응답 받은 클라이언트는 Cookie를 저장합니다. 
```json
 // 첫번째 접속한 경우
{
  sessionID: "TcZ8P0XZMk4gBb3mhWH4-NHXnOGBJSFq",
  cookies: { }
}
```

두번째 클라이언트가 요청 했을 때, 클라이언트는 서버로 Cookie를 전달하고 서버는 Cookie를 분석해서 메모리에 Session이 존재하는지 확인 합니다. Session이 존재하므로 Session을 다시 생성하지 않습니다. Session은 서버에 저장하고 Cookie는 클라이언트에 저장하여 `statefull` 상태가 됩니다. 이후 임의의 사용자가 웹 서비스를 이용할 때, 무엇을 했는지에 대한 추적이 가능해 집니다. (주의. 로그인과 동일시 생각하면 안됩니다.)
```json
// 두번째 접속한 경우
{
  sessionID: "TcZ8P0XZMk4gBb3mhWH4-NHXnOGBJSFq",
  cookies: {
    yourCookieName: "s:TcZ8P0XZMk4gBb3mhWH4-NHXnOGBJSFq.5Pykjnn5ekwwJUTDYnuxnXeO4N2Kdwbiske1xZegu+g"
  }
}
```

##### 문제점 파헤치기

1. <u>자, 그러면 app 이 오류가 발생해서 죽었다고 가정합시다.</u> app 을 재시작을 재접속하면 어떻게 될까요? Cookie는 그대로이지만 서버의 Session이 변경되었습니다. 왜 바뀌었는지 아시겠나요? app 을 재시작하면서 메모리에 저장된 Session은 모두 초기화 되었습니다. 그래서 Cookie는 동일 하지만 메모리에 Session이 없기 때문에 Session을 재생성 한 것입니다. Session이 변경되면 임의의 사용자가 무엇을 했는지에 대한 정보가 손실됩니다.

  ```json
  // 세번째 접속한 경우
  {
    sessionID: "79SN6VIQvVh_b3Tfi0Ybw41zoboYNmMp", // 변경된 Session
    cookies: {
      yourCookieName: "s:TcZ8P0XZMk4gBb3mhWH4-NHXnOGBJSFq.5Pykjnn5ekwwJUTDYnuxnXeO4N2Kdwbiske1xZegu+g"
    }
  }
  ```

2. app 이 죽지 않고 잘 돌아가면 문제가 없을까요? 또 다른 이유로는 `Memory` 용량입니다. 메모리는 항상 제한적입니다. 사용자가 급격히 증가하면, Session이 차지하는 메모리는 증가하고 용량 app 은 시스템 메모리 초과로 인해 죽게 될 것입니다.

---

#### File storage

In Memory 방식의 문제점을 보완하여 파일형태로 Session을 저장하도록 하겠습니다.

app.js

```js
const session = require("express-session");
// Session을 파일로 생성해주는 모듈
const FileStore = require("session-file-store")(session);

app.use(
  session({
    name: "yourCookieName",
    secret: "yourPrivateKey",
    resave: false,
    saveUninitialized: true,
    store: new FileStore() // 모듈 적용
  })
);
```

http://localhost:3030/session 로 접속해서 Session 디렉토리 Session을 file 로 생성합니다.

mYCHWOju_gFEiGM_tvxBvi2FtsVdepju.json (sessionId 값이 파일명이 됨.)
```json
{
  cookie: { 
    originalMaxAge: null, 
    expires: null, 
    httpOnly: true, 
    path: "/" 
  },
  __lastAccess: 1555819224539
};
```

서버를 강제로 재시작 해도 Session file 은 그대로 유지되며, 임의 사용자에 대한 정보 손실을 막을 수 있게 되었습니다. 클라이언트의 Cookie도 그대로 입니다.
```json
{
  sessionID: "mYCHWOju_gFEiGM_tvxBvi2FtsVdepju",
  cookies: {
    yourCookieName: "s:mYCHWOju_gFEiGM_tvxBvi2FtsVdepju.tFSC7IGe+j0Dp9fHYtVFUZBD3E2NpvLWCW0u2PY+0NY"
  }
}
```

##### 문제점 파헤치기

이제 더 이상 문제가 없어 보이는데, 또 다른 문제가 있을까요? 문제가 될만한 상황을 만들겠습니다.

{% image fancybox center clear group:travel example1.png "로드 밸런서에 의한 트래픽 분산" %}



사용자의 급증으로 서버를 2대로 늘렸습니다. 기존 서버는 server_1, 새로운 서버는 server_2 입니다. 사용자는 로드 밸런서에 의해 분할되어 접속하게 됩니다.

서버를 늘리기 전에 사용자는 온라인 사이트에 물품을 장바구니에 넣었습니다. 서버의 Session 파일에는 사용자의 장바구니 정보가 저장되어 있습니다. 

그런데 서버를 늘린 후에 사용자는 자신의 장바구니의 물품이 사라졌다고 합니다. 왜 이런 문제가 발생한 걸까요? 원인은 새로운 서버에 사용자의 Session 파일이 없기 때문입니다. 이와 같은 상황은 다음 그림과 동일하다 볼 수 있습니다.

{% image fancybox center clear group:travel example3.png "동일 사용자가 여러 디바이스로 접속" %}

실제 동일한 사용자이지만 여러 디바이스를 통해 접속 하므로 각각의 디바이스 마다 Session과 Cookie가 생성되는 것과 동일한 상황이라 볼 수 있습니다.

그러면 위와 같은 문제를 어떻게 해결해야 할까요? 아쉽게도 Session 만으로는 한계가 있습니다. 여기까지가 Session의 역할은 다 했다고 생각합니다.

---

#### Database storage

마지막으로 DB에 Session을 저장하는 방법입니다. 이 부분은 file Storage에서의 Session 의 한계를 보완하고자 DB를 통해 로그인 기능이 필요한 이유 때문에 준비했습니다.

{% image fancybox center clear group:travel example2.png "Session을 저장하기 위한 DB" %}

위 그림은 app 서버는 2개이고 DB 서버는 1개인 상황입니다.

사용자는 회원 가입을 하고 로그인을 합니다. DB 에는 사용자 정보를 저장하고 더 이상 Session으로 사용자 정보를 저장할 필요가 없습니다. 로그인을 통해 장바구니에 상품을 담는 것도 Session 대신에 DB에 저장하면 동일 유저의 여러 디바이스에서 로그인만 하면 내 장바구니 정보는 DB에서 불러오기만 하면 됩니다. 

그러면 '굳이 Session을 DB 에 저장할 필요가 없지 않나?' 라는 생각이 하실겁니다. Session의 역할은 사용자의 정보를 저장하는 것 이외에 생명주기를 설정할 수 있습니다. 쉽게 말해서 로그인을 유지하기 위한 시간을 설정 할 수 있습니다. 

Session을 DB 로 관리하는 구현 방법을 설명하기에는 개발 환경마다 차이가 있으므로 대신에 제가 사용 중인 [Parse](https://parseplatform.org/) 오픈소스에서는 아래와 같이 사용자의 Session 을 관리하고 있습니다.

{% image fancybox center clear group:travel dbstorage.png "Parse Dashboard Session Collection" %}

MongoDB 를 사용 중이며 위 그림의 `Session` Collection 을 살펴보면, sessionToken 과 expiresAt(로그인 유지 만료시간), user(사용자 외래키) 속성이 있습니다. sessionToken 을 자세히 설명하면 사용자가 로그인 후에 주어지는 입장권 같은 것입니다. 입장권이 있으면 서비스를 이용할 때, 매번 로그인(ID,PW 입력)을 할 필요 없이 DB 에서는 sessionToken(입장권) 을 비교하고 expiresAt(만료일)이 지났는지 확인합니다. 사용자는 더 간편하게 서비스를 이용 할 수 있죠. 또한 동일한 사용자이면서 여러 디바이스 마다 recode 를 생성하여 1:N 관계로 Session을 관리 할 수 있게 됩니다.

## 마치며

이번 글을 정리하면서 느낀점은 개념에 대한 이해를 넘어서 기존의 문제점을 분석하고 더 나은 방법을 채택하는 이유에 대해서 파헤쳐보는 것에 프로그래밍과는 또 다른 흥미를 느꼈습니다.

번외로 개발자분들이 Session과 Cookie에 대해 얼마나 알고 계신지 궁금하여 생활코딩을 통해서 설문조사를 진행 했습니다. 
{% image fancybox center clear group:travel facebook.jpeg 80% %}
글을 쓰면서 생각해보니 `Session을 DB 에 저장하는 이유를 알고 있다` 라는 항목을 잘 못 적었다는 생각이 드네요. 정확히는 `Session의 한계를 알고 로그인 기능이 필요한 이유를 알고 있다` 라고 했어야 했네요.