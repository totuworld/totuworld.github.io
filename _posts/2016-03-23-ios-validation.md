---
layout: post
title:  "애플 인앱결제 검증 웹 서비스 제작"
date:   2016-03-23 00:01:00
categories: etc validation Billing iab iap nodejs
comments: true
meta : ios iap validation
description : Node.js로 애플 인앱결제(IAP) 결제 영수증 검증 웹 서비스 구현 
publish : true
---

* content
{:toc}

## 들어가는 말

아이폰 사용자는 안드로이드보다 탈옥 과정이 복잡하고 업데이트때마다 난리를 쳐야한다. 탈옥이 어려우니 탈옥후에 진행되는 결제 크랙도 적을 줄 알았다.

하지만 [도트레인저스](http://drserverdev.japanwest.cloudapp.azure.com:3001) 출시 후 로그를 살펴보니 내가 너무 순진했다.

이런 실수를 막기 위해 [구글 인앱 결제 검증 웹 서비스 만들기](/2016/02/10/google-oauth)처럼 애플용 웹 서비스도 제작하는 과정을 살펴보도록 하겠다.


---
 
## 웹 서비스 제작

애플의 인앱결제 검증은 간단하다. 결제 후 얻은 영수증을 애플이 제공하는 REST API로 송신하여 결과를 받아보면 끝이다.

### Express 프로젝트 생성

초기 작업을 또 진행하지 않는다. 대략의 과정은 [구글 인앱 결제 검증 웹 서비스 만들기](/2016/02/10/google-oauth)에서 이미 설명했다.

오늘은 이미 제작된 소스를 가지고 시작하겠다.

1. [InAppPurchaseValidationWebService v0.0.1](https://github.com/totuworld/InAppPurchaseValidationWebService/archive/v0.0.1.zip) 파일을 다운로드한 후 압축을 해제한다(이하 압축 해제 폴더를 `inapppurchaseWebService`라고 지칭한다).

2. 압축해제한 폴더로 이동하여(inapppurchaseWebService 폴더) 필요한 module을 설치한다.

    {% highlight bash%}
    $ npm install
    {% endhighlight %}

---

## 결제 영수증 검증 제작

이제 본격적으로 검증 서비스를 제작하겠다.


사용할 모듈을 추가한다.

{% highlight bash%}
$ npm install --save in-app-purchase
{% endhighlight %}

모듈이 해주는 역할은 두 원격지(`buy.itunes.apple.com`와 `sandbox.itunes.apple.com`)에 `/verifyReceipt` path 영수증을 전송해서 올바른 값인지 확인하는 것이다. 허가 부분이 별도로 존재하지 않아서 몹시 간단하지만 여기서는 모듈로 간단히 처리하도록 한다.

이제 `routes/index.js`파일의 10번 줄에 다음 내용을 추가한다.

{% gist totuworld/8687dc2c59d26ca97ae9 index.js %}

1~12 번째 줄 : 모듈을 사용하기 위해서 불러온 뒤 기본 설정을 진행하는 부분이다.

31~46 번째 줄 : 검증을 진행하는 부분이다.


너무 간단해서 어떻게 표현할 방법이 없다.

![방법이 없다](https://i.ytimg.com/vi/CaAZ6TCMiTY/hqdefault.jpg)


### 사용방법

사용도 간단하다. 

{% highlight bash%}
$ node /bin/www
{% endhighlight %}

실행한 뒤 `localhost:3000/appleiap/receipt/validation` 주소로 `RawReceipt`포함하여 POST 방식으로 호출하면 끝이다.

그럼 boolean 값을 얻게된다.


### 주의 사항

한가지 주의 사항이라면 `RawReceipt`의 형식이다.

필자는 아래에서 나오는 것처럼 영수증을 String 형태로 파싱해서 사용했다.

{% highlight json%}
{
    "RawReceipt":"{\"verification-state\":0,\"transaction-receipt\":\"MIISiAYJKoZIh=\",\"product-identifier\":\"yoyo\",\"transaction-identifier\":\"8FXXX\",\"quantity\":1,\"transaction-state\":1,\"error\":\"\"}"
}
{% endhighlight %}

만약 JSON 형식으로 보낸다면 17번 줄을 다음처럼 변경해야한다.

{% highlight javacript%}
{
    let parseRawRecipt = req.body.RawReceipt;
}
{% endhighlight %}


---

## 맺음말

Apple 인앱결제 확인은 허가 과정이 없다는 이유만으로 매우 간단하다. 그러니 꼭 검증 서버를 마련해서 결제 검증을 진행하기 바란다.

---

[완료된 웹 서비스 소스코드](https://github.com/totuworld/InAppPurchaseValidationWebService/archive/v0.0.2.zip)