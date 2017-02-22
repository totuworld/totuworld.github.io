---
layout: post
title:  "이세계에 진입한 서버 개발 - 6"
date:   2017-02-20 19:01:00
categories: nodejs azure webapps unity
comments: true
meta : nodejs azure webapps webservice
description : 자원 지급 관리
image : 
  twitter: /images/rvwendy_tutorial_06.png
  facebook: /images/rvwendy_tutorial_06.png
publish : true
---

* content
{:toc}

{% include youtubePlayer.html id="ZSloWdCq4LY" %}

## 들어가는 말

지난 시간에 아이템 관리까지 다루면서 게임 내 자원을 지급할 수 있는 기반이 마련되었다.

이번 시간에는 자원을 지급하도록 해보겠다.


## 모델 추가
모델은 총 4개가 필요하다.

`models` 폴더에 파일을 추가하고 각각 아래 내용을 적용한다.

* [RewardSetGroup.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto6.1/models/RewardSetGroup.js)
* [RewardSet.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto6.1/models/RewardSet.js)
* [RewardGoodsGroup.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto6.1/models/RewardGoodsGroup.js)
* [RewardGoods.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto6.1/models/RewardGoods.js)

위 모델 중 가장 기본이 되는 것은 `RewardGoodsGroup`과 `RewardGoods`이다.

`RewardGoodsGroup`에서 관리되는 `RewardGoodsGroupID`를 통해 다수의 `RewardGoods` 목록을 그룹으로 묶는다.

![RewardGoodsGroup과RewardGoods의 관계](/images/rvwendy05_03.png){:width="250px"}

이렇게 묶어두는 이유는 하나의 상품으로 다수의 자원이 지급될 수 있도록 하기 위함이다.

> 행운 상자라고 지칭한 뒤 최소 확율로 요원이 나오고 높은 확율로 골드 정도를 묶은 상품을 생각하면 된다.

`RewardSetGroup`은 `RewardSetGroupID`를 통해 `RewardSet`에 다수의 `RewardGoodsGroupID`를 등록할 수 있는 형태이다.

이렇게되면 묶음(패키지) 상품을 제공할 수 있다.

> 골드와 보석, 키 등이 한번에 포함된 상품을 제공하는 스타터 팩을 생각하면 된다.

![RewardSetGroup과RewardSet의 관계](/images/rvwendy05_04.png){:width="500px"}

단순한 묶음 상품이상으로 작동하도록 `RewardSet`에도 드랍 확율을 넣었다. 묶음 상품 안에서 운이 좋으면 추가 재화를 얻을 수 있도록 말이다.

## 라우터 추가

자원 세트(묶음 상품)와 지급 가능한 자원 정보를 추가하겠다.

* GET /reward/set
* GET /reward/goods

`routes`폴더에 `reward.js`파일을 추가하고 아래 내용을 적용한다.

* [routes/reward.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto6.1/routes/reward.js)

### app.js에 등록
`app.js` 파일을 수정하여 reward 라우터를 추가해보자.

1. `app.js`에서 아래 내용을 찾아서 그 아래쪽에 코드를 추가한다.

    * 찾아야하는 내용

            const routes = require('./routes/index');

    * 추가할 코드

            const reward = require('./routes/reward');

2. `app.js`에서 아래 내용을 찾아서 그 아래쪽에 코드를 추가한다.

    * 찾아야하는 내용

            app.use('/', routes);

    * 추가할 코드

            app.use('/reward', reward);

## 공통 사용 로직 추가
지금까지 모델을 추가했으며 간단한 2개의 API를 등록했다.

이제 지급과 관련된 작업을 할 때 필요한 메서드를 추가하도록 하겠다.

### 자원 추가 메서드 incrementMaterials

지난 시간에 `logics/materialCtrl.js`를 추가한 것이 기억나는가? 자원을 차감하는 메서드가 있었다.

여기에 자원을 추가하는 메서드를 추가한다.

다른 메서드와 겹치지 않게 `logics/materialCtrl.js` 제일 아래쪽에 추가한다.

```javascript
{% github_sample totuworld/Wendy/blob/tuto6.1/logics/materialCtrl.js 140 281%}
```

### logics/reward.js 추가

RewardSetGroup과 RewardGoodsGroup을 바탕으로 `logics/materialCtrl.js`의 자원 추가 메서드로 자원을 지급하는 로직을 제작하겠다.

`logics/reward.js` 파일을 추가한 뒤 아래 내용을 적용한다.

* [logics/reward.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto6.3/logics/reward.js)

위 코드가 문제 없이 작동하려면 2가지를 더 추가해야한다.

먼저 `utils/commonFunc.js`에 아래 2가지 메서드를 추가한다.

```javascript
{% github_sample totuworld/Wendy/blob/tuto6.1/utils/commonFunc.js 18 44%}
```

그리고 에러코드를 추가한다. `utils/error.js`에 아래 클래스를 추가한다.

```javascript
{% github_sample totuworld/Wendy/blob/tuto6.1/utils/error.js 169 204%}
```

`utils/error.js`의 `errorMap` 오브젝트에 추가할 클래스를 등록한다.

> 추가할 때 이전의 마지막 부분 - 여기서는 NoLongerUpgrade - 에 콤마(,)를 추가하여야 한다.

```javascript
{% github_sample totuworld/Wendy/blob/tuto6.1/utils/error.js 228 232%}
```

## 간단 테스트
지급 가능하도록 테이블에 데이터를 입력하고 실제 지급이 이뤄지는 테스트해보자.

### 간단 테스트용 라우터 추가

> 이 내용은 앞서 routes/reward.js 를 추가할 때 이미 들어가있으므로 추가 하지 않는다.

`routes/reward.js` 하단에 아래 코드를 추가한다.

```javascript
{% github_sample /totuworld/Wendy/tuto6.2/routes/reward.js 46 62%}
```

환경변수 `NODE_ENV`가 `development`일 때만 사용가능도록 조치했다.

### 데이터 입력 

테스트를 진행하기 위해서는 총 4개 테이블에 데이터를 등록해야한다.

> DBeaver를 이용한다.

`RewardGoodsGroup` 테이블에 데이터를 입력한다.

RewardGoodsGroupID | Description
--- | ---
1011 | NULL
1012 | NULL

`RewardGoods` 테이블에 데이터를 입력한다.

RewardGoodsUID | AmountMin | AmountMax | DropRatio | ItemID | RewardGoodsGroupID
--- | --- | --- | --- | --- | ---
NULL | 10 | 1000 | 0.9 | 103 | 1011
NULL | 1 | 10 | 0.15 | 102 | 1011
NULL | 1 | 1 | 0.05 | 2001 | 1011
NULL | 10 | 100 | 0.1 | 102 | 1012

`RewardSetGroup` 테이블에 데이터를 입력한다.

RewardSetGroupID | Description
--- | ---
6001 | NULL
6002 | NULL

`RewardSet` 테이블에 데이터를 입력한다.

RewardSetUID | DropRatio | RewardGoodsGroupID | RewardSetGroupID
--- | --- | --- | --- | --- | ---
NULL | 1 | 1011 | 6001
NULL | 0.5 | 1012 | 6001
NULL | 1 | 1012 | 6002

### 즉시 지급 테스트

* 패스 : GET
* URL : localhost:3000/reward/test/6001

* Headers : Authorization을 추가하고 토큰 내용을 value 부분에 넣는다.

정상적으로 요청되면 아래와 같은 형식의 결과를 확인할 수 있다.

```json
{
  "result": 0,
  "reward": {
    "item": [],
    "currency": [
      {
        "TotalQNTY": 900000,
        "OwnCurrencyUID": 7,
        "CurrencyID": 103,
        "CurrentQNTY": 1217,
        "NowMaxQNTY": 900000,
        "AddMaxQNTY": 0,
        "UpdateTimeStamp": "2017-02-21T00:00:00.000Z",
        "GameUserID": 1
      }
    ]
  }
}
```

---

## 쿠폰 제작

자원을 지급할 수 있으면 제작가능한 기능이 많다.

빨리 만들 수 있는 쿠폰을 제작해보도록 하겠다.

![쿠폰](/images/rvwendy06_01.png)

### 쿠폰 모델 추가

쿠폰은 총 3개의 모델이 필요하다. `models` 폴더에 파일을 추가하고 각각 아래 내용을 적용한다.

* [DefineCoupon.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto6.2/models/DefineCoupon.js)
* [CouponQNTY.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto6.2/models/CouponQNTY.js)
* [LogCoupon.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto6.2/models/LogCoupon.js)

`DefineCoupon`은 쿠폰을 정의하고 `CouponQNTY`는 정의된 쿠폰 중 수량만 별도로 적을 수 있도록 한다.

`DefineCoupon`은 `RewardGoodsGroup`이나 `RewardSetGroup`의 ID를 이용해서 지급한다.

> 이때 주의할 점은 둘 중 하나는 꼭 등록을해야 올바로 지급이 가능하다는 점이다. 관리자 도구 제작 시 참고한다.

![DefineCoupon예시](/images/rvwendy06_02.png){:width="400px"}

`LogCoupon`은 사용자의 쿠폰 사용 이력을 기록하여 중복 사용을 방지한다.

### 쿠폰 라우터 추가

정의된 쿠폰 리스트를 요청할 수 있고 쿠폰을 사용할 수 있다.

* GET /coupon/list
* POST /coupon/use/:CouponName

`routes`폴더에 `coupon.js`파일을 추가하고 아래 내용을 적용한다.

* [routes/coupon.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto6.2/routes/coupon.js)

쿠폰 사용부분이 조금 복잡하게 되어있을 것이다. 중복 사용이 불가능하도록 해야하고 유효한 쿠폰인지 검증하는 부분이 많아서 그렇다.

단순하게 절차를 나타내면 아래처럼 진행된다.

1. 유효한 쿠폰인지 검증
2. 쿠폰 사용 기록
3. 자원 지급

### app.js에 등록
`app.js` 파일을 수정하여 coupon 라우터를 추가해보자.

1. `app.js`에서 아래 내용을 찾아서 그 아래쪽에 코드를 추가한다.

    * 찾아야하는 내용

            const routes = require('./routes/index');

    * 추가할 코드

            const coupon = require('./routes/coupon');

2. `app.js`에서 아래 내용을 찾아서 그 아래쪽에 코드를 추가한다.

    * 찾아야하는 내용

            app.use('/', routes);

    * 추가할 코드

            app.use('/coupon', coupon);

### 테스트용 데이터 추가

`CouponQNTY` 테이블에 데이터를 입력한다.

CouponQNTYID | CurrentQNTY
--- | ---
- | 10

`DefineCoupon` 테이블에 데이터를 입력한다.

CouponID | CouponName | ExpiredDate | CouponQNTYID | RewardSetGroupID | RewardGoodsGroupID 
--- | --- | --- | --- | --- | ---
- | test | 2017-02-24 | 1 | - | 1011

> `CouponQNTYID`는 반드시 앞서 CouponQNTY 테이블에 등록된 CouponQNTYID를 확인하여 올바로 입력한다.

### 쿠폰 테스트

* 패스 : POST
* URL : localhost:3000/coupon/use/test

* Headers : Authorization을 추가하고 토큰 내용을 value 부분에 넣는다.

요청이 올바르면 앞서 살펴본 `즉시 지급 테스트`와 유사한 결과를 얻을 수 있다.

## 맺음말

아직 관리자 도구가 없어서 운영까지는 불가능하지만 어쨌든 Wendy에 쿠폰까지 제작해봤다.

> 쿠폰은 1차 제작 범위 안에 없었는데 만들다보니 들어갔다. 서비스~ 서비스~

다음 강좌는 인앱 영수증 검증부분이다. 영수증을 검증하는 부분을 제외하면 자원 지급을 그대로 사용하므로 비교적 간단하다.

---

## 참고자료
완성된 소스코드는 아래 링크에서 다운로드받으면 된다.

[Wendy 6강 완료 버전](https://github.com/totuworld/Wendy/archive/0.0.5.zip)