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

### 사용자 모델 정의

1. models폴더에 `GameUser.js`파일을 만든다.

2. 아래 소스코드를 입력한다.

    {% gist /72ffa52a423d192023514081880b593a GameUser.js %}

    외래키 설정이 없어서 더 간단하다.

## 라우터 작성
express의 라우터는 패스와 콜백 메서드로 이뤄진다.

    router.METHOD( /* 패스(path) */ , /* 콜백 메서드(callback method) */ );  

앞의 METHOD는 HTTP 메서드(get, post, put 등)를 제공한다. 패스는 URL의 패스를 입력하면 된다. 콜백 메서드가 실제로 모든 일을 처리하게 된다.

### 기기 라우터 추가
1. `routes` 폴더에 `device.js`파일을 생성한다.

2. 아래 코드를 입력한다.

   ```javascript
   'use strict'

   const debug = require('debug')('Wendy:router:device');
   const auth = require('../utils/auth');
   const commonFunc = require('../utils/commonFunc');
   const models = require("../models");
   const wendyError = require('../utils/error');

   const express = require('express');
   const router = express.Router();

   module.exports = router;
   ```

   * 3~7 번 줄 : Node.js 에서 require('경로'); 를 활용하면 그 .js 파일 기준으로 상대 경로에 위치한 js 파일을 가져와 쓸 수 있다.
   * 9~10 번 줄 : express 모듈에서 router를 추가하는 방법이라고 알고 있도록 하자.
   * 12번 줄 : `module.exports`는 사용자 모듈을 만들 때 사용한다.

   이 코드는 router를 로딩하여 편집하고 다시 모듈로 내보내는 형태이다.

### 사용자 라우터 추가
1. `routes` 폴더에 `user.js`파일을 생성한다.

2. 아래 코드를 입력한다.

   ```javascript
   'use strict'
   
   /// <reference path="../typings/node/node.d.ts" />
   /// <reference path="../typings/express/express.d.ts" />
   
   const debug = require('debug')('Wendy:router:user');
   const auth = require('../utils/auth');
   const commonFunc = require('../utils/commonFunc');
   const models = require("../models");
   const wendyError = require('../utils/error');
   
   const express = require('express');
   const router = express.Router();
   
   module.exports = router;
   ```

    routes/device.js와 다른 것은 reference를 추가한 것 뿐이다.

### 라우터 등록
이렇게 라우터를 추가해도 express.js가 바로 인식할 수 있지 않다. `app.js` 파일을 수정하여 2가지 라우터를 추가해보자.

1. `app.js`에서 아래 내용을 찾아서 그 아래쪽에 코드를 추가한다.

    * 찾아야하는 내용

       const routes = require('./routes/index');

    * 추가할 코드

   ```javascript
   {% github_sample totuworld/Wendy/blob/0.0.2/app.js 9 10%}
   ```

2. `app.js`에서 아래 내용을 찾아서 그 아래쪽에 코드를 추가한다.

    * 찾아야하는 내용

       app.use('/', routes);

    * 추가할 코드

   ```javascript
   {% github_sample totuworld/Wendy/blob/0.0.2/app.js 26 27%}
   ```

위 처럼 변경이 완료되면 특정 패스로 시작되는 것을 해당 라우터가 모두 처리하게 된다.

예를 들어 `hostname/device` 로 시작되는 모든 패스는 `routes/device.js`가 관리하게 되는 것이다.

## 로직 등록
기기의 UUID를 통해서 게임 입장 시 이뤄지는 기본 동작은 다음과 같은 순서로 진행된다.

![사용자등록](/images/mks_v03.png)

게임 서버가 처리해야할 일은 총 4가지다.

* 토큰 요청(기기등록 여부 확인)
* 기기 등록
* 사용자 등록(닉네임 등록)
* 사용자 정보 조회

4가지 로직을 프로그래밍해보자.

### 기기 등록 로직
기기에 필요한 로직은 크게 2가지이다. 등록된 UUID를 통해서 토큰을 얻는 것과 기기 등록 요청을 하는 것이다.

아래 코드를 `routes/device.js`의 `module.exports` 위쪽에 추가한다.

```javascript
{% github_sample totuworld/Wendy/blob/0.0.2/routes/device.js 57 92%}
```

소스코드를 읽기위해서 `Promise 패턴`을 눈에 익힐 필요가 있는데 이건 시간이 해결해줄거라고 생각하겠다.

> Promise 패턴에 대한 상세한 설명은 [Javascript Promise](https://developers.google.com/web/fundamentals/getting-started/primers/promises)를 참고하면 된다.

> 한글로된 설명을 원한다면 [바보들을 위한 Promise 강의](http://programmingsummaries.tistory.com/325)를 참고하면 된다.

* 9~13번 줄 : 서버 도착한 요청에 body부분에 필요한 정보(여기서는 UUID, DeviceType)이 존재하는지 체크하여 없으면 오류 처리한다.
* 16번 줄 : UUID로 등록된 기기가 있는지 기기테이블(GameDevice)를 검색한다.
* 17~31번 줄 : 기기가 없으면 등록하고 존재하면 등록절차만 건너뛰고 모두 정상 결과를 송신한다.

> 우리가 클라이언트에 보내는 응답은 { result : 결과 코드 } 형태로 보낼 것이다.

#### ORM 활용 코드 읽기
잠깐 설명하고 넘어가자면 앞서 본 코드의 16번 줄은 ORM을 이용해서 데이터베이스를 조회한 것이다.

대략 다음과 같은 형태의 코드가 모두 ORM을 이용하는 것이다.

```
models.[테이블명].[명령어]
```

테이블명은 models에 추가한 모델의 명칭을 뜻하므로 추가하지 않은 것을 넣으면 문제가된다.

자주 사용되는 명령어는 다음 4개이고 각 명령마다 넣는 값이 조금씩 다르다.

* findOne : 조건에 맞는 한 개의 데이터를 찾는다. where로 조건을 나타낸다.
* findAll : 조건에 맞는 모든 데이터를 찾는다. where로 조건을 나타낸다.
* create : 새로운 행(데이터)을 추가한다. 별도의 조건이 없다.
* update : 조건에 맞는 것을 설정한 값으로 모두 수정한다. where로 조건을 나타낸다.

위에 나온 실제 코드를 읽어보자.

    models.GameDevice.findOne({where:{UUID:req.body.UUID}})

> `GameDevice` 모델에서 `UUID` 열이 req.body.UUID와 같은 1개의 행을 찾아라

자주 보게되면 금방 익숙해지니 외우려고하지 말고 넘어가자.

### 토큰 요청 로직
주민등록번호는 편리하다. 번호만으로 그 사람이 누구인지 확인해니 관공서 업무에 많이 활용된다.

거꾸로 말하면 유출 시 손쉽게 다른 사람이 될 수 있다.

> 누군가 내 주민등록번호로 네이버 아이디를 2개나 만들어서 쓰고있었다.

그래서 우리는 사용자 정보를 직접 공개하지 않고 일정 기간만 유효한 토큰(token)을 발급하고 이를 통해 기능을 활용할 것이다.

> 우리가 사용할 JWT(Json Web Token)은 이런 목적으로 만들어진 것은 아니다.

아래 코드를 [기기 등록 로직](#section-10)에서 처럼 `routes/device.js`의 `module.exports` 위쪽에 추가한다.

```javascript
{% github_sample totuworld/Wendy/blob/0.0.2/routes/device.js 12 56%}
```

위 코드는 3가지 예외상황을 확인하고 정상적인 경우에 토큰을 생성하여 전달한다.

* 11~14번 줄 : 등록된 기기인지 확인한다.
* 16~19번 줄 : 기기가 등록되었지만 GameUserID를 부여받았는지 확인한다. 닉네임 등을 요구해서 유저로 등록해야하는 것이다.
* 21~24번 줄 : 주 사용기기인지 체크한다. 이는 1명의 사용자가 다수의 기기를 돌릴 경우를 대비한 것으로 지금은 무의미하다.

### 사용자 정보 조회 로직
사용자 관련 로직도 복잡도가 낮은 정보 조회부터 제작해보자.

아래 코드를 `routes/user.js`의 `module.exports` 위쪽에 추가한다.

```javascript
{% github_sample totuworld/Wendy/blob/0.0.2/routes/user.js 135 152%}
```

* 9번 줄 : 로그인 시간을 최신시각으로 변경한다.
* 11번 줄 : GameUserID를 검색한다.

이 소스코드가 앞선 코드들과 다른 것은 사실 6번 줄이다.

자세히 살펴보면 패스와 콜백 메서드 사이에 미들웨어가 끼어있다. 이 미들웨어는 요청 시 Header의`Authorization`에 포함되는 JWT가 사용가능한 것인지 확인한다.

> 앞으로 이런 코드를 자주 보게 될 것이다.

### 사용자 등록 로직
예외처리가 이곳저곳에 있어서 복잡해보이지만 로직을 묶어서보면 간단한 편이다.

아래 코드를 `routes/user.js`의 `module.exports` 위쪽에 추가한다.

```javascript
{% github_sample totuworld/Wendy/blob/0.0.2/routes/user.js 14 134%}
```

* 12~19번 줄 : 서버 도착한 요청에 body부분에 필요한 정보와 닉네임 길이를 확인한다.
* 46~74번 줄 : UUID를 통해 기기 등록 여부를 확인한다.
* 76~85번 줄 : 닉네임 사용여부를 확인한다.
* 87~111번 줄 : GameUser 정보를 등록하고 GameDevice와 연결한다. 그리고 결과를 전송한다.

## 테스트
작성한 코드가 올바로 작동하는지 확인해보자.

API를 테스트하려면 `cURL`이나 `HTTPie`같은 CLI용 클라이언트를 사용해도 되지만 GUI로 가능한 `Postman`을 사용하겠다.

### app.js 수정
우리가 만드는 게임 서버는 에러가 발생하면 HTML 형식으로 리턴한다.

왜냐하면 express.js의 기본이 그렇게 설정되어있기때문이다.

이것을 JSON 형식으로 리턴하도록 수정한다.

1. `app.js`에서 다음 내용을 찾는다.

       res.render('error', {

    2 곳 있을 것이다.

2. 아래처럼 `render`를 `send`로 변경하고 'error' 부분을 제거한다.

       res.send({

### Postman 설치

1. [www.getpostman.com](https://www.getpostman.com)으로 접속해서 클라이언트를 다운받는다.

2. 설치한다.

### 테스트 코드
Postman을 실행하여 `토큰 요청`, `기기 등록`, `사용자 등록`, `사용자 정보 조회` 순으로 실행해보겠다.

우선 테스트를 위해서 vs code에서 `F5`를 누르거나 `Debug` 메뉴로 이동해서 `Start Debugging`버튼을 클릭하여 게임 서버를 동작하도록 한다.

![debug](/images/rvwendy03_01.png)

#### 토큰 요청
Postman의 `Request Editor`를 다음과 같이 입력한다.

![Request Editor](https://www.getpostman.com/img/v1/docs/thumbs/2.png)

* 패스 : GET
* URL : localhost:3000/device/token/testdeviceuuid

입력이 끝나고 `Send`버튼을 누르면 다음과 같은 에러를 리턴할 것이다.

```
{
  "result": 80101,
  "message": "등록하지 않은 기기",
  "error": {
    "name": "UnregisteredDevice",
    "code": 80101
  }
}
```

정상이다. 기기등록을 해보자.

#### 기기 등록

* 패스 : POST
* URL : localhost:3000/device
* body : 아래 내용을 넣는다.

```
{
    "UUID":"testdeviceuuid",
    "DeviceType":10
}
```

![postmanbody](/images/rvwendy03_02.png)

결과로 0번 코드를 리턴하면 정상이다.

```
{
    "result": 0
}
```

#### 사용자 등록
닉네임을 입력하여 사용자 등록을 진행한다.

* 패스 : POST
* URL : localhost:3000/user
* body : 아래 내용을 넣는다.

```
{
    "NickName":"cat냥이",
    "Locale":"ko-kr",
    "UUID":"testdeviceuuid",
    "OffsetTime":9
}
```

정상 등록이 되면 아래 같은 결과가 나온다.

> 단 token은 다를 수 있다.

```
{
    "result": 0,
    "token": "blah.blah.blah"
}
```

#### 사용자 정보 조회
위에서 받은 토큰을 활용해서 사용자 정보 조회를 해보자.

* 패스 : GET
* URL : localhost:3000/user
* Headers : Authorization을 추가하고 토큰 내용을 value 부분에 넣는다.

성공하면 다음과 같은 정보를 돌려준다.

```
{
  "result": 0,
  "UserInfo": {
    "GameUserID": 2,
    "NickName": "cat냥이",
    "Locale": "ko-kr",
    "OffsetTime": 9,
    "OffsetTimeUpdateAt": "2016-12-27T02:57:08.000Z",
    "createAt": "2016-12-27T02:57:08.000Z",
    "loginAt": "2016-12-27T05:26:25.107Z"
  }
}
```

## 맺는말
처음 프로그래밍을 진행하는 것이고 익숙치않은 javascript와 Promise 패턴 덕분에 뭐가 뭔지 하나도 모르겠다고 느낄 수 있다.

처음 Unity로 게임을 시작할 때도 비슷했을 것이라고 본다. 자주 접하다보면 점점 익숙해진다.

다음 시간에는 재화를 관리해보도록 하겠다.

---

## 참고자료
완성된 소스코드는 아래 링크에서 다운로드받으면 된다.

[Wendy 3강 완료 버전](https://github.com/totuworld/Wendy/archive/0.0.2.zip)