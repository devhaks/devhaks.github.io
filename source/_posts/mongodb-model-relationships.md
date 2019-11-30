---
title: MongoDB 데이터 관계 모델링
date: 2019-11-30 16:08:01
description: NoSQL의 대표적인 mongoDB 데이터 관계를 모델링하는 방법
tags:
 - mongoDB
 - data relationships
 - data modeling
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

이번 포스팅은 NoSQL의 대표적인 mongoDB 는 어떻게 데이터 관계를 모델링 하는지 살펴 보겠습니다. 

<!-- excerpt -->

## 목차

* [배경](#배경)

* [Document 관계 데이터 저장 유형](#Document-관계-데이터-저장-유형)
 * [Embedded](#Embedded)
 * [Reference](#Reference)

* [Document 관계 유형](#Document-관계-유형)
 * [One-to-One](#One-to-One)
 * [One-to-Many](#One-to-Many)
  * [시나리오 1](#시나리오-1)
  * [시나리오 2](#시나리오-2)
 * [Many-to-Many](#Many-to-Many)

* [마치며](#마치며)

* [참고](#참고)

## 배경

학교 프로젝트를 진행 했을 때, 처음으로 접한 데이터베이스는 RDBMS 중에 하나인 MySQL 을 사용했었습니다. 그 이후로 전세계 점유율이 가장 높은 CMS(Content Management System) 인 Wordpress 를 사용하여 외주 개발을 하면서도 MySQL에 점점 익숙해졌던것 같습니다. 그리고 Wordpress가 MySQL 을 어떻게 사용하고 있는지 Data modeling 관점에서도 현재까지 아주 많은 도움이 되고 있습니다. 

지금 다니는 회사는 MongoDB를 사용하고 있는데, 회사의 서비스를 초창기 개발하면서 NoSQL(
Not Only SQL)인 MongoDB 를 처음으로 접했습니다. NoSQL 이라고 해서 관계가 없는 데이터베이스라고 생각했었지만 찾아본 내용으로는 <span style="color:red">NoSQL이 무엇의 약자인지는 사람에 따라 NoSQL, Not Only SQL, Non-Relational Operational Database SQL로 엇갈리는 의견들이 있습니다만, 현재 Not Only SQL로 풀어 설명하는 것이 다수를 차지하고 있다고 합니다.</span>

즉, 쉽게 말하면 `관계형 데이터베이스 + @` 라고 이해하면 쉽겠네요. 그러면 의문 생기는데 <span style="text-decoration: underline;">"과연 RDMBS 처럼 관계 모델링 방법이 같을까요?"</span>

이에 대한 의문을 해소하기 위해 MongoDB 공식 문서를 살펴 보겠습니다. 

## Document 관계 데이터 저장 유형

Document에 데이터를 저장 할 때, 어떠한 데이터 유형으로 저장 할 수 있을까요? 다들 아시는 것처럼 여러가지가 있습니다. Number, String, Boolean, Date, Array, Object, Pointer 등이 있습니다. 데이터베이스 마다 지원하는 데이터 유형은 제각각입니다. 여기서는 MongoDB 에 대해서만 설명하겠습니다.

그러면 mongoDB는 관계 모델링을 할 때, 어떤 데이터 유형을 사용해서 관계를 저장 할까요? 

#### Embedded 

Embedded 저장 방법은 2가지 종류의 Document가 있을 때, 1개의 Document 데이터를 다른 Document key의 value에 저장하는 방법입니다.

2가지 종류의 Person, Address Document가 있습니다.
```json
// Person 
{
   _id: "joe",
   name: "Joe Bookreader"
}

// Address
{
   pataron_id: "joe",
   street: "123 Fake Street",
   city: "Faketon",
   state: "MA",
   zip: "12345"
}
```

위의 Document를 Embedded 방식으로 관계를 저장하면 다음과 같습니다. `Person.address` 에 Address Document가 통째로 저장되어 있는 것을 확인 하실 수 있습니다.

```json
// Person 
{
   _id: "joe",
   name: "Joe Bookreader",
   address: {
      street: "123 Fake Street",
      city: "Faketon",
      state: "MA",
      zip: "12345"
  }
}

// Address
{
   pataron_id: "joe",
   street: "123 Fake Street",
   city: "Faketon",
   state: "MA",
   zip: "12345"
}
```

#### Reference

Reference 저장 방법은 pointer 개념으로 이해하면 쉬울 것 같습니다. Embedded 방식과 달리 Document를 통째로 저장하는것이 아니라 참조 할 수 있도록 ID를 저장하는 것입니다.

2가지 종류의 Publisher, Book Document가 있습니다. Book Document를 유심히 확인해 봅시다. 어떤게 눈의 띄는지 보이시나요? 바로 `Book.publisher_id` 에 `Publisher._id` 의 value 가 저장되어 있습니다.

```json
// Publisher
{
   _id: "oreilly",
   name: "O'Reilly Media",
   founded: 1980,
   location: "CA"
}

// Book
{
   _id: 123456789,
   title: "MongoDB: The Definitive Guide",
   author: [ "Kristina Chodorow", "Mike Dirolf" ],
   published_date: ISODate("2010-09-24"),
   pages: 216,
   language: "English",

   publisher_id: "oreilly" // <- Publisher._id
}

```

2가지 저장 방식을 통하여 이제 어떻게 데이터를 저장하여 관계를 맺을수 있는지 살펴 보았습니다.

## Document 관계 유형

이 부분은 관계를 `얼마나` 맺을 것인가에 대한 설명입니다.

#### One-to-One

단순한 1:1 관계입니다. 이 관계 유형을 비유하자면, [Embedded](#Embedded) 예시를 든 것처럼 Person이 실제 주민등록등본상에 거주지가 Address 인 것으로 시나리오를 가정하는 관계 유형입니다.

#### One-to-Many

앞서 [Reference](#Reference)의 예제를 재활용하여 이번에는 1개의 Publisher Document 와 2개의 Book Document 가 있습니다.

이 예제의 가정은 `Book 은 무조건 1개의 Publisher 만 갖는다` 라고 설정하겠습니다. 

[Embedded](#Embedded) 방식으로 했을 때, 데이터 구조는 어떻게 저장 될까요? `Book.publisher`의 value 에는 `Publisher` 데이터가 통째로 저장되어 있네요. 

```json
// Publisher
{
   _id: "oreilly",
   name: "O'Reilly Media",
   founded: 1980,
   location: "CA"
}

// Book 1
{
   _id: 123456789,
   title: "MongoDB: The Definitive Guide",
   author: [ "Kristina Chodorow", "Mike Dirolf" ],
   published_date: ISODate("2010-09-24"),
   pages: 216,
   language: "English",

   publisher: {
      name: "O'Reilly Media",
      founded: 1980,
      location: "CA"
   }
}

// Book 2
{
   _id: 234567890,
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English",

   publisher: {
      name: "O'Reilly Media",
      founded: 1980,
      location: "CA"
   }
}
```

[Reference](#Reference) 방식으로 했을 때, 2개의 `Book.publisher_id` 의 value 는 `Publisher._id` value 가 저장되어 있습니다.

```json
// Publisher
{
   _id: "oreilly",
   name: "O'Reilly Media",
   founded: 1980,
   location: "CA"
}

// Book 1
{
   _id: 123456789,
   title: "MongoDB: The Definitive Guide",
   author: [ "Kristina Chodorow", "Mike Dirolf" ],
   published_date: ISODate("2010-09-24"),
   pages: 216,
   language: "English",

   publisher_id: "oreilly" // <- Publisher._id
}

// Book 2
{
   _id: 234567890,
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English",

   publisher_id: "oreilly" // <- Publisher._id
}
```

One-to-Many 관계에서 Embedded 와 Reference 방식으로 비교하여 봅시다. 그냥 데이터가 저장된 형태만 봐서는 비교가 되질 않을 것 같습니다. 비교를 하기 위해 적절한 시나리오를 설정하겠습니다.

##### 시나리오 1

만약 publisher 의 'name' 을 변경하거나 'age' 라는 데이터를 추가해야 하는 경우에 DB를 수정해야 하는데, 어떻게 수정해야 할까요?

Embedded 방식으로 저장된 데이터는 Publisher, Book 의 Document 를 모두 수정해서 데이터의 `일관성`을 유지해야 합니다. 복잡하거나 데이터가 자주 변경되는 상황이 생긴다면 `일관성`을 유지하지가 어려워 질 수 있습니다.

Reference 방식으로 저장된 데이터는 Publisher Document 만 수정해주면 참조하고 있는 모든 Document 는 수정할 필요가 없습니다. 자연스럽게 데이터의 `일관성`이 유지가 됩니다.

##### 시나리오 2

다음과 같이 Document 가 모두 관계가 저장되어 있습니다.
- Publisher Document 개수: 100개
- Book Document 개수: 100만개

극단적으로 100만개의 Book Document 를 <span style="color:red">Publisher 정보를 포함</span>하여 불러오려고 할 때, 어떤 차이점을 발견 할 수 있을까요?

Embedded 방식으로 저장된 데이터는 아무런 어려움 없이 Book Document 를 가져 올 때, Publisher 정보를 통째로 가져올 수 있습니다.

Reference 방식으로 저장된 데이터는 `Publisher_id` 에 해당되는 Publisher 정보를 포함하도록 요청해서 가져와야 하기 때문에 한번의 요청만으로는 Publisher 정보를 가져 올 수는 없을 것입니다. 즉, 추가적인 요청을 해야만 Publisher 정보를 가져 올 수 있다는 말입니다.

단편적으로 시나리오 1과 2만을 같이 고려 할 때, Embedded 또는 Reference 둘 중 어떠한 저장 방법이 효율적일지는 서비스의 기획에 따라 달라 질 것 같습니다. 퍼포먼스의 중점을 둔다면 시나리오1을 선택하여 빠른 시간내에 데이터를 불러와야 할 것이고 개발의 속도(효율성, 편의성)에 중점을 둔다면 시나리오2를 선택할 수 있을 것입니다.


#### Many-to-Many

One-to-Many 관계에서 확장된 관계 유형입니다. Document 관계가 m:n 입니다. 

Publisher 와 Book 의 관계를 예시로 들면, One-to-Many 에서 Book의 Publisher 가 1개 이상이 될 수 있는 구조로 되는 것입니다.

One-to-Many 에서 Publisher:Book = 1:n 이었지만, Many-to-Many 는 m:n 이 가능한 구조입니다. 아래 예시를 보시면 `Book.publisher_id` value 가 1개 이상의 `Publisher.id`를 참조하고 있습니다.

```json
// Publisher 1
{
   _id: "oreilly",
   name: "O'Reilly Media",
   founded: 1980,
   location: "CA"
}

// Publisher 2
{
   _id: "devhak",
   name: "devhak'Reilly Media",
   founded: 1980,
   location: "CA"
}


// Book 1
{
   _id: 123456789,
   title: "MongoDB: The Definitive Guide",
   author: [ "Kristina Chodorow", "Mike Dirolf" ],
   published_date: ISODate("2010-09-24"),
   pages: 216,
   language: "English",

   publisher_id: ["oreilly", "devhak"] // <- Publisher._id
}

// Book 2
{
   _id: 234567890,
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English",

   publisher_id: ["oreilly", "devhak"] // <- Publisher._id
}
```


## 마치며

Database 의 구조는 실제 서비스를 구현하는 환경 및 개발 인력, 서비스의 규모에 따라, 많이 다를 수 있다고 생각합니다. 하지만, 지극히 개인적인 생각은 스타트업 처럼 서비스의 규모가 작은 상태에서 서비스의 초기에는 시나리서오 1의 퍼포먼스 보다는 시나리오 2의 개발의 편의성과 효율성에 초점을 두고 진행해야 개발의 속도에 조금이나마 도움이 될 것이라고 생각합니다.

## 참고

- [Model Relationships Between Documents](https://docs.mongodb.com/manual/applications/data-models-relationships/)