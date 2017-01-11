---
layout: post
title:  "이세계에 진입한 서버 개발 - 3"
date:   2017-01-12 00:00:01
categories: nodejs azure webapps unity
comments: true
meta : nodejs azure webapps webservice
description : 기기와 사용자 관리
image : 
  twitter: /images/rvwendy_tutorial_03.png
  facebook: /images/rvwendy_tutorial_03.png
publish : true
---

* content
{:toc}

{% include youtubePlayer.html id="q8reG_gVcUQ" %}

## 들어가는 말
게임에서 사용자는 아이디(혹은 닉네임)을 가지고 있으나 그것외에 서로를 구별할 수 있는 정보가 부족하다.

설상 가상으로 사용자는 스마트폰뿐 아니라 테블릿이나 가상 기기 등 여러 기기로 게임을 즐긴다.

없는 정보로 기기를 구분하고 기기와 사용자를 연결하려면 어떻게 해야할까?

이제부터 데이터베이스를 활용해서 어떻게 이를 관리하는지 알아보자.

> 이번에 다룰 내용에서는 복수의 기기를 한 사용자에게 연결하는 부분은 제외한다.

## utils 파일 추가
프로그래밍에 필요한 파일 3개를 `utils`폴더에 추가해야한다.

* [commonFunc.js](https://gist.githubusercontent.com/totuworld/72ffa52a423d192023514081880b593a/raw/b15d71a14ba6880b3a65f27528d1c658a0bdc9ee/commonFunc.js)
* [auth.js](https://gist.githubusercontent.com/totuworld/72ffa52a423d192023514081880b593a/raw/b15d71a14ba6880b3a65f27528d1c658a0bdc9ee/auth.js)
* [error.js](https://gist.githubusercontent.com/totuworld/72ffa52a423d192023514081880b593a/raw/b15d71a14ba6880b3a65f27528d1c658a0bdc9ee/error.js)

위 3개 파일을 다른이름으로 저장하여 `utils`폴더에 추가한다.

> vs code에서 파일을 생성한 뒤 내용을 복사 붙여넣기해도 무방하다.

## 테이블 정의

친한 친구 중에 `John`이 3명 있다고 해보자. 이름이 같지만 각자가 가진 특징이 달라서 구분할 수 있다.

![john](/images/rvwendy03_m01.png)

하지만 이들이 인터넷 공간에서 `John`으로 불린다면 다 구분할 수 있을까? 프로필 사진이 있어도 그 `John`이 그 `John`이란 보장은 어렵다(도용하면 노답).

이런 문제는 게임에서도 사용자를 구분하는게 쉬운 문제가 아니다. 사용자 이메일이나 소셜미디어 로그인 등을 지원해야 그나마 의미있는 정보를 얻을 수 있다. 그러나 이를 쉽게 요구하기 어렵다.

대안으로 `범용 고유 식별자`(이하 UUID)를 활용하여 기기를 구분하겠다. 사용자는 `별명`(이하 닉네임)만 요구하는 형태로 사용하겠다.

> 각각의 테이블은 하나의 스프레드 시트로 생각하면 된다.

### 기기 테이블
기기 정보를 기록하려면 기록지(혹은 대장)가 필요한 것처럼 관계형 데이터베이스에서도 어떻게 기록할지 정할 필요가 있다.

> 엑셀로 치면 새로운 시트를 추가하고 등록할 정보를 컬럼명으로 적는 것이다.

테이블이 다음과 같다고 해보자.

GameDeviceUID | UUID | DeviceType | GameUserID | MainFlag
--- | --- | --- | --- | ---
1 | 12341234 | iOS | 2 | true
2 | 343434 | aOS | 3 | true

이 테이블의 컬럼을 아래와 같이 정의할 수 있다.

Name | Type | Default | Attributes | Index | AutoIncrease
--- |:--- | --- | --- | --- | ----
GameDeviceUID | INTEGER | - | - | PRIMARY | o
UUID | STRING | - | - | - | -
DeviceType | INTEGER | 0 | - | - | -
MainFlag | BOOLEAN | true | - | - | -
GameUserID | INTEGER | 0 | - | - | -

### 사용자 테이블
사용자 테이블은 닉네임, 로케일, 사용시간대 등을 기록한다.

Name | Type | Default | Attributes | Index | AutoIncrease
--- |:--- | --- | --- | --- | ----
GameUserID | INTEGER | - | - | PRIMARY | o
NickName | STRING | - | - | - | -
Locale | STRING | - | - | - | -
OffsetTime | INTEGER | 0 | - | - | -
OffsetTimeUpdateAt | DATE | now | - | - | -
createAt | DATE | now | - | - | -
loginAt | DATE | now | - | - | -


## 모델 추가

테이블 정의에 따라 모델을 정의해보자.

Sequelize.js를 사용해서 model의 기본구조는 아래와 같다.

```javascript
module.exports = function(sequelize, DataTypes) {
    let _yourTableName = sequelize.define('모델명', { /* 특성 */ }, { /* 옵션 */ });
    return _yourTableName;
};
```

* 특성 : 각 컬럼을 정의하는 내용이 포함된다.
* 옵션 : 필요에 따라 추가하면 된다.

### 기기 모델 정의

앞서 살펴본 테이블에 따라 기기 모델을 정의해보겠다.

1. models폴더에 `GameDevice.js`파일을 만든다.

	> 지난 시간에 vs code로 신규 파일을 생성하는 단계를 설명해서 이는 생략한다.

2. 아래 소스코드를 입력한다.

    {% gist /72ffa52a423d192023514081880b593a GameDevice.js %}

    코드가 길어졌지만 기본구조와 크게 다를게 없다.

    * 10번 줄 : Sequelize.js는 기본으로 업데이트와 생성된 시간을 기록하는 타임스탬프를 추가한다. 이를 제거하는 옵션.
    * 11번 줄 : 테이블명은 정확히 명시한 것이다. Sequelize.js의 테이블명은 강제로 복수형으로 입력되는데 경우에 따라 가독성이 떨어지므로 필요에 따라 수정한다.
    * 12번 줄 : 후에 추가할 GameUser테이블과의 관계를 정의하는 `associate`를 추가한 것이다. `belongsTo`는 1:1 관계에 대한 외래키가 존재하는 것을 의미한다.