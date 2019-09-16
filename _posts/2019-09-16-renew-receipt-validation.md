---
layout: post
title:  "구글, 애플 인앱 영수증 결제 검증 웹 서비스 리뉴얼"
date:   2019-09-16 00:01:00
categories: etc validation Billing iab iap nodejs TypeScript
comments: true
meta : iab iap validation
description : 2016년에 작성한 인앱 영수증 결제 검증 웹 서비스를 리뉴얼
image : 
  twitter: /images/rcp-custom-renew.png
  facebook: /images/rcp-custom-renew.png
publish : true
---

* content
{:toc}

## 들어가는 말
~~싸질러놓은~~ 작성한 글 중에서 가장 자연유입이 많은 것은 [구글 인앱 결제 검증 웹 서비스 만들기](/2016/02/10/google-oauth/)와 [애플 인앱결제 검증 웹 서비스 제작](/2016/03/23/ios-validation/) 이다.

그런데 이 글을 2016년에 작성되었고 그 코드는 3년이 넘게 그대로 방치 중이다. 그대로 사용하면 보안 위험이 있다. 

> 리뉴얼할 수 있게 요청해준 순순스튜디오 고마워요.

node.js와 express.js는 그대로 유지하면서 변경해보았다.

## 주요 기능
* Google Play Store 인앱결제(in app billing) 검증
  * JWT 토큰 생성 방식
* Apple App Store 인앱결제(in app purchase) 검증

게임에서 결제가 이뤄진 뒤 영수증을 각 url로 제출하면 정상 결제인지 확인한다.
> 단, 데이터베이스에 영수증을 기록하는 기능은 없기때문에 정상 결제 1건을 가지고 복수번 결제 검증을 요청하는 중복 결제(duplicate payment)는 방어하지 못함.

## 변경점
* node.js v10.16 사용
* `TypeScript` 사용
* 비동기 요청은 callback 에서 `async/await` 로 변경
* JSON Schema 를 통해서 request를 검증
* 요청 url 변경
  * /validation/iap/google
  * /validation/iap/apple
* aws beanstalk 배포를 위한 packing 스크립트 추가
* `dev` 모드로 실행 시 아래 명령으로 실행
```bash
npm run dev
```

## 레포지토리
[인앱 영수증 결제 검증 웹 서비스](https://github.com/totuworld/InAppPurchaseValidationWebService)

살펴보시거나 사용하시다가 질문이 있으시면 해당 레포지토리의 이슈로 등록하면 된다.

## 맺는말
이런 서비스가 필요없는 신뢰 사회가 되길 바란다.