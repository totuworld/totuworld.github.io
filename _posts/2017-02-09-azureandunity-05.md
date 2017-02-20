---
layout: post
title:  "이세계에 진입한 서버 개발 - 5"
date:   2017-02-08 19:00:00
categories: nodejs azure webapps unity
comments: true
meta : nodejs azure webapps webservice
description : 아이템 관리
image : 
  twitter: /images/rvwendy_tutorial_05.png
  facebook: /images/rvwendy_tutorial_05.png
publish : true
---

* content
{:toc}

{% include youtubePlayer.html id="KVK4JgeWe6c" %}

## 들어가는 말
퍼즐 게임이 아니라면 게임 내에서 아이템은 그 종류가 정말 다양하다.

> 요즘은 퍼즐 게임도 특수 아이템을 제공하거나 구매할 수 있는 것도 많다.

게임 클라이언트에서는 특수 능력을 발휘하거나 체력을 채워줄 수도 있고, 9강 검이 될 수도 있다.

하지만 게임 서버안에서는 보유와 강화 등의 정보만 기록하면 된다.

> MORPG 게임 서버를 만드는게 아니니까 다른건 모른다(?)

이번 강좌에서는 아이템 보유와 강화, 승급 기능을 제작해도보도록 하겠다.

## 모델 추가
기본이 되는 것은 아래 2개이다. 아이템 정의와 아이템 보유 모델.

`models` 폴더에 파일을 추가하고 각각 아래 내용을 적용한다.

* [DefineItem.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto5.3/models/DefineItem.js)
* [OwnItem.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto5.3/models/OwnItem.js)

그리고 부수적으로 아래 4개의 모델을 추가한다.

> 강화와 승급을 설명할 때 추가하려고 했으나 미리 추가해놓는게 편해서 일단 모두 등록한다.

* [DefineReinforceItem.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto5.3/models/DefineReinforceItem.js)
* [DefineReinforceRequireItem.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto5.3/models/DefineReinforceRequireItem.js)
* [DefineUpgradeItem.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto5.3/models/DefineUpgradeItem.js)
* [DefineUpgradeRequireItem.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto5.3/models/DefineUpgradeRequireItem.js)

각각 클릭해서 models 폴더에 추가한다.

### 관계 설명

> 와 이렇게 뭐 없이 설명하니까 되게 좋네!!

![오예](/images/15940e9cd6ba5ee6.jpg)

너무 설명이 없는 듯하니까 각 모델간의 관계를 설명해보겠다.

![아이템다이어그램](/images/rvwendy05_m01.png){:width="500px"}

추가 모델을 제거하면 기본은 `OwnItem`과 `DefineItem`이다.

이 중 `DefineItem`은 강화와 승급을 위해 `ReinforceItemID`과 `UpgradeItemID`를 가질 수 있다.

각각의 외래키는 `Define__RequireItem`으로 이어지고 여기서는 1개의 `__ItemID`로 다수의 아이템이 필요하다는 정보를 저장하게 된다.

왜 이런 구조를 가지냐면 어떤 경우는 골드만 사용하고 싶을 수 있지만 어떤 경우는 골드와 특정 아이템을 함께 소모해서 다음으로 넘기기 위한 방법이다.

다만 이런 처리로는 경험치를 쌓아서 강화하는 형태는 불가능하다.

이런 구조를 원한다면 별도의 테이블을 구상해야할 것이다.

> 좋은 고민거리죠?!

## 라우터 추가
`routes`폴더에 `item.js`파일을 추가하고 아래 내용을 적용한다.

* [item.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto5.1/routes/item.js)

### app.js에 등록
`app.js` 파일을 수정하여 item 라우터를 추가해보자.

1. `app.js`에서 아래 내용을 찾아서 그 아래쪽에 코드를 추가한다.

    * 찾아야하는 내용

            const routes = require('./routes/index');

    * 추가할 코드

            const item = require('./routes/item');

2. `app.js`에서 아래 내용을 찾아서 그 아래쪽에 코드를 추가한다.

    * 찾아야하는 내용

            app.use('/', routes);

    * 추가할 코드

            app.use('/item', item);

> 몇번 해본 작업이라서 손이 알아서 처리하고 있을것이다.

## 로직 작성
로직은 총 6개를 추가할 것인데 그중 4개는 정보를 요청하는 것이고 2개는 강화, 승급을 각각 처리하는 것이다.

통화를 기억해보면 단순히 정보를 요청하는 코드는 단순하다

routes/item.js에 다음 코드를 추가한다.

> 자주 해봐서 알겠지만 modules.export = router; 위쪽에 추가하면 된다.

### 보유 아이템 목록 요청

```javascript
{% github_sample totuworld/Wendy/blob/tuto5.2/routes/item.js 13 29%}
```

### 아이템 정보 목록 요청

```javascript
{% github_sample totuworld/Wendy/blob/tuto5.2/routes/item.js 30 44%}
```

### 강화 관련 데이터 요청

```javascript
{% github_sample totuworld/Wendy/blob/tuto5.2/routes/item.js 44 61%}
```

### 승급 관련 데이터 요청

```javascript
{% github_sample totuworld/Wendy/blob/tuto5.2/routes/item.js 62 77%}
```

정의된 데이터를 요청하는 것은 이처럼 간단하다.

### 강화 요청
강화와 승급의 로직은 기본은 같다.

`Define__RequireItem`에 등록된 `ItemID`가 `RequireQNTY`만큼 보유했으면 아이템을 소모시키고 해당 레벨이나 등급을 올리면 된다.

#### 공통 사용 로직 추가

자원 - 아이템, 통화를 통합하여 지칭 - 이 충분하지 체크하거나 소모시키는 일은 자주 일어날 수 있다.

이런 내용은 따로 관리하는게 편리하므로 `logics`폴더에 `materialCtrl.js`파일을 추가하고 아래 링크의 내용을 반영한다.

* [materialCtrl.js](https://raw.githubusercontent.com/totuworld/Wendy/tuto5.2/logics/materialCtrl.js)

그리고 `routes/item.js` 파일에서 아래 부분을 찾아서 필요한 내용을 추가한다.

* 찾아야하는 내용

          const wendyError = require('../utils/error');

* 추가할 내용

          const materialCtrl = require('../logics/materialCtrl');

#### 강화 요청 등록

앞서 말한 로직을 바탕으로 내용을 추가해보자.

먼저 사용할 메서드 3개를 등록한다.

* LoadOwnItem
* LoadDefineItem
* DecoretorMaterials

2개 메서드는 보유 아이템과 아이템 정의 관련 데이터를 로딩할 때 사용된다.

`DecoretorMaterials`은 소모할 아이템 및 통화를 검증하거나 소모할 때 상용된다. fn 매개변수로 메서드를 전달하면 실행한다.

```javascript
{% github_sample totuworld/Wendy/blob/tuto5.2/routes/item.js 80 141%}
```

위 메서드를 활용해서 실제 로직을 추가한다.

```javascript
{% github_sample totuworld/Wendy/blob/tuto5.4/routes/item.js 142 260%}
```

* 24~57번 줄 : 보유 아이템과 아이템 정보를 로딩한다.
* 58~81번 줄 : 강화를 진행하기에 적합한지 데이터를 검증한다.
* 83~101번 줄 : `DecoretorMaterials` 메서드에 사용할 전달하여 충분한 자원을 보유했는지 확인하고 각각 소모한다.
* 103~110 줄 : 레벨을 증가시켜 기록한다.

### 승급 요청
앞서 살펴본 강화 요청과 승급은 거의 비슷하다.

다른점이라면 불러야하는 테이블이 다르고 에러 코드가 다르다는 것 정도다.

```javascript
{% github_sample totuworld/Wendy/blob/tuto5.4/routes/item.js 261 380%}
```

## 에러코드 추가

아이템과 관련한 공통 에러가 3개, 강화와 승급에 각각 3개의 에러코드가 추가된다.

앞서 사용하도록 코드를 작성했으므로 `utils/error.js`에 필요한 에러코드를 추가하도록 한다.

먼저 아래 9개 클래스를 등록한다.

> 다른 클래스가 등록된 것을 참고하여 붙여넣으면 된다.

* NotInOwnItemList
* MaterialDoesNotExistOwnItemList
* NotEnoughItem
* CantReinfoceItem
* DidntRegisterReinfoceRequireItem
* NoLongerReinforce
* CantUpgradeItem
* DidntRegisterUpgradeRequireItem
* NoLongerUpgrade

```javascript
{% github_sample totuworld/Wendy/blob/tuto5.2/utils/error.js 79 159%}
```

9개 클래스를 모두 등록했으면 다음 부분을 찾아서 아래와 같이 수정한다.

* 찾아야하는 내용

      "NickNameToLongOrShot":NickNameToLongOrShot

* 변경할 내용

```javascript
{% github_sample totuworld/Wendy/blob/tuto5.2/utils/error.js 167 179%}
```

## 테스트
아이템을 몇개 등록하고 강화와 승급도 진행해보자.

### 정의된 아이템 추가

`DefineItem` 테이블에 데이터를 입력한다.

ItemID | ItemType | Name | Multiple | MaxQNTY | ReinforceItemID | UpgradeItemID
--- | --- | --- | ---
101 | 10 | key | 1 | 500 | NULL | NULL
102 | 10 | gem | 1 | 900000 | NULL | NULL
103 | 10 | gold | 1 | 900000 | NULL | NULL
2001 | 1 | sward | 0 | 1 | NULL | NULL

> ItemType 10은 통화를 지칭하기로 약속한다.

모든 길은 아이템으로 통해야 추후 관리가 쉽다. 그래서 통화도 아이템으로 등록한다.

다만 통화는 Currency에서 별도 관리는 것이라고 생각하면 된다.

> materialCtrl.js 파일을 보면 공통 요청을 하지만 ItemType에 따라 적용하는 메서드가 달라지는 것을 볼 수 있다.

2001번 아이템은 현재는 강화와 승급이 불가능하다. 강화 승급에 관련한 데이터를 추가로 등록하자.

### 강화 관련 데이터 입력
아이템 강화와 관련된 테이블은 2개다.

`DefineReinforceItem`과 `DefineReinforceRequireItem`.

DefineReinforceItem을 먼저 입력한다.

| ReinforceItemID |
| --- |
| 2001 |

그리고 DefineReinforceRequireItem에 아래 내용을 입력한다.

> 소모할 아이템이 통화로 사용할 gem, gold 뿐이라서 여기서는 모두 gold로 처리한다.

id | Level | RequireQNTY | ReinforceItemID | ItemID
NULL | 1 | 10 | 2001 | 103
NULL | 2 | 20 | 2001 | 103

### 승급 관련 데이터 입력
승급도 강화와 마찬가지로 2개 테이블에 입력해야한다.

먼저 DefineUpgradeItem을 입력한다.

| UpgradeItemID |
| --- |
| 2001 |

그리고 DefineUpgradeRequireItem에 아래 내용을 입력한다.

id | Tier | RequireQNTY | ReinforceItemID | ItemID
NULL | 1 | 100 | 2001 | 103
NULL | 2 | 200 | 2001 | 103

### 정의된 아이템 수정

DBeaver에서 DefineItem 테이블을 선택하고 ItemID 2001 상품의 `ReinforceItemID`와 `UpgradeItemID`를 각각 2001로 변경한다.

1. DefineItem 테이블 선택.

2. Data 탭에서 2001 아이템을 아래처럼 수정

    ![ReinforceItemID/UpgradeItemID변경](/images/rvwendy05_01.png){:width="500px"}

3. 아래쪽에 `Save`를 클릭하여 변경사항을 반영한다.

    ![save](/images/rvwendy03_09.png)

### 보유한 아이템 추가

1번 유저에게 2001번 아이템을 지급하자.

`OwnItem`테이블에 아래 내용을 추가한다.

OwnItemUID | ItemID | CurrentQNTY | Level | Tier | UpdateTimeStamp | GameUserID
--- | --- | --- | --- | --- | --- | ---
NULL | 2001 | 1 | 1 | 1 | 2017-01-01 | 1

### postman으로 테스트

postman을 실행하고 토큰을 획득한 다음 아래처럼 요청을 해보자.

> 데이터를 로딩하는 4개 API는 테스트하지 않는다. 여기서는 강화와 승급만 다루겠다.

#### 강화 요청 테스트

* 패스 : POST
* URL : localhost:3000/item/reinforce/1

> 위 URL에서 가장 뒷 부분은 `OwnItemUID`이다. 자신의 테이블을 확인해보고 OwnItemUID가 필자와 다르다면 자신의 값을 넣어야한다.

* Headers : Authorization을 추가하고 토큰 내용을 value 부분에 넣는다.

* body : 아래 내용을 넣는다.

```json
{
    "materialItemUID":[],
    "materialCurrencyUID":[3]
}
```

> `materialCurrencyUID`에는 OwnCurrency 등록된 소모할 OwnCurrencyUID를 넣는다. CurrencyID 103가 등록된 OwnCurrencyUID를 넣어줘야 정상 동작한다.

성공하면 다음 내용을 수신한다.

```
{
  "result": 0
}
```

DBeaver에서 `OwnItem` 테이블의 Data탭에서 `새로고침`버튼을 클릭하면 Level이 2로 반영된 것을 확인할 수 있다.

![Dbeaver새로고침](/images/rvwendy05_02.png){:width="500px"}

#### 승급 요청 테스트

* 패스 : POST
* URL : localhost:3000/item/upgrade/1

> 위 URL에서 가장 뒷 부분은 `OwnItemUID`이다. 자신의 테이블을 확인해보고 OwnItemUID가 필자와 다르다면 자신의 값을 넣어야한다.

* Headers : Authorization을 추가하고 토큰 내용을 value 부분에 넣는다.

* body : 아래 내용을 넣는다.

```json
{
    "materialItemUID":[],
    "materialCurrencyUID":[3]
}
```

> `materialCurrencyUID`에는 OwnCurrency 등록된 소모할 OwnCurrencyUID를 넣는다. CurrencyID 103가 등록된 OwnCurrencyUID를 넣어줘야 정상 동작한다.

성공하면 다음 내용을 수신한다.

```
{
  "result": 0
}
```

DBeaver에서 `OwnItem` 테이블의 Data탭에서 `새로고침`버튼을 클릭하면 Tier이 2로 반영된 것을 확인할 수 있다.

## 맺음말

아이템까지 어찌어찌 왔으니 자원에 영향을 주는 기능을 만들 때 필요한 기반이 되었다.

> 인앱 영수증 검증, 쿠폰, 메시지, 상점, 가챠 등

다음 강좌에서 지급 기능을 만들고 차차 진행해보도록 하자.

[6강 바로가기](http://totuworld.github.io/2017/02/21/azureandunity-06/)

---

## 참고자료
완성된 소스코드는 아래 링크에서 다운로드받으면 된다.

[Wendy 5강 완료 버전](https://github.com/totuworld/Wendy/archive/0.0.4.zip)