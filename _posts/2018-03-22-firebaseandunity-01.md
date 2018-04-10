---
layout: post
title:  "3분 게임 서버(Firebase) - Auth 익명 로그인"
date: 2018-03-22 00:00:00
categories: nodejs firebase unity
comments: true
meta : nodejs firebase unity
description : Firebase 인증(Authentication)을 활용한 익명 로그인 제작
image :
  twitter: /images/fb_cover_02.png
  facebook: /images/fb_cover_02.png
publish : true
---

* content
{:toc}

## 들어가는 말

### 고백

[이세계에 진입한 서버 개발](http://totuworld.github.io/2016/12/21/azureandunity-01/)을 시작하고 1년이 지났다. 그 동안 이직을 했고 바쁜 시간을 보냈다는 핑계로 업데이트 없는 9개월을 보냈다.

놀기만 했으면 좋으련만 6월부터 `이세계에 진입한 서버 개발`을 책으로 출판할 생각으로 작업중이었는데 최근 절필했다. 2가지 이유가 있다. 하나는 오래된 코드라서 봐줄 수 없었다(이건 내 실력이...). 다른 하나는 게임 클라이언트 프로그래머만 있는 소규모 게임 개발팀에서 node.js와 관계형 데이터베이스를 공부해서 게임용 웹 서버를 만들 수 있을까하는 의문이다.

> 게임 개발할 시간도 없는데 서버는 젠장!

Wendy를 활용하는 게임 클라이언트 프로그래머가 인프라 공부에 시간 보낼까봐 PaaS(Platform as a Service)를 사용하도록 유도했지만 충분한 대답이 아니었다. 이보다 더 낮은 진입이 가능해야 순수 클라이언트 프로그래머가 공부할 수 있을 듯 했다. 그렇다면 좋은 서비스는 Parse나 Firebase같은 BaaS(Backend as a Service)다.

> Facebook이 인수하여 서비스하던 Parse는 서비스가 종료되었다.

### Firebase의 ~~약팔이~~ 새로운 기능

과거의 Firebase는 모바일 게임용 백엔드로 활용하기에 문제가 있었다. 게임 클라이언트에 Firebase SDK를 활용해 비지니스 로직을 넣게된다. 운영중에 비지니스 로직에 문제가 발생하면 모바일 게임은 플랫폼 심사 시간이 있어 빠른 대처가 어렵다. 출시 후 운영중에 대처가 늦으면 사용자 감소와 함께 매출이 감소한다. 이런 부담은 QA로 해소될 수 있으나 소규모 게임 제작팀은 비용때문에 불가능하다.

2017년 3월 Firebase에 node.js 환경으로 프로그래밍이 가능한 `Cloud Functions`이 추가되었다. 이 기능을 활용하면 비지니스 로직을 게임 클라이언트 외부로 옮길 수 있다. 또 Firebase에 `Realtime Database`외에도 `Cloud Firestore`란 데이터베이스가 추가되었다. 이제 거대한 파일 하나에 모든 데이터를 넣지 않아도 된다. 단, 아직까지 Firebase Unity SDK에서는 사용할 수 없다.

> MS에서도 [Playfab](http://Playfab.com)을 인수하여 Google의 Firebase같은 서비스를 제공할 예정이다.

새롭게 연재하는 글에서는 Firebase를 이용해서 게임 서버를 사용하도록 권장한다. 특히 클라이언트 프로그래머가 보유한 팀이라면 더더욱 이 방법을 추천한다.

> 처음부터 c++로 서버 프로그래밍하는 책 사지 마시라. 어느정도 필요가 차올랐을 때 아주 간단한 것부터 시작하길 권한다.

## 사용자 로그인 흐름
서론이 길었다. 이번에 진행할 것은 익명 로그인이다. 먼저 사용자가 게임을 시작했을 때 어떤 단계를 걸치는지 상상해보자.

* 타이틀 화면 
    * 게스트/페북 로그인 클릭(1회면 진행됨)

    ![타이틀 화면](/images/fb_02_01_01n.png){:width="50%"}

* 로그인 절차

    ![로그인](/images/fb_02_01_02n.png){:width="30%"}

* 게임 플레이

    ![게임 플레이](/images/fb_02_01_03n.png){:width="50%"}

이 과정을 거치는 이유는 제작자 혹은 운영자가 통제하는 시스템에 등록된 사용자인지 확인하기 위해서이다. 그럼 가장 간편한 로그인 방식인 익명 로그인 - 혹은 게스트 로그인 - 을 Firebase 인증(Authentication)을 활용해서 제작해보자.

가장 간편한 로그인 방식인 익명 로그인을 알아보자.

## 준비 과정
Unity 클라이언트에서 Firebase 인증을 활용해서 익명 로그인을 구현하려면 몇가지 선행 과정이 필요하다.

### 프로젝트 추가

Firebase 프로젝트 추가에 앞서 Firebase를 사용하려면 google 계정이 필요하다.

* google 계정으로 로그인 한다.
* [Firebase console](https://console.firebase.google.com/)로 이동한다.
* `프로젝트 추가`를 클릭한다.

    ![프로젝트 추가](/images/fb_01_01.png){:width="30%"}

* `프로젝트 이름`을 입력하고 `국가/지역`을 선택한 뒤 `프로젝트 만들기`를 클릭한다.

    ![프로젝트 추가](/images/fb_01_02.png){:width="50%"}

### 프로젝트 설정
Firebase를 클라이언트(Android, iOS, Web)에서 사용하려면 설정을 추가하면 된다.

> 여기서는 iOS로 설명하지만 Android라고 크게 다를건 없다.

* `Project Overview` 메뉴 옆에 기어 모양 설정 버튼을 클릭하고 프로젝트 설정을 클릭한다.
    ![프로젝트 설정](/images/fb_01_03_1.png){:width="50%"}

* 일반 탭에서 하단의 `iOS 앱에 Firebase 추가` 클릭한다.

    ![iOS 추가](/images/fb_01_03_2.png){:width="20%"}

* `iOS 번들 ID`를 입력하고 나머지 선택사항도 필요에 따라 입력한다.

    ![입력](/images/fb_01_03_3.png){:width="30%"}

* `GoogleService-Info.plist` 파일을 다운받고 계속 버튼을 눌러 등록 과정을 마친다.


## 익명 로그인
익명 로그인이 요구하는 사항은 간단하다. 여러가지 정보를 취합해서 게임 내에서 사용자를 고유하게 인식할 수 있으면 된다. 다음 절차로 진행된다.

* 사용자) 익명 로그인 버튼 클릭
* 게임 클라이언트) Firebase 인증 SDK에 익명 로그인 요청
* Firebase) 여러가지 정보 확인 후 고유한 사용자 ID 반환
* 게임 클라이언트) 고유한 사용자 ID 사용

### 프로그래밍 준비

* 먼저 간단한 UI를 추가해놓은 아래 Unity Project를 다운 받는다.

[샘플 프로젝트](https://github.com/totuworld/firebase_toturial/archive/base.zip)

* Firebase SDK를 다운받아서 Firebase Auth SDK를 설치한다. 그리고 앞서 Firebase 프로젝트에서 다운받은 `GoogleService-Info.plist`나 `google-services.json`을 추가한다.

* Firebase의 Authentication 메뉴에서 `로그인 방법` 탭을 클릭한 뒤 `익명` 사용 설정한다.

    ![Authentication](/images/fb_02_02_01.png){:width="30%"}
    ![익명 사용 설정](/images/fb_02_02_02.png){:width="80%"}

### 익명 로그인 구현
Firebase 인증을 활용할 때 초기화 후 익명 로그인 요청을 해야한다.

* unity 프로젝트의 프로젝트 탭에서 scripts 폴더를 만들고 `gfb_auth_test`란 이름의 새로운 c# script 파일을 생성한다.

> 이름은 달라도 무방하다.

* 추가한 스크립트를 편집할 수 있게 더블 클릭한 후 아래 코드를 복사하여 붙여넣는다.
    * 4~7번: using으로 필요한 네임스페이스 형식 사용 허용
    * 11~18번: 필요한 필드 추가
    * 23~49번: `auth`필드를 추가하고 상태변화 시 로그가 남도록 AuthStateChanged 메서드 연결.
    * 52~69번: 익명 로그인을 요청하는 메서드

{% gist /785a451847aebd36a548061a47cf261a gfb_auth_test.cs %}

* unity 프로젝트로 돌아가서 `test`씬의 `Main Camera` 게임 오브젝트에 `gfb_auth_test` 스크립트를 추가한다. Canvas - Scroll View - ViewPort - Content 게임 오브젝트를 `gfb_auth_test`의 Txt Print에 연결한다.

    ![게임 오브젝트 연결](/images/fb_02_03.png){:width="50%"}

* guest login 버튼을 클릭했을 때 gfb_auth_test의 `anoymousLogin` 메서드가 실행되어야 익명 로그인을 시도할 수 있다.
    * Canvas - Button 게임 오브젝트의 Button 컴포넌트를 찾는다.
    * On Click의 + 버튼을 클릭한 뒤 Main Camera 게임 오브젝트를 연결한다.
    * gfb_auth_test 스크립트의 anoymousLogin을 선택한다.

    ![버튼 연결](/images/fb_02_04.png){:width="50%"}

### 디버깅

unity 프로젝트에서 `play` 버튼을 클릭하여 작동 시킨 뒤 화면 중앙에 나타나는 `guest login` 버튼을 클릭한다. 아래 스크롤뷰에 즉시 `Sign in...` 메시지가 출력된다.

> 모바일 앱으로 빌드해서 확인하려면 앞서 다운받은 `GoogleService-Info.plist`나 `google-services.json` 파일을 반드시 프로젝트에 추가해야한다.

모바일 앱으로 빌드해서 해당 과정을 진행하면 Firebase console에 아래처럼 익명 사용자가 등록되는 것을 확인할 수 있다.

![익명 사용자 확인](/images/fb_02_05.png){:width="80%"}

## 마무리

Firebase 인증을 활용하여 익명 로그인을 만들어봤다. 이를 바탕으로 다음에는 소셜 로그인 기능을 추가해보자.