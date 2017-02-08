---
layout: post
title:  "이세계에 진입한 서버 개발 - 6"
date:   2017-01-23 00:01:00
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

지난 시간에 `logics/materialCtrl.js`를 추가한 것이 기억나는가? 자원을 차감하는 메서드가 있었다.

여기에 자원을 추가하는 메서드를 추가한다.

그리고 `logics/reward.js` 파일을 추가하여 RewardSetGroup과 RewardGoodsGroup을 바탕으로 지급하도록 한다.

### 자원 추가 메서드 incrementMaterials

다른 메서드와 겹치지 않게 `logics/materialCtrl.js` 제일 아래쪽에 추가한다.

```javascript
{% github_sample totuworld/Wendy/blob/tuto6.1/logics/materialCtrl.js 140 267%}
```

### logics/reward.js 추가
logics 폴더에 `reward.js` 파일을 생성하고 아래 내용을 적용한다.

* [logics/reward.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto6.1/logics/reward.js)

* 에러코드 추가
* utils/commonFunc.js 메서드 추가