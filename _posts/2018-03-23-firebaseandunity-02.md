---
layout: post
title:  "3분 게임 서버(Firebase) - Auth 소셜 로그인"
date: 2018-03-22 00:00:00
categories: nodejs firebase unity
comments: true
meta : nodejs firebase unity
description : Firebase 인증(Authentication)을 활용해 Facebook 로그인 제작
image :
  twitter: /images/fb_cover_03.png
  facebook: /images/fb_cover_03.png
publish : true
---

* content
{:toc}

## 들어가는 말

지난 시간에 다룬 [익명 로그인](https://blog.totu.dev/2018/03/22/firebaseandunity-01/)은 기본이다. 익명 로그인은 Apple의 앱스토어 검수 조건에도 해당한다.

사용자가 선택할 수 있는 그 다음 선택지는 소셜 로그인이다. 비밀번호와 같은 자산이 Facebook이나 Google 등 대형 회사가 관리되니 안심되고 설령 게임을 지워도 다시 데이터를 복원을 기준을 제공해주기 때문이다.

Firebase 인증은 Google, Play 게임(구글 플레이), Facebook, Twitter, Github 등의 로그인이 가능하다. 그 중 여기서는 페이스북 로그인을 다뤄보겠다.

## 준비 과정
Firebase 인증의 `로그인 방법`에 Facebook을 등록해야한다. 그러기위해서는 Facebook에 앱을 생성해야한다.

* [facebook for developers의 앱 대시보드](https://developers.facebook.com/apps/)에 접속 한 뒤 `새 앱 추가` 버튼을 클릭한다.

    ![새 앱 추가](/images/fb_03_01_01.png){:width="10%"}

* 표시할 이름과 연락처 이메일을 등록한다.

    ![새 앱 ID만들기 입력](/images/fb_03_01_02.png){:width="50%"}

* 페이스북 앱의 대시보드 중 `Facebook 로그인`의 `설정` 버튼을 클릭한다.

    ![Facebook 로그인 설정](/images/fb_03_01_03.png){:width="30%"}

* 이제 `빠른 시작` 절차가 시작된다. 이를 무시하고 좌측 메뉴에서 `설정` - `기본 설정`을 클릭하여 `앱 ID`와 `앱 시크릿 코드`를 확인한다.

    ![설정 - 기본 설정](/images/fb_03_01_05.png){:width="30%"}

    ![앱 ID와 앱 시크릿 코드 확인](/images/fb_03_01_06.png){:width="30%"}

* 하단의 `+ 플랫폼 추가` 버튼을 클릭하고 iOS나 Android 등의 플랫폼을 선택한다(여기서는 iOS를 기준으로 설명한다). `번들 ID`를 입력한다. 다 입력하면 하단의 `변경 내용 저장`을 클릭한다.

    ![iOS 번들 ID 입력](/images/fb_03_01_07.png){:width="80%"}

* 이제 아대시보드로 접속하여 `로그인 방법`탭을 선택한 뒤 `Facebook`을 클릭한다.

* `사용 설정`을 클릭하고 앞서 확인한 `앱 ID`와 `앱 시크릿 코드`를 입력한다. `OAuth 리디렉션 URI`를 복사한다(복사 아이콘 클릭)하고 `저장` 버튼을 클릭한다.

    ![앱 ID, 앱 시크릿 코드 입력](/images/fb_03_01_08.png){:width="60%"}

* facebook for developers 대시보드로 돌아가서 복사한 `OAuth 리디렉션 URI`를 `Facebook 로그인` - `설정`에서 `유효한 OAuth 리디렉션 URI`에 붙여넣고 `변경 내용 저장`을 클릭한다.

    ![좌측 메뉴의 Facebook 로그인 - 설정](/images/fb_03_01_09.png){:width="30%"}

    ![유효한 OAuth 리디렉션 URI 입력](/images/fb_03_01_10.png){:width="40%"}

## Facebook 로그인

Facebook 로그인도 익명 로그인과 비슷한 절차로 진행된다.

* 사용자) Facebook 로그인 버튼 클릭
* 게임 클라이언트) Facebook SDK를 활용해서 로그인 화면 표시
* 사용자) 로그인 진행
* Facebook) access token 반환
* 게임 클라이언트) access token을 Firebase 인증 SDK를 통해 등록
* Firebase) 사용자 등록 과정 진행
* 게임 클라이언트) 고유한 사용자 ID를 사용


### 프로그래밍 준비

> [익명 로그인](https://blog.totu.dev/2018/03/22/firebaseandunity-01/)을 진행했다면 그 프로젝트를 이어서 사용해도 무방하다. 

* 아래 프로젝트 파일을 다운받는다.

    [샘플 프로젝트](https://github.com/totuworld/firebase_toturial/archive/auth_guest.zip)

* [Facebook SDK for Unity](https://developers.facebook.com/docs/unity)로 접속해서 Facebook Unity SDK를 다운로드한다(작성일 기준 v7.11.1).

* Unity 2017.3를 실행한 뒤 샘플 프로젝트를 열고 [익명 로그인](https://blog.totu.dev/2018/03/22/firebaseandunity-01/#프로젝트-설정)의 `프로젝트 설정` 부분을 참고하여 `GoogleService-Info.plist` 파일을 생성하여 샘플 프로젝트에 추가한다. 다운로드한 Facebook Unity SDK도 추가한다.


    Facebook Unity SDK를 추가하면 Firebase Auth SDK와 충돌하는 파일이 4개 있다. 프로젝트에 사용된 Firebase Unity SDK는 4.3이고 Facebook Unity SDK는 7.11.1이다. 충돌이나는 파일은 `PlayServicesResolver의` 폴더에 있는 4개 파일인데 1.2.59 버전 파일을 모두 삭제한다.

    ![PlayServicesResolver 내 파일 삭제](/images/fb_03_02_01.png){:width="80%"}

    > Android 빌드는 확인해보지 않아서 어떤 문제가 발생할지 모릅니다.

* Unity에서 `Facebook` - `Edit Settings` 메뉴 선택한 뒤 facebook for developers에서 확인한 `앱 ID`를 입력한다.

    ![Facebook - Edit Settings 메뉴 선택](/images/fb_03_02_02.png){:width="20%"}
    ![Facebook 앱 ID 입력](/images/fb_03_02_03.png){:width="50%"}


### Facebook 로그인 구현

Facebook 로그인은 2 단계로 진행된다. Facebook Unity SDK로 로그인을 진행하고, Facebook에서 반환한 access token으로 Firebase Auth SDK로 등록을 진행한다.

* `gfb_auth_test.cs` 스크립트를 열고 아래처럼 변경한다.
    * 8번 줄: using으로 필요한 네임스페이스 형식 사용 허용(Facebook.Unity)
    * 29~32번 줄(`Start` 메서드 내): Facebook SDK를 초기화 추가
    * 40~45번 줄: Firebase 인증 SDK로 signin이 된 상태인지 확인하는 프로퍼티 추가
    * 90~184번 줄(페이스북 로그인 관련 메서드 region): 페이스북 로그인을 시도하고 firebase에 등록하는 메서드가 모여있다. 아래 흐름도를 살펴보고 코드를 보면 조금 더 이해하기 수월하다.

![페이스북 흐름도](/images/fb_03_03_01.png)

{% gist /785a451847aebd36a548061a47cf261a gfb_auth_test_for_fb.cs %}

* gfb_auth_test의 `facebookLogin` 메서드가 실행되어야 페이스북 로그인을 시도할 수 있다.
    * Canvas - Button 게임 오브젝트의 Button 컴포넌트를 찾아서 복사한 뒤 적당히 위치 시키고 `Facebook Login`으로 텍스트를 변
    * gfb_auth_test 스크립트의 `facebookLogin`을 선택한다.

    ![버큰 복사 후 텍스트 변경](/images/fb_03_03_02.png){:width="30%"}
    ![버튼 연결](/images/fb_03_03_03.png){:width="50%"}

### 디버깅

Firebase 인증 SDK로 Facebook 로그인을 시도하면 Unity Editor에서는 디버깅이 불가능하다. 빌드 후 앱 클라이언트에서 확인해야한다.

> iOS의 빌드의 경우 Facebook SDK 추가 후 빌드 시 Build Settting에서 Enable Bitcode를 No로 설정해야 linker 에러 없이 진행된다.

![등록 확인](/images/fb_03_04_01.png)

Facebook 로그인 후 Firebase에 등록을 마치면 위 그림처럼 나타난다.

[완료된소스코드](https://github.com/totuworld/firebase_toturial/archive/auth_facebook.zip)

## 마무리

간단히 끝날 내용이었는데 linker 에러와 Firebase 내의 Facebook 앱 ID를 잘못 입력해서 많은 빌드 실패가 있었다. 꼭 설정은 2번 3번 확인하길 바란다.

그리고 이런 다양한 문제를 해결하고 있는 클라이언트 프로그래머 여러분 대단하십니다👍👍👍.

---

## 참고자료
* [Authenticate Using Facebook Login and Unity](https://firebase.google.com/docs/auth/unity/facebook-login)
* [Link Multiple Auth Providers to an Account in Unity](https://firebase.google.com/docs/auth/unity/account-linking)
* [Facebook SDK for Unity - Examples](https://developers.facebook.com/docs/unity/examples#login)
* [Facebook SDK for Unity isssue 135](https://github.com/facebook/facebook-sdk-for-unity/issues/135)