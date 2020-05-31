---
title: MongoDB Query 튜닝
date: 2020-01-31 23:59:00
description: 데이터를 검색 속도를 높이기 위한 indexing 하는 방법
tags:
 - mongoDB
 - query
 - NoSQL
categories:
 - Database
thumbnailImage: thumbnail.png
thumbnailImagePosition: left
autoThumbnailImage: yes
coverSize: full
coverMeta: in
coverImage: cover.jpg
---

이번 포스팅은 mongoDB 에서 수많은 데이터를 빠르게 검색하기 위한 indexing 에 대해 살펴보겠습니다.

<!-- excerpt -->


## 목차

* [배경](#배경)

* [Index란](#Index란)

* [Sample DB 설치](#Sample-DB-설치)

* [Index 유형](#Index-유형)
  * [Single field index](#Single-field-index)

* [마치며](#마치며)

* [참고](#참고)



## 배경

자사 서비스 중에 공공데이터 사이트에서 <b>국민연금 open api</b> 를 사용하고 있었습니다.

그러던 중 api측에서 과도한 api 요청으로 부하가 발생하여 국민연금의 2015~2018년 데이터를 제공하지 않겠다고 공지했습니다. 

저희는 기업을 대상으로 서비스를 운영 중이었는데, 더 이상 데이터를 제공받지 못하게 되었습니다.
그래서 국민연금 파일 데이터 제공받아 파일에 있는 국민연금 정보를 DB에 저장하는 작업을 하게 되었습니다.

(파일 데이터로 전환한 또 다른 이유는 api 서버에 여러 문제가 많습니다. 응답 속도도 느릴 뿐더러 서버에서 block 을 피하기 위해 프로그래밍한 것도 너무 짜증이 났습니다...)

2015~2020년 기간의 파일 데이터를 DB 에 저장 작업을 마치고 나서 최종 데이터의 용량, document 수 등을 확인했습니다. DB collection 용량은 약 3GB, Document 수는 90만개가 넘었습니다. 

저는 방대한 데이터를 사용한 적이 없어서 어떻게 데이터를 다룰지 고민 했습니다. 문뜩 생각난 indexing 이란 것을 들어만 봤었는데, 어떻게 사용하는지에 대해서는 전혀 알지 못했기 이참에 공부하여 정리 해봤습니다.

## Index란

Index 라는 단어는 익숙 합니다. 목차라고 하며 책의 앞쪽 부분에 원하는 내용을 쉽고 빠르게 찾기 위해  나열된 소주제와 페이지 수를 정리해 놓은 것입니다. 

만약 목차가 없는 사전에서 '정보'라는 단어를 찾는다면 어떻게 단어를 찾아야 할까요? 

생각만 해도 끔찍한데 그래도 생각해봅시다... 개발자들은 끔찍한걸 경험해봐야 더 효율적으로 생각 하게됩니다.  

책의 'ㄱ' 부터 시작해서 'ㅈ'을 찾은 다음에는 'ㅏ' 부터 시작해서 'ㅓ'를 계속해서 찾아가야 하는 엄청난 불편함이 생깁니다.

그래서 DB에도 이러한 개념이 있는 것이겠죠. 감사한 일입니다...


## Sample DB 설치

Indexing 을 알아 보기전 실습을 하기 위한 sampleDB 를 준비합니다. 이 DB는 국민연금을 납부한 전국의 사업장 정보를 갖고 있습니다. (약 40MB, 73만개의 document)

1. 실습 전에 [mongoDB](https://www.mongodb.com/download-center) 를 설치합니다.
2. [Sample DB 다운로드](./sampleDB.zip)하고 압축을 풀어줍니다.
3. Command 을 열어서 아래와 같이 진행합니다.

```  
// DB 복구하기
$ mongorestore sampleDB // 기본 설정 host=localhost, port=27017 

2020-01-31T23:31:31.770+0900	preparing collections to restore from
2020-01-31T23:31:31.771+0900	reading metadata for sample.NationalPension from sampleDB/sample/NationalPension.metadata.json
2020-01-31T23:31:31.838+0900	restoring sample.NationalPension from sampleDB/sample/NationalPension.bson
2020-01-31T23:31:34.768+0900	[##########..............]  sample.NationalPension  70.8MB/155MB  (45.7%)
2020-01-31T23:31:37.768+0900	[######################..]  sample.NationalPension  146MB/155MB  (94.0%)
2020-01-31T23:31:38.141+0900	[########################]  sample.NationalPension  155MB/155MB  (100.0%)
2020-01-31T23:31:38.142+0900	restoring indexes for collection sample.NationalPension from metadata
2020-01-31T23:31:40.756+0900	finished restoring sample.NationalPension (731815 documents)
2020-01-31T23:31:40.756+0900	done

```
{% alert info %}
  참고로 DB 를 미리 생성하지 않아도 자동으로 DB 를 생성합니다.
{% endalert %}

1) 복구된 데이터 확인하기

```
$ mongo // mongo 콘솔로 접속하기

> use sample  // DB 선택
switched to db sample

// collection document 개수 확인
> db.NationalPension.count()
731815

// collection 상세 정보 보기 , 1024*1024 는 용량 단위 MB 로 환산
> db.NationalPension.stats(1024*1024)
... 출력 생략

```

2) Collection 설명
```
// document field 정보 확인
> Object.keys(db.NationalPension.findOne())
[
  "_id",
  "adptDt",       // 국민연금 최초 등록일자
  "wkplNm",       // 사업장명
  "bzowrRgstNo",  // 사업자번호 앞 6자리
  "wkplJnngStcd", // 사업장 상태 코드 - 1: 정상 / 2: 탈퇴
  "sido",         // 시도
  "sigungu",      // 시군구
  "_created_at",  // document 생성일
  "_updated_at"   // document 수정일
]

// bzowrRgstNo 종류 개수 
> db.NationalPension.distinct("bzowrRgstNo").length
8673
```

## Index 유형

indexing 하는 방법은 여러 가지가 있습니다. 그 중 제가 적용한 방법에 대해서 살펴 보겠습니다.

#### Single field index

1개의 field 만 indexing 합니다. 적용 전, 임의의 쿼리를 실행해 봅시다.

1) 적용 전, 쿼리 실행 결과 상세 정보

- totalDocsExamined: 731,815개
- executionTimeMillis: 325ms

```
> db.NationalPension.find({ bzowrRgstNo: "105879" }).explain('executionStats')
{
  "queryPlanner" : {
    "plannerVersion" : 1,
    "namespace" : "sample.NationalPension",
    "indexFilterSet" : false,
    "parsedQuery" : {
      "bzowrRgstNo" : {
        "$eq" : "105879"
      }
    },
    "winningPlan" : {
      "stage" : "COLLSCAN",
      "filter" : {
        "bzowrRgstNo" : {
          "$eq" : "105879"
        }
      },
      "direction" : "forward"
    },
    "rejectedPlans" : [ ]
  },
  "executionStats" : {
    "executionSuccess" : true,
    "nReturned" : 290,
    "executionTimeMillis" : 325,  //  실행 시간
    "totalKeysExamined" : 0,
    "totalDocsExamined" : 731815, //  document 검사 개수
    "executionStages" : { ... },
  },
  "serverInfo" : { ... },
  "ok" : 1
}
```

2) Indexing 적용하기

```
// 1은 오름차순, -1 내림차순을 의미합니다.
> db.NationalPension.createIndex({ bzowrRgstNo: 1 });
{
  "createdCollectionAutomatically" : false,
  "numIndexesBefore" : 1,
  "numIndexesAfter" : 2,
  "ok" : 1
}

// collection 에 적용된 모든 index 확인
> db.NationalPension.getIndexes()
// 자동으로 _id 로만 indexing 적용된 상태
[
  {
    "v" : 2,
    "key" : {
      "_id" : 1
    },
    "name" : "_id_",
    "ns" : "sample.NationalPension"
  }
]
```

3) 적용 후, 쿼리 실행 결과 상세 정보

- totalDocsExamined: 290개
- executionTimeMillis: 1ms

```
> db.NationalPension.find({ bzowrRgstNo: "105879" }).explain('executionStats')
{
  "queryPlanner" : {
    "plannerVersion" : 1,
    "namespace" : "sample.NationalPension",
    "indexFilterSet" : false,
    "parsedQuery" : {
      "bzowrRgstNo" : {
        "$eq" : "105879"
      }
    },
    "winningPlan" : {
      "stage" : "FETCH",
      "inputStage" : {
        "stage" : "IXSCAN",
        "keyPattern" : {
          "bzowrRgstNo" : 1
        },
        "indexName" : "bzowrRgstNo_1",
        "isMultiKey" : false,
        "multiKeyPaths" : {
          "bzowrRgstNo" : [ ]
        },
        "isUnique" : false,
        "isSparse" : false,
        "isPartial" : false,
        "indexVersion" : 2,
        "direction" : "forward",
        "indexBounds" : {
          "bzowrRgstNo" : [
            "[\"105879\", \"105879\"]"
          ]
        }
      }
    },
    "rejectedPlans" : [ ]
  },
  "executionStats" : {
    "executionSuccess" : true,
    "nReturned" : 290,
    "executionTimeMillis" : 1,  //  실행 시간
    "totalKeysExamined" : 290,
    "totalDocsExamined" : 290,  //  document 검사 개수
    "executionStages" : { ... }
  },
  "serverInfo" : { ... },
  "ok" : 1
}
```

상세 정보 중에 가장 중요한 정보만 비교하면 다음과 같습니다.

- totalDocsExamined: document 검사 개수가 731815 -> 290
- executionTimeMillis: 325ms -> 1ms

Document 검색 개수가 대폭 줄었으며 실행 시간이 무려 325% 증가하였습니다. 

<!-- 
#### compound indexes

2개 이상의 field 를 같이 indexing 합니다. -->

## 마치며

indexing 을 접하고나니 전혀 어렵지 않았고 조금의 설정만으로도 서비스 성능에 만족스러운 결과를 얻었습니다.
아직은 맛보기에 불과하지만 이후에 indexing 을 더 효율적으로 사용하는 방법을 찾아서 적용하려고 합니다.

## 참고

- Indexing Strategies - [Selectivity](https://docs.mongodb.com/manual/tutorial/create-queries-that-ensure-selectivity/)

- 연습용 [mongoDB 웹 쉘](https://mws.mongodb.com/?version=4.2)