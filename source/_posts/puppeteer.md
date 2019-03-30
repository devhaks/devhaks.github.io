---
title: '[nodejs] puppeteer 뉴스 기사 메타 정보 크롤링'
date: 2019-03-27 22:37:48
description: headless browser 라이브러리 puppeteer 를 이용하여 뉴스 기사의 제목, 요약문, 썸네일의 정보를 수집 합니다.
tags:
  - puppeteer
  - crawler
  - headless browser
categories:
  - nodejs
  - javascript
thumbnailImage: thumbnail.png
thumbnailImagePosition: left
coverSize: full
coverMeta: out
coverImage: cover.png
---

이번 포스팅은 저희 회사 서비스 <b>기업 관리 통합 솔루션 [질링스](https://www.zillinks.com)</b> 의 기능 중에 하나인 기업 뉴스 메타 정보 크롤링하는 방법에 대해 설명하려고 합니다.

<!-- excerpt -->

안녕하세요. 저는 현재 <b>기업 관리 통합 솔루션 [질링스](https://www.zillinks.com)</b>의 SW 리더로서 시스템 관리 및 백앤드 개발을 담당하고 있습니다. 최근 서비스에서 기업의 <u>뉴스 요약문</u>을 수집해야 하는 이슈가 생겼습니다. 그러면 어떠한 과정을 거쳐 뉴스 요약문을 수집 했는지에 대해 글을 써보도록 하겠습니다. 

이 글은 단순히 기능 설명이 아닌 저의 경험 지식을 녹여내려 쓴 것이므로 내용이 많을 수 있습니다. 

---

## 배경

질링스는 기업의 정보를 쉽게 관리 할 수 있도록 지원합니다. 그 기능 중에 하나인 <b>기업 뉴스 관리</b> 기능이 있습니다. 질링스는 현재 사용자가 직접 뉴스를 관리 하지 않아도 기업의 뉴스를 주기적으로 수집하는 스케쥴러 서버를 통해 뉴스를 저장하고 있습니다. 이때 기본적으로 저장하는 데이터는 <u>주소, 제목, 언론사, 발행일자</u> 입니다. 따로 <u>기사 요약문</u>을 저장하지는 않았습니다.

그런데 질링스 고객사인 [스타트업 엑셀러레이터 엔슬파트너스](http://www.enslpartners.com)가 질링스에서 수집한 뉴스를 엔슬파트너스 홈페이지에 보여주고 싶다는 needs 와 추가적으로 <u>기사 요약문</u>도 가능한가라는 얘기를 듣고 이 부분을 기획하고 개발하기로 결정하였습니다. 

{% alert info %}
질링스는 기업의 정보를 REST API(Application Programming Interface)로 제공 하고 있습니다.
{% endalert %}

#### 개발 목적

사용자가 뉴스 정보 쉽게 관리 할 수 있도록 지원

#### 개발 목표

국내 몇백개가 되는 언론사의 <b>뉴스 기사 메타 정보</b> 자동 수집

#### 개발 사항 

1. 사용자측 기업 뉴스 관리 기능 강화
   - 기존 방식 : 사용자가 직접 기사의 <u>주소, 제목, 언론사, 발행일자</u>를 기입해야함
   - 새로운 방식 : 기사의 주소 기입시, <u>제목, 언론사, 요약문</u> 자동 입력됨

2. 스케쥴러 서버에서 요약문도 같이 수집하도록 개선

---

## 웹 페이지 크롤링 방법

웹 페이지를 크롤링하는 방법은 크게 2가지로 나눌 수 있습니다. 

1. 웹페이지 주소로 Method GET 요청
2. Headless browser 라이브러리를 사용

### 1. 웹페이지 주소로 Method GET 요청하기

Get 요청에도 Case 가 다른 경우가 있습니다. 웹 사이트가 어떤 유형으로 만들어진 것인지에 따라 응답 결과가 다릅니다. 간단히 Static vs Dynamaic 2가지 유형으로 나눌 수 있습니다. 기술적인 이해를 돕고자 2가지 유형의 결과와 함께 설명하겠습니다.

#### 1-1. 정적(Static) 사이트
```bash
// linux cmd 에서 실행
$ curl https://devhaks.github.io
```

위 요청 결과는 아래 링크의 내용과 같습니다. 
https://github.com/devhaks/devhaks.github.io/blob/master/index.html

이 결과는 웹페이지의 콘텐츠가 미리 생성된 것을 랜더링 해주기 때문입니다. 

#### 1-2. 동적(Dynamic) 사이트

Demo: http://fiddle.jshell.net/leejognhak/oe0x2r8u/show/#/user/foo

```bash
// linux cmd 에서 실행
$ curl http://fiddle.jshell.net/leejognhak/oe0x2r8u/show/#/user/foo
```

응답 결과

```html

<!DOCTYPE HTML>
<html>
  <head>
    ...(생략)
  </head>
  <body>
    <div id="wrapper">
      <header>
        <h1><a href="//jsfiddle.net/leejognhak/oe0x2r8u/?utm_source=website&amp;utm_medium=embed&amp;utm_campaign=oe0x2r8u" target="_blank">Edit in JSFiddle</a></h1>
        <div id="actions">
          <ul class="normalRes">
              <li class=&quot;active&quot;>
                <a data-trigger-type="result" href="#Result">Result</a>
              </li>
          </ul>
          <div class="hl"></div>
        </div>
      </header>

      <div id="tabs">
        <div class="tCont result active" id="result"></div>
          <script type="text/javascript">
            window.addEventListener('load', function(){
              if (typeof(EmbedManager) === undefined){
                  EmbedManager.loadResult();
              }
            }, false);
          </script>
      </div>
    </div>
  </body>
</html>
```

데모용 페이지를 열어보면 각 페이지 별로 `Home, Profile, Posts` 단어가 존재 합니다. 위의 페이지 소스 결과를 보면 body 태그 안에는 내용이 없습니다. 즉, 동적 사이트는 미리 만들어진 콘텐츠를 보여주는 것이 아니라 필요할 때마다 프로그래밍으로 제작해둔 콘텐츠를 보여주는 것입니다. 


#### 1-3. GET 방법의 한계점

##### 1-3-1. 한글 깨짐

국내 언론사의 뉴스 기사를 예로 들겠습니다.

```bash
$ curl https://www.mk.co.kr/news/it/view/2019/03/191946/
```
응답 결과

```html
<!DOCTYPE html>
<html lang="ko">
    <head>
        <title>[ī�崺��] AI�� ���躸�� �ô롦�����ɷ� ���� 1����? - ���ϰ���</title>
        <meta http-equiv="Content-Type" content="text/html; charset=euc-kr">
        <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
        <meta property="article:author" content="�ſ���"/>
        <meta name="apple-mobile-web-app-title" content="mk"/>
        <link rel='stylesheet' type='text/css' href='https://common.mk.co.kr/common/css/2015/news_2015.css'>
        <link rel='stylesheet' type='text/css' href='https://common.mk.co.kr/common/css/2019/error.css'>
        <link rel='shortcut icon' href='//img.mk.co.kr/main/2015/mk_new/favicon_new.ico'>
        <link rel='canonical' href='https://www.mk.co.kr/news/it/view/2019/03/191946/'>
        <link rel='amphtml' href='http://m.mk.co.kr/news/amp/headline/2019/191946?PageSpeed=off'>
        <link rel='alternate' media='only screen and ( max-width: 640px)' href='http://m.mk.co.kr/news/it/2019/191946/'>
        <meta name='title' content='[ī�崺��] AI�� ���躸�� �ô롦�����ɷ� ���� 1����? - ���ϰ���'>
        <meta name='description' content='[ī�崺��] AI�� ���躸�� �ô롦�����ɷ� ���� 1����?, �ۼ���-�ſ���, ����-it, ����-LG CNS�� ������ 12�� ��������(KorQuAD��The Korean Question Answering Dataset)������ ���� ù AI(�ΰ�����)�� ������ ������. �����ڵ��� �ڻ� AI ���α׷� �ɷ��� ���������� ������ �� �ֵ��� ��������. AI�� �����ڰ�'>
        <meta name='classification' content='it'>
        <meta property='article:published'  content='2019-03-29' />
    </head>
    <body>
      ...(생략)
    </body>
</html>
```
브라우저로 접속하면 한글로된 글이 보이지만, GET 요청 결과는 한글이 깨진 상태입니다. 

```html
<meta http-equiv="Content-Type" content="text/html; charset=euc-kr">
```
그 이유는 위 소스코드의 `charset`이 `euc-kr` 이기 때문입니다. 현대에 만들어진 사이트는 대부분 `utf-8`이지만 오래된 홈페이지의 경우 `euc-kr`인 경우가 있습니다.

##### 1-3-2. 페이지 리다이렉트

이 경우는 url 로 접속 했는데, 다른 url 주소로 튕기는 것입니다. 리다이렉트가 발생되는 시점은 해당 사이트가 어떻게 처리 했는가에 따라 다릅니다. 서버에서 처리 했는지 또는 브라우저의 자바스크립트에 의해 처리 되었는지 파악하기에 무리 입니다.  

{% alert info %}
GET 요청 방법으로 웹페이지 크롤링 하려는 경우, <u>정적 사이트 & 콘텐츠가 깨지지 않는 사이트 & 리다이렉트 되지 않는 페이지</u>를 대상으로 하시는 것을 권장합니다.
{% endalert %}




### 2. Headless browser 라이브러리를 사용하기

앞서 GET 요청 방법으로는 기능을 개발하는데 한계점이 있습니다. 그 한계점을 극복하고자 두번째 방법을 채택하였습니다.

#### Headless browser

우리가 일상적으로 사용하는 IE, 크롬, 사파리 등의 브라우저는 사용하기 쉽게 GUI(Graphical User Interface) 를 제공합니다. 그러면 Headless 란 의미는 단순히 GUI 를 지원하지 않는다는 것입니다. GUI 를 제공하지 않으면 어떻게 조작을 할 수 있지 라는 의문은 프로그래밍을 통해서 가능합니다. 사용자가 브라우저를 열고 마우스 클릭이나 타이핑 등의 행동을 똑같이 프로그래밍으로 자동화 할 수 있습니다. 

#### Puppeteer 

[Puppeteer](https://github.com/GoogleChrome/puppeteer) 는 구글 Chrome 또는 Chromium 브라우저를 조작할 수 있도록 도와주는 [nodejs](https://nodejs.org) 의 패키지 라이브러리 입니다. 

Puppeteer 로 할 수 있는 기능은 다음과 같습니다.

- <b>SPA (단일 페이지 응용 프로그램)를 크롤링</b>하고 사전 렌더링 된 콘텐츠 (즉, "SSR"(서버 측 렌더링))를 생성합니다.
- 페이지의 <b>스크린 샷 및 PDF 생성</b>
- 양식 제출, UI 테스트, 키보드 입력 등을 자동화 합니다.

이외 더 많은 기능은 [Puppeteer API 문서](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md)를 참고하시기 바랍니다.

#### 설치

[cheerio](https://github.com/cheeriojs/cheerio) 는 웹페이지 html 을 파싱해주는 라이브러리 입니다. 파싱 후에는 jQuery 처럼 사용 할 수 있습니다.

``` bash
$ npm i puppeteer cheerio
```

puppeteer 를 설치하기 위해서는 각 OS별로 [의존성 패키지를 설치](https://github.com/GoogleChrome/puppeteer/blob/master/docs/troubleshooting.md#chrome-headless-doesnt-launch) 해야합니다.
개발용으로 사용하는 PC에는 이미 패키지가 설치되어 있을 수 있기 때문에 puppeteer 설치 오류가 발생하지 않을 수 있습니다.

만약 오류가 발생할 경우, 아래 경로로 찾아갑니다. (디렉토리명이 다를 수 있습니다.)

```bash
$ cd project/node_modules/puppeteer/.local-chromium/linux-624492/chrome-linux

$ ldd chrome | grep not # 설치되지 않은 패키지 목록을 출력합니다.
#Output not install package
...

# ubuntu 패키지 설치 예시
$ sudo apt-get install libatspi-dev libatspi2.0-0 libgtk-3-0
```







{% alert info %}
Puppeteer를 설치하면 API가 작동하는 Chromium의 최신 버전 (~ 170MB Mac, ~ 282MB Linux, ~ 280MB Win)이 다운로드됩니다.
{% endalert %}

{% alert warning %}
Puppeteer는 노드 v6.4.0 이상을 필요로하지만 설명할 코드의 예제는 노드 v7.6.0 이상에서만 지원되는 async / await를 사용합니다.
{% endalert %}

#### 시나리오 설명

앞서 예시로 사용한 https://www.mk.co.kr/news/it/view/2019/03/191946/ 기사의 메타 정보를 수집하도록 하겠습니다.

사람이 수집하는 것처럼 puppeteer 를 통해 똑같이 수집 하도록 순서를 나열 해보겠습니다.

1. 브라우저를 띄운다.
2. 새 탭을 생성한다.
3. 새 탭에 뉴스 기사 주소를 입력해서 접속한다.
4. 웹페에지의 페이지 소스를 확인한다.
5. 페이지 소스에서 아래 속성 값을 가진 `<meta>` 태그의 `content` 속성 값을 추출한다.
   - 제목: title, og:title, twitter:title
   - 요약문: description, og:description, twitter:description
   - 썸네일: og:image, twitter:image
6. 새 탭 닫기
7. 브라우저 닫기



아래 코드는 질링스 서비스의 뉴스 자동 수집 기능에 사용한 일부분 입니다.

```js
const puppeteer = require('puppeteer');
const cheerio = require('cheerio');

(async () => {

  // 기사 메타 정보 추출하기 
  // 제목, 요약문, 썸네일 정보를 추출합니다.
  function extractNewsData(html){
    const newsData = {
      title: null,
      description: null,
      image: null
    };

    const $ = cheerio.load(html);
    const $metaList = $('meta');

    for (let index = 0; index < $metaList.length; index += 1) {
      const element = cheerio($metaList[index]);

      let content = element.attr('content');

      if (!content || !content.trim()) {
        continue;
      }
      content = content.trim();

      let propertyAttr = element.attr('property');
      if (propertyAttr) {
        propertyAttr = propertyAttr.toLocaleLowerCase();
      }

      switch (propertyAttr) {
        case 'og:title':
          newsData.title = content;
          break;
        case 'og:description':
          newsData.description = content;
          break;
        case 'og:image':
          newsData.image = content;
          break;
        default:
          break;
      }

      let nameAttr = element.attr('name');
      if (nameAttr) {
        nameAttr = nameAttr.toLocaleLowerCase();
      }

      switch (nameAttr) {
        case 'title':
        case 'twitter:title':
          newsData.title = content;
          break;
        case 'description':
        case 'twitter:description':
          newsData.description = content;
          break;
        case 'twitter:image':
          newsData.image = content;
          break;
        default:
          break;
      }
    } // end for

    return newsData
  } // end extractNewsDate()

  // 브라우저 옵션 설정
  const browserOption = {
    headless: false // 디버깅용으로 false 지정하면 브라우저가 자동으로 열린다.
  };

  // 1. 브라우저를 띄운다.
  const browser = await puppeteer.launch(browserOption);

  // 2. 새 탭을 생성한다.
  const page = await browser.newPage();
  
  let response;
  try {
    // 리다이렉트 되는 페이지의 주소를 사용.
    const url = 'http://www.thebell.co.kr/front/free/contents/news/article_view.asp?key=201807250100046030002891';

    // 탭 옵션
    const pageOption = {
      // waitUntil: 적어도 500ms 동안 두 개 이상의 네트워크 연결이 없으면 탐색이 완료된 것으로 간주합니다.
      waitUntil: 'networkidle2',
      // timeout: 20초 안에 새 탭의 주소로 이동하지 않으면 에러 발생
      timeout: 20000  
    };

    // 3. 새 탭에 뉴스 기사 주소를 입력해서 접속한다.
    response = await page.goto(url, pageOption);
  } catch (error) {
    await page.close();
    console.error(error)
    return;
  }

  let html;
  try {
    // 4. 웹페에지의 페이지 소스를 확인한다.
    html = await response.text();
  } catch (error) {
    console.error(error);
    return;
  } finally {
    // 6. 새 탭 닫기
    await page.close();
  }

  // 7. 브라우저 닫기
  browser.close()

  // 5. 페이지 소스에서 아래 속성 값을 가진 `<meta>` 태그의 `content` 속성 값을 추출한다.
  const newsData = extractNewsData(html);

  // 크롤링 결과
  console.log(newsData)

})();
```

- 프로그래밍 순서상 5번 가장 나중에 온 이유는 브라우저를 종료해도 정보를 추출 할 수 있기 때문입니다. 

- `page.goto(url, pageOption)` 함수에 리다이렉트 되는 url로 예시를 든 이유는 GET 요청 방법의 한계점을 해결 할 수 있기 때문입니다.

- [Browser option](https://github.com/GoogleChrome/puppeteer/blob/v1.14.0/docs/api.md#puppeteerlaunchoptions)

- [Page option](https://github.com/GoogleChrome/puppeteer/blob/v1.14.0/docs/api.md#pagegotourl-options)

<br>
{% image fancybox center clear crawl_news.png crawl_news.png 75% "실제 서비스 기능 화면" %}


#### Tips

<b>Page Event</b> - 불필요한 리소스 막기

어떤 리소스를 허용 또는 막을지를 설정하여 페이지 로드 시간을 단축 할 수 있습니다.
뉴스를 크롤링 하는데 `image, stylesheet, font`같은 용량이 많은 리소스는 다운로드 시간이 오래 걸리기 때문에 시간 단축에 효과적입니다.

```js

const blockResource = [
  'image', 
  'script', 
  'stylesheet', 
  'xhr', 
  'font', 
  'other'
];

await page.setRequestInterception(true);

page.on('request', req => {
  // 리소스 유형
  const resource = req.resourceType(); 
  if (blockResource.indexOf(resource) !== -1) {
    req.abort();  // 리소스 막기
  } else {
    req.continue(); // 리소스 허용하기
  }
});

```

<b>Page Event</b> - Window.alert 강제 끄기

해당 페이지에 dialog 가 발생하면 해당 페이지만 정지 상태가 됩니다. 여러 페이지를 크롤링하는 상황에서 정지 상태로 내버려 두면, 페이지를 종료 할 수 없기 때문에 메모리 누적 문제가 발생합니다.
dialog 를 강제로 끄면 페이지 옵션 `timeout` 시간 초과에 의해 페이지가 종료 됩니다.

```js
page.on('dialog', async dialog => {
  console.log(`dialog message:' ${dialog.message()}`);
  await dialog.dismiss();
});
```

<b>CPU, Memory 사용량</b>

Headless browser 를 사용함으로써 개발은 확실히 편하지만, 컴퓨팅 자원을 많이 사용하기 때문에 제한된 컴퓨팅 자원으로 어떻게 효율적으로 사용할 지를 고민 해야할 것 같습니다.

---

## 마치며...

뉴스 크롤링 기능을 구현하면서 다양한 CASE 를 마주하였고 어떻게 풀어 나갈지에 대해 고민한 부분을 정리하였습니다. 이 글을 보신 분은 최대한 도움이 되셨으면 합니다. 

피드백은 언제나 환영합니다.