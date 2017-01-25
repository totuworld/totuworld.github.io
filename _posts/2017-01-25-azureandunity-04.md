---
layout: post
title:  "이세계에 진입한 서버 개발 - 4"
date:   2017-01-25 19:00:00
categories: nodejs azure webapps unity
comments: true
meta : nodejs azure webapps webservice
description : 통화 관리
image : 
  twitter: /images/rvwendy_tutorial_04.png
  facebook: /images/rvwendy_tutorial_04.png
publish : true
---

* content
{:toc}

{% include youtubePlayer.html id="Zrem0cNKWiM" %}

## 들어가는 말
통화는 위키피디아에 의하면 아래와 같이 정의된다. 

> 유통 수단이나 지불 수단으로서 기능하는 교환 수단

`돈`을 떠올리면 된다.

![money](http://www.marshu.com/articles/images-website/articles/presidents-on-us-paper-money/one-hundred-100-dollar-bill.jpg)

그럼 게임 안에 3가지 통화가 존재한다고 해보자.

* 하트 : 게임 입장에 사용된다. 일정 시간마다 재충전된다.
* 코인 : 게임 플레이나 업적 등으로 자주 획득할 수 있는 통화.
* 보석 : 주로 인앱 결제로 획득하며 게임 플레이나 업적으로 가아아아끔 획득하는 통화.

이 중 사용자가 가장 소중히 여기는 것은 무엇일까? 당연히 보석이다.

![friendspopcorn](/images/IMG_4984.PNG){:width="500px"}

게임 내에서 희귀한 통화이고 인앱 결제 상품으로 구입이 가능한 형태로 주로 제공되기 때문이다.

> 이거 잘못되면 너와 나, 지옥에서 만나겠지?

이런 통화를 어떻게 관리하는지 알아보도록하자.

> 지급 관리에서 지급과 관계된 내용을 다루므로 이번 내용에서는 빠진다.

## 모델 추가
크게 2가지 모델이 필요하다. 하나는 게임 안에서 통용되는 통화를 정의하는 모델이다. 큰 변화는 없다.

남은 하나는 통화 보유 모델이다. 어떤 것을 얼마나 가지고 있는지 기록해야하기 때문이다.

### 통화 정의, 통화 보유 모델 추가
아래 2개 파일을 `models`폴더에 `DefineCurrency.js`, `OwnCurrency.js`파일을 추가한다.

#### DefineCurrency.js
아래 내용을 입력한다.

```javascript
'use strict';

module.exports = function(sequelize, DataTypes) {
    let DefineCurrency= sequelize.define('DefineCurrency', {
        CurrencyID : { type : DataTypes.INTEGER, primaryKey: true}, //고유한 번호를 할당해야한다.
        Name:{type:DataTypes.STRING(10)},
        MaxQNTY:{type:DataTypes.INTEGER, defaultValue:9000}
    }, {
        timestamps: false,
        tableName: 'DefineCurrency'
    });
    return DefineCurrency;
};
```

DefineCurrency 모델은 `CurrencyID`로 구분된다. 

즉, `CurrencyID`절대로 중복된 숫자를 입력하면 안된다. MaxQNTY는 OwnCurrency의 CurrentQNTY가 절대 넘을 수 없는 최대치를 뜻한다.

### OwnCurrency.js

아래 링크의 내용을 입력한다.

* [OwnCurrency.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto4.2/models/OwnCurrency.js)

OwnCurrency 모델은 GameUser의 `GameUserID`와 DefineCurrency의 `CurrencyID`를 외래키로 가진다.

또한 보유 수량(CurrentQNTY)을 기록할 수 있으며 특정 조건에 의해 최대 보유 수량을 늘릴 수 있는 AddMaxQNTY 컬럼도 있다. 실제 컬럼으로 존재하지 않지만 getterMethods로 `TotalQNTY`를 등록해서 실제 최대보유수량을 계산한다. 

## 라우터 추가

통화에 사용된 라우터를 추가하겠다.

1. `routes` 폴더에 `currency.js`파일을 생성한다.

2. 아래 코드를 입력한다.

   ```javascript
   'use strict'

   const debug = require('debug')('Wendy:router:currency');
   const auth = require('../utils/auth');
   const commonFunc = require('../utils/commonFunc');
   const models = require("../models");
   const wendyError = require('../utils/error');

   const express = require('express');
   const router = express.Router();

   module.exports = router;
   ```

앞으로 라우터를 추가하는 것은 자주 할 것이므로 금방 익숙해질 것이다.

## 로직 추가
2가지 로직을 추가할 것이다.

* 정의된 통화 목록 요청 : 게임 내에서 어떤 통화를 가질 수 있는지 리스트 형태로 리턴한다.
* 보유한 통화 목록 요청 : 사용자가 실제로 보유한 통화를 리스트 형태로 리턴한다.

2가지로 로직은 거의 차이가 없을 것이다.

### 정의된 통화 목록 요청
아래 코드를 `routes/currency.js`의 `module.exports` 위쪽에 추가한다.

```javascript
{% github_sample totuworld/Wendy/blob/tuto4.1/routes/currency.js 12 23%}
```

* 7번 줄 : `findAll` 메서드로 `DefineCurrency` 테이블에 등록된 모든 내용을 검색한다. 여기서처럼 조건을 넣지 않으면 모든 것이 리턴된다.
* 9번 줄 : 찾아진 모든 내용을 `list`노드에 담아서 리턴한다. 이때 `list`노드는 별뜻이 없다(fxxk이라고 해도 된다). 클라이언트가 헷갈리지 않고 구분할 수 있으면 된다.

### 보유한 통화 목록 요청
앞서 제작한 `정의된 통화 목록 요청`과 다른 것은 조회해야할 테이블 다르다는 것과 findAll메서드에 조건을 추가한 정도다.

아래 코드를 `routes/currency.js`의 `module.exports` 위쪽에 추가한다.

```javascript
{% github_sample totuworld/Wendy/blob/tuto4.1/routes/currency.js 24 35%}
```

* 7번 줄 : findAll메서드에 사용될 오브젝트를 만들고 where 노드에 조건을 추가했다. 조건은 GameUserID로 특정 사용자를 지칭했다.

모든 것이 이렇게 간단하면 좋으련만.

### 라우터 등록
`app.js` 파일을 수정하여 currency 라우터를 추가해보자.

1. `app.js`에서 아래 내용을 찾아서 그 아래쪽에 코드를 추가한다.

    * 찾아야하는 내용

            const routes = require('./routes/index');

    * 추가할 코드

            const currency = require('./routes/currency');

2. `app.js`에서 아래 내용을 찾아서 그 아래쪽에 코드를 추가한다.

    * 찾아야하는 내용

            app.use('/', routes);

    * 추가할 코드

            app.use('/currency', currency);

## 일정 시간마다 충전되는 통화는?

`애니x`이나 `드래곤플x이트`가 공전의 히트를 칠 때 하트 구하려고 대포친구(?)를 구했던 기억이 있다.

그 이후 많은 게임들이 재충전 시간을 상품으로 팔았다.

> 돈놓고 시간 사기.

아마도 소규모 게임 개발팀이 만드는 게임에도 이런 통화 하나쯤은 있을 수 있다.

이런 통화 숫자가 많은 편이 아니라 신나게 config 파일 하나 만들고 설정을 넣은 뒤에 계산하곤 했다.

그런데 요즘 게임 어떤가?

특정 레이드 진입 하기위한 `열쇠`도 있고 요일 던전용 하루 입장 제한같은 것도 있다.

어떤 `열쇠`는 기준시각(예를들어 새벽 4시)이 지나면 재충전 시간 계산없이 보유할 수 있는 최대 수량으로 채워지는 형태도 있을 수 있다.

### 기능 정의

얘기된 몇가지 특징을 뽑아보면 대략 아래와 같다.

* 재충전 주기마다 일정량이 재충전된다.
* 시간에 의한 재충전으로 최대치를 넘진 못한다.
* 최대치만큼 보유하면 더이상 재충전되지 않는다.
* 어떤 상품은 기준시각을 지나면 최대치를 보유한다.

위 내용을 바탕으로 `DefineRechargeCurrency` 모델에 포함될 내용을 정리하면 아래 표와 같다.

컬럼명 | 하는 일
--- | ---
RechargeCurrencyID | 재충전통화 고유 ID
IntervalTime | 재충전 주기(초단위 숫자)
IntervalChargeAmount | 재충전 주기마다 충전되는량
SetMaxSwitch | 최대치 충전 주기를 가지는지 나타낸다.
SetMaxPattern | 최대치 충전 주기(Cron 표현식)

### DefineRechargeCurrency 모델 추가

1. `models` 폴더에 `DefineRechargeCurrency.js`파일을 생성한다.

2. 아래 소스코드를 열어서 입력한다.

    > 굉장히 긴 정규표현식이 중간에 삽입되어있어 링크형태로 대치한다.

* [DefineRechargeCurrency.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto4.2/models/DefineRechargeCurrency.js) 

이전에 추가했던 모델과 다른점은 컬럼에 `validate`가 추가된 것이다.

`SetMaxPattern`이 Cron 표현식을 사용하므로 잘못된 입력을 막기 위해 검증코드를 넣은 것이다.

### models/DefineCurrency.js 수정
통화 중에서 재충전이 필요한 경우 `DefineRechargeCurrency` 모델을 참조해야하므로 `RechargeCurrencyID`를 외래키로 등록한다.

models/`DefineCurrency.js` 파일에서 아래 내용을 찾는다.

* 찾아야하는 내용

```javascript
        tableName: 'DefineCurrency'
```

> 위 내용 뒤에 반드시 쉼표(,)를 더해야 에러나지 않는다.

아래 코드를 추가한다.

* 추가할 코드

```javascript
        classMethods: {
            associate: function (models) {
                //재충전되는 통화인 경우 해당 값을 가진다.
                DefineCurrency.belongsTo(models.DefineRechargeCurrency, {
                    onDelete: "CASCADE",
                    foreignKey: {
                        name:'RechargeCurrencyID',
                        allowNull: true
                    },
                    as: 'RechargeInfo'
                });
            }
        }
```

외래키를 등록할 때 `as` 노드는 별명(alias)으로 `DefineRechargeCurrency` 이름이 너무 길어서 짧게 줄이려고 추가한 것이다.

### 재충전 로직 작성
vs code에서 `logics` 폴더를 새로 생성하고 `currency.js` 파일을 추가한다.

1. vs code에서 `logics` 폴더를 추가한다.

2. logics 폴더에 `currency.js`파일을 생성한다.

3. 아래 소스코드를 열어서 입력한다.

    * [currency.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto4.2/logics/currency.js)

* 21~23번 줄 : 메서드를 실행할 필요가 있는지 체크한다.
* 31~41번 줄 : 최대치 충전 주기에 해당하는지 체크하여 해당한다면 최대치로 설정하도록 한다.
* 43~65번 줄 : 재충전 주기를 기준으로 시간경과를 고려하여 충전량을 계산한다.

#### later 모듈 추가
logics/currency.js에서 later 모듈로 cron 표현식을 해석한다.

모듈을 추가해보자.

1. 터미널을 통해 지난 시간에 다운받은 소스코드가 있는 폴더로 이동한다.

    > windows는 파일 탐색기로 해당 폴더를 선택한 뒤 `shift + 마우스 우클릭`하여 `여기서 명령창 열기`를 실행하면 편리하다.

2. 아래 명령을 입력하여 모듈을 설치한다.

        $ yarn add later

#### 환경 설정
Azure 웹앱은 기본 타임존이 UTC(GMT+0)로 설정되어있다. 하지만 우리의 로컬환경은 다르기때문에 이를 맞춰야 시간과 관련된 계산이 편리하다.

vs code의 환경 설정을 수정하여 이를 동일하게 맞추도록 한다.

1. vs code에서 `.vscode`폴더 안에 `launch.json`파일을 선택한다.

2. `env`노드 안에 다음 내용을 추가한다.

        "TZ":"UTC"

### routes/currency.js 수정
2개의 api를 모두 수정할 것이다.

#### currency/define api 수정
정의된 통화목록을 로딩할 때 재충전 관련 데이터도 함께 엮어서 내려질 수 있도록 변경한다.

아래 내용을 찾아서 변경한다.

* 찾아야하는 내용

        models.DefineCurrency.findAll()

* 변경할 코드

        models.DefineCurrency.findAll({
            include: [{
                model: models.DefineRechargeCurrency,
                as: 'RechargeInfo'
            }]
        })

findAll()할 때 include로 DefineRechargeCurrency 모델의 데이터를 포함하여 내려지게 된다.


#### currency/own api 수정
앞서 만든 logics/currency.js를 routes/currency.js에서 활용하도록 해보자.

아래 코드를 찾아서 코드를 추가한다.

* 찾아야하는 내용 

        const wendyError = require('../utils/error');

* 추가할 코드

        const currencyLogic = require('../logics/currency');

그리고 route.get('/own' ... ) 부분 전체를 아래 코드로 변경한다.

```javascript
{% github_sample totuworld/Wendy/blob/tuto4.2/routes/currency.js 30 119%}
```

* 12~23번 줄 : OwnCurrency를 검색할 때 정의된 통화에 관한(재충전 포함) 데이터를 함께 로딩한다.
* 24~32번 줄 : 재충전 주기를 확인해야하는지 체크한다. 만약 어떤 통화도 재충전이 필요없다면 68번 줄로 이동한다.
* 44~67번 줄 : 재충전 주기나 최대치 충전 주기를 확인해서 업데이트 사항이 발생하는지 계산한다. 만약 충전량이 존재하면 이를 반영한다.
* 73~82번 줄 : OwnCurrency를 다시 검색하여 업데이트 사항을 확인한다.
* 83~85번 줄 : 결과 전송.

위 코드에서 특이한 곳은 57~61번 줄이다. OwnCurrency 모델에 업데이트를 요청하고 이를 모두 promises 배열에 할당한다. 그리고나서 65번 줄에서 Promise.all 메서드로 이를 처리한다.

Promise.all 메서드는 지금처럼 반복되는 일을 병렬로 처리해도 상관없을 때 사용하면 된다.

> 단 성능에 큰 문제를 야기할 수 있다면 순차적으로 처리하는 것을 권한다.

## 테스트
필요한 데이터를 넣고 테스트를 진행할 것이다.

하지만 아직 Wendy가 데이터를 넣는 어떤 로직도 제공하지 않기때문에 DB에 직접 데이터를 넣어야한다.

### DBeaver 설치
DB에 접속해서 필요한 일을 처리해줄 GUI 툴로 `DBeaver`를 사용하겠다.

> 무료이고 MSSQL을 지원하면서 Mac, Windows를 모두 사용가능하다.

다음 링크로 이동해서 [DBeaver](http://dbeaver.jkiss.org/)를 설치한다.

### Connection 추가
DB에 접속하기 위해 Connection을 등록해보자.

> SQL 데이터베이스의 서버 방화벽 설정으로 등록된 주소지에서 접속 가능하다.

1. DBeaver를 실행한 뒤 `New Connection`을 클릭한다.

    ![NewConnection](/images/rvwendy03_03.png)

2. `MS SQL Server - jTDS driver`를 선택하고 Next를 클릭한다.

    ![jTDS_driver](/images/rvwendy03_04.png)

3. `Host`에 Azure SQL 데이터베이스 호스트를 입력한다. `User name`과 `Password`도 입력한다. Next 선택하고 모두 입력되면 Finish 클릭한다.

    ![host](/images/rvwendy03_05.png)

### 재충전 통화 정보 추가

> Connection 전에 로컬 서버를 실행한다.

1. 등록된 Connection에 접속을 한 뒤 자신의 db를 연다(여기서는 wendydb).

2. `dbo - tables - DefineRechargeCurrency`를 더블 클릭 한다.

    ![selecttable](/images/rvwendy03_06.png)

3. `Data` 탭을 누른다.

    ![Data_tab](/images/rvwendy03_07.png)

4. `+` 버튼(Add new row)을 클릭 한 뒤 아래 표처럼 입력한다.

    ![plusbutton](/images/rvwendy03_08_0.png)

    RechargeCurrencyID | IntervalTime | IntervalChargeAmount | SetMaxSwitch | SetMaxPattern
    --- | --- | --- | --- | ---
    101 | 3600 | 2 | 1 | 15 10 * * ? *

    101은 3600초(1시간)에 2만큼 재충전된다. 그리고 매일 10:15분에 최대치로 충전된다.

    ![newrow](/images/rvwendy03_08.png)

5. 아래쪽에 `Save`버튼을 클릭해서 추가한다.

    ![savebutton](/images/rvwendy03_09.png)

### 정의된 통화 추가

위의 과정처럼 `DefineCurrency` 테이블에 데이터를 입력한다.

CurrencyID | Name | MaxQNTY | RechargeCurrencyID
--- | --- | --- | ---
101 | key | 500 | 101
102 | gem | 900000 | NULL
103 | gold | 900000 | NULL

위 테이블처러 2가지 상품을 등록한다. 그 중 101 상품은 재충전 가능한 상품이 된다.

### 보유한 통화 추가
지난 강좌에서 사용자를 등록했다는 가정하에 진행한다.

> 만약 사용자 등록을 하지 않았다면 등록한 뒤 GameUserID란을 자신의 것과 동일하게 넣으면 된다.

OwnCurrencyUID | CurrencyID | CurrentQNTY | NowMaxQNTY | AddMaxQNTY | UpdateTimeStamp | GameUserID
--- | --- | --- | --- | --- | ---
NULL | 101 | 5 | 5 | 0 | 2016-12-30 | 1
NULL | 102 | 10 | 900 | 0 | 2016-12-30 | 1
NULL | 103 | 1000 | 90000 | 0 | 2016-12-30 | 1

`OwnCurrencyUID`는 자동으로 증가되므로 NULL을 넣어도 알아서 값을 가지게 된다.

### postman으로 테스트

* 패스 : GET
* URL : localhost:3000/currency/own
* Headers : Authorization을 추가하고 토큰 내용을 value 부분에 넣는다.

성공하면 다음과 같은 정보를 돌려준다.

```
{
  "result": 0,
  "list": [
    {
      "TotalQNTY": 5,
      "OwnCurrencyUID": 7,
      "CurrencyID": 101,
      "CurrentQNTY": 5,
      "NowMaxQNTY": 5,
      "AddMaxQNTY": 0,
      "UpdateTimeStamp": "2017-01-04T05:46:19.658Z",
      "GameUserID": 1
    },
    {
      "TotalQNTY": 100,
      "OwnCurrencyUID": 8,
      "CurrencyID": 102,
      "CurrentQNTY": 1,
      "NowMaxQNTY": 100,
      "AddMaxQNTY": 0,
      "UpdateTimeStamp": "2016-12-30T00:00:00.000Z",
      "GameUserID": 1
    }
  ]
}
```

보는 바와 같이 101 상품은 재충전이 일어나서 CurrentQNTY가 5가 되었고 UpdateTimeStamp도 변경이 있다.

## 맺음말
이번에 다룬 것은 통화였다. 그중에서 재충전에 관한 것이 많은 부분을 차지했다.

실제로 통화 관리는 이보다 더 복잡하게 이뤄질 수 있다.

이 과정을 넘어서 다음 강좌에서 만나자!!

---

## 참고자료
완성된 소스코드는 아래 링크에서 다운로드받으면 된다.

[Wendy 4강 완료 버전](https://github.com/totuworld/Wendy/archive/0.0.3.zip)