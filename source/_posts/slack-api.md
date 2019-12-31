---
title: slack 을 이용한 모니터링 알림 구축하기
date: 2019-12-31 23:22:11
# description: NoSQL의 대표적인 mongoDB 데이터 관계를 모델링하는 방법
tags:
 - slack
 - monitoring
categories:
 - api
thumbnailImage: thumbnail.png
thumbnailImagePosition: left
autoThumbnailImage: yes
coverSize: full
coverMeta: in
coverImage: cover.jpg
---

이번 포스팅은 Slack 을 이용하여 서비스 운영시, 모니터링을 할 수 있는 방법에 대해 살펴 보겠습니다.

<!-- excerpt -->


## Slack 구조 이해하기

Slack 은 하나의 작업공간(Workspace)에 여러개의 채널(channels)로 관리합니다. 
예시로 작업공간은 기업 또는 팀이고 채널은 각 부서로 비유할 수 있습니다.

## Slack API 다루기

Slack API 를 다루기 위해서는 기본적으로 Workspace 와 App 을 생성해야합니다.

#### Slack Workspace 생성

[Slack](#https://slack.com/) 페이지에 접속하여 Workspace 를 생성합니다.

{% image fancybox center clear workspace.png 80% "workspace 생성" %}

회원가입 없이 이메일 인증이 가능한 이메일을 기입하여 계속 진행 합니다.

몇가지 질문에 대답만 하면 쉽게 Workspace 를 생성 할 수 있습니다.

#### Slack App 생성

[Slack API](#https://api.slack.com/apps) 페이지에 접속하여 App 을 생성합니다.

{% image fancybox center clear create_app_1.png 80% "App 생성 1" %}

{% image fancybox center clear create_app_2.png 80% "App 생성 2" %}

앱 생성이 끝난 후, 화면의 하단에 생성된 자신의 App을 클릭합니다.

#### App Webhook 생성

{% image fancybox center clear webhook_1.png 80% "Webhook 생성 1" %}

{% image fancybox center clear webhook_2.png 80% "Webhook 생성 2" %}

{% image fancybox center clear webhook_3.png 80% "Webhook 주소" %}

생성한 웹훅 주소는 메시지를 Slack 에 보내기 위한 주소입니다. 이 주소를 복사하여 다음 과정의 코드에 붙여 넣을 것입니다.

## 코드 작성하기

#### 패키지 설치

간단히 2가지 패키지만 설치 합니다.

1. [axios](#https://www.npmjs.com/package/axios) - http 전송 패키지

2. [lodash](#https://www.npmjs.com/package/lodash) - 데이터를 쉽게 다루기 위한 패키지

#### slack.js 코드 파일

```js

const axios = require('axios');
const _ = require('lodash');

class Slack {

  // 색상으로 메시지를 꾸밀수 있습니다.
  static get Colors() {
    return {
      primary: '#007bff',
      info: '#17a2b8',
      success: '#28a745',
      warning: '#ffc107',
      danger: '#dc3545'
    };
  }

  // 메시지를 전송할 webhook 주소
  static get Channels() {
    return {
      devhaks: 'https://hooks.slack.com/services/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'
    };
  }

  /**
   * @static
   * @summary slack 채널에 메시지 보내는 함수
   * 
   * @param {Array} attachments
   * @param {String} url
   * 
   * @example
   * 
    "attachments": [
        {
            "fallback": "Required plain-text summary of the attachment.",
            "color": "#36a64f",
            "pretext": "Optional text that appears above the attachment block",
            "author_name": "Bobby Tables",
            "author_link": "http://flickr.com/bobby/",
            "author_icon": "http://flickr.com/icons/bobby.jpg",
            "title": "Slack API Documentation",
            "title_link": "https://api.slack.com/",
            "text": "Optional text that appears within the attachment",
            "fields": [
                {
                    "title": "Priority",
                    "value": "High",
                    "short": false
                }
            ],
            "image_url": "http://my-website.com/path/to/image.jpg",
            "thumb_url": "http://example.com/path/to/thumb.png",
            "footer": "Slack API",
            "footer_icon": "https://platform.slack-edge.com/img/default_application_icon.png",
            "ts": 123456789
        }
      ]
   */
  static async sendMessage(message) {
    if (!message) {
      console.error('메시지 포멧이 없습니다.');
      return;
    }

    const data = {
      mrkdwn: true,
      text: '',
      attachments: []
    };

    if (_.isString(message)) {
      data.text = message;
    } else {
      if (!message.title && !message.text) {
        console.error('메시지 내용이 없습니다.');
        return;
      }

      message.ts = Math.floor(Date.now() / 1000);
      message.footer = `From 알림 서버 [${process.env.NODE_ENV}]`;
      data.attachments.push(message);
    }

    axios({
      url: this.Channels.devhaks,
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      data
    });
  }
}

module.exports = Slack;


```

## 테스트 메시지 전송하기

Sample 1
```js
Slack.sendMessage(
  {
    color: Slack.Colors.success,
    title: '테스트 메시지 전송 success'
  }
);
```

{% image fancybox center clear sample1.png 80% "샘플 1 메시지" %}

Sample 2
```
Slack.sendMessage(
  {
    color: Slack.Colors.danger,
    title: '테스트 메시지 전송 danger',
    text: 'text를 기입하세요.',
    fields: [
      {
        title: 'title1',
        value: 'value1'
      },
      {
        title: 'title2',
        value: 'value2'
      },
      {
        title: 'title3',
        value: 'value3'
      }
    ]
  }
);

```

{% image fancybox center clear sample2.png 80% "샘플 2 메시지" %}


## 마무리

자신의 상황에 맞게 알림을 받을 채널을 생성하고 웹훅주소를 생성하여 쉽게 모니터링 알림을 받아 볼 수 있을 것입니다.

## 참고

- Message 포멧 설정 - https://api.slack.com/reference/surfaces/formatting

- Message builder - https://api.slack.com/docs/messages/builder