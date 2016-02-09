---
layout: post
title:  ""
date:   2016-02-11 10:00:00
categories: etc validation Billing iab nodejs
comments: true
meta : google iab validation
description : Node.js로 구글 결제 검증 웹 서비스 구현 
publish : false
---

* content
{:toc}

## 들어가는 말

인디게임을 개발하고 런칭한 이후 이슈는 무엇일까?

`홍보`와 `버그`가 주된 이슈겠지만 가장 화가나는 일은 `거짓 결제`다.

특히 안드로이드 버전의 경우 결제 로그의 상당수가 거짓말이다.

열심히 [공식 결제 검증 가이드](http://developer.android.com/intl/ko/google/play/billing/billing_integrate.html#billing-security)를 따랐는데도 문제가 발생하니 더 미칠노릇이다.

---
 
## 왜 이럴까?

[공식 결제 검증 가이드](http://developer.android.com/intl/ko/google/play/billing/billing_integrate.html#billing-security)를 확인해보면 구글의 결제 검증은 구글 서버를 거치지 않는다.

이게 무슨 소리일까? 

구글의 인앱 결제 과정을 먼저 살펴보자(여기서는 init 과정은 생략한다).

### 구매 과정

1. 클라이언트가 구매 요청(`getPurchase()`)을 보낸다.
2. 구글 Play App이 결제와 관련된 정보를 돌려준다.
3. 클라이언트가 소진을 요청(`consumePurchase()`)한다.
4. 구글 Play App이 성공여부를 돌려준다.
5. 클라이언트가 성공 시 상품을 지급한다.


예제를 따라서 진행했다면 결제가 결제가 성공한 시점에서 변조된 결제인지 체크하게된다.

### 예제만 믿었는데 이게 아닌가?

문제는 이 지점에서 발생한다.

대부분 예제를 그대로 따라한 결제 검증만 사용하기 때문이다. 예제를 따라한게 왜 잘못인가?

구글은 결제와 관련된 정보를 전달할 때 [공개키 암호화 방식](https://ko.wikipedia.org/wiki/%EA%B3%B5%EA%B0%9C_%ED%82%A4_%EC%95%94%ED%98%B8_%EB%B0%A9%EC%8B%9D)으로 결제 사항을 싸인(sign)된 JSON 문자열로 전달한다. 

예제에서는 공개키를 사용해서 결과가 조작되었는지 확인하는데 apk가 털려서 공개키가 들통나면 가짜 영수증을 마음껏 생성해도 결제 검증이 된다.

### 애플은 다르다.

그런데 애플이 공식 문서를 살펴보면 대놓고 [App Store를 통한 영수증 검증](https://developer.apple.com/library/ios/releasenotes/General/ValidateAppStoreReceipt/Chapters/ValidateRemotely.html)예시가 나온다.

클라이언트에 반환된 영수증(결제 정보)가 올바른지 App Store의 검증 URL로 전달해서 실제 존재하는 영수증인지 확인하는 것이다.


---

## 구글 API로도 할 수 있다

구글에서 검색하다보면 [Android In-App Billing 보안 완벽 정리](http://theeye.pe.kr/archives/2130)를 보게 될 것이다.

이 글의 제일 마지막즈음에 나오는 `중요 보안팁` 내용을 살펴보자.

![중요 보안팁](http://theeye.pe.kr/wp-content/uploads/2014/02/android_iab3_07-600x564.png) 

해당 내용은 위 이미지 하나로 압축된다.

구글에서도 [서버에서 호출가능한 API](https://developers.google.com/android-publisher/api-ref/purchases/products)가 있으니 이를 통해서 `Google 앱내결제제 체크 API & 검증`단계를 거치는 것이다.

### 뭘로 어떻게 하는거야?

[Purchases.products: get](https://developers.google.com/android-publisher/api-ref/purchases/products/get#http-request)를 살펴보면 아주 심플하게 요청이 가능하다.

```
GET https://www.googleapis.com/androidpublisher/v2/applications/`packageName`/purchases/products/`productId`/tokens/`token`
```

호출 시 앱마다 변경되는 부분은 크게 아래 3가지이다.

파라미터 명 | 값 형식 | 설명
--- | --- | ---
packageName | string | 인앱 상품이 포함된 어플리케이션의 패키지명 (예, 'com.some.thing')
productId | string | 인앱상품 SKU(Stock Keeping Unit), 다시말해 상품명 (예, 'com.some.thing.inapp1')
token | string | 인앱상품 구매시 기기에서 제공되는 토큰

위 URI에 파라미터만 올바로 입력하여 구글 서버에 요청하면 된다. 보기보다 간단하다. 


### 거대한(?) 장벽, 허가(Authorization)

매우 간단하게 설명되어있는데 자세히 읽어보면 생소한 내용이 하나 들어있다.

상단 부분에 별표까지 넣어서 강조한 사항인 `Requires authorization`이다.

아래 범위(Scope)에 관한 허가가 있어야만 이용가능한 것이다.

```
https://www.googleapis.com/auth/androidpublisher
```

무턱대고 검증 요청을 진행해도 올바른 결과가 나오지 않는 것은 이 때문이다.

### 그런데 이게 왜 장벽인가?

이유는 `OAuth 인증 방식`을 통해서 허가가 진행되기 때문이다.

> OAuth가 궁금하다면 [OAuth와 춤을](http://d2.naver.com/helloworld/24942)의 앞부분을 읽어보시기 바란다.

게임 개발에 매진하다보니 클라이언트와 관련된 것은 익숙하다. 그러다보니 웹 서버와 관련된 사항이 생소하여 이해하기 힘들다.

그렇지만 언제나 그랬듯 이또한 극복되리라.

---

## 웹 서비스 제작

서론에서 얘기한 부분을 바탕으로 제작할 웹 서비스를 간단히 정의해보면 다음 두 가지다.

1. 허가 획득
2. [Purchases.products: get](https://developers.google.com/android-publisher/api-ref/purchases/products/get#http-request)을 활용한 결제 영수증 검증 

2번이 주된 목적인데 1번을 처리하느라 더 많은 노력을 드리는 본말전도가 일어날 것이다.

> 안드로이드 제작자가 삼성에게 구매를 요구했을 때 샀어야했다. 그랬다면 Bada나 Tizen처럼 몰라도 되는 것일텐데...

### Express 프로젝트 기초 작업

Node.js 설치와 Express 제네레이터가 설치되었다고 가정하겠다.[^1]
다음 명령으로 Express 프로젝트를 생성하고 필요한 모듈을 설치한다. 

    $ express GoogleIABValidation
    $ cd GoogleIABValidation
    $ npm install
    
설치가 완료되면 구글 API 모듈과 request 모듈을 설치한다.

    $ npm install googleapis request --save
    
> `npm`으로 모듈을 설치할 때 뒤에 `--save`를 붙이면 `package.json`파일의 `dependencies`에 추가된다. 추가된 정보를 바탕으로 배포시 npm install 명령으로 모듈을 일괄 설치할 수 있어서 편하다.

기본 설치는 모두 끝났다.

이제 `routes/index.js`파일 전체 내용을 아래처럼 교체한다.

{% highlight javascript linenos %}
'use strict';

const express = require('express');
const router = express.Router();

const request = require('request');
const readline = require('readline');
const google = require('googleapis');

module.exports = router;
{% endhighlight %}

---

## 허가 기능 제작

구글 서버와 허가를 얻는 과정을 대화로 표현하면 아래와 같다.

![허가 주세요](/images/authorization.png)

위 과정에서 웹 서비스가 처리하는 부분은 `토큰 요청`과 `코드와 토큰 교환`이다.


### 토큰 요청 필요사항

[구글 공식 문서](https://developers.google.com/identity/protocols/OAuth2WebServer#redirecting)를 참고하면 `https://accounts.google.com/o/oauth2/v2/auth`이 URL로 query string 매개변수를 포함하여 요청하도록 되어있다.

공식 문서의 예시

```
https://accounts.google.com/o/oauth2/v2/auth?
scope=email%20profile&
state=security_token%3D138r5719ru3e1%26url%3Dhttps://oa2cb.example.com/myHome&
redirect_uri=https%3A%2F%2Foauth2-login-demo.appspot.com%2Fcode&,
response_type=code&
client_id=812741506391.apps.googleusercontent.com
```

위와 같은 URL을 `googleapis`모듈로 생성할 수 있다. 단 `CLIENT_ID`, `CLIENT_SECRET`, `REDIRECT_URL` 3가지를 준비해야한다.


### 필요사항 준비

구글 개발자 콘솔에서 `OAuth`로 사용자를 인증하는 아이디를 등록해서 `CLIENT_ID`, `CLIENT_SECRET`를 얻는 과정을 알아보자. 

1. [구글 개발자 콘솔 API 관리자](https://console.developers.google.com/apis)에 접속한 후 `사용자 인증 정보` 탭으로 이동한다.

2. `사용자 인증 정보 - 새 사용자 인증 정보` 버튼을 클릭하고 `OAuth 클라이언트 ID`를 선택한다.  
    ![OAuth 클라이언트 ID 생성](/images/OAuth_clientid.png)

3. `웹 어플리케이션`을 선택하고 `이름`을 입력한 후 `생성`버튼을 클릭한다.
    ![클라이언트 ID 만들기](/images/OAuth_clientid1.png)

4. 생성이 완료되면 `클라이언트 ID`와 `클라이언트 보안 비밀`이 출력된다.
    ![클라이언트 ID 확인](/images/OAuth_clientid2.png)

이로써 `CLIENT_ID`, `CLIENT_SECRET`는 확보되었다. `REDIRECT_URL`은 이후 과정에서 업데이트하도록 하겠다.



허가를 통해서 얻게되는 토큰은 아래 4가지 정보를 포함하고 있다.

명칭 | 역할
--- | ---
access_token | 사용할 토큰
token_type | 토큰 종류
expires_in | 토큰 사용 종료시간
refresh_token | 토큰 갱신용 토큰

이 정보를 `routes/index.js` 안에 `tokenStorage` 변수로 저장한다.

{% highlight javascript %}
var tokenStorage = {
    access_token: null,
    token_type: null,
    expires_in: null,
    refresh_token: null
};
{% endhighlight %}

TODO : 전체 서비스가 작동하는 모습을 보여주기 위해서 변수를 등록하고 이를 통해서 분기하는 과정을 알려주도록 하자

---

[^1] : [Node.js + express 프로젝트 구성]()을 참고.