---
layout: post
title:  "Azure 웹앱 시작하기"
date:   2016-05-30 00:01:00
categories: nodejs azure webapps
comments: true
meta : nodejs azure webapps webservice
description : Azure 앱 서비스인 웹앱을 사용하는 기본 사용방법을 익힌다.  
image : 
  twitter: /images/webapp_tutorial_title_01.png
  facebook: /images/webapp_tutorial_title_01.png
publish : true
---

* content
{:toc}

## 들어가는 말

[이상한모임](http://blog.weirdx.io)에서 얘기를 하던 중 Azure 앱 서비스 중 하나인 웹앱이 사용하기 편한데 기본 내용이 부족하단 말을 들었다.

![치요아버님say](/images/devcloud_chiyodad.png){:width="300px"}

동감한다. 웹앱을 사용하기까지 기본 튜토리얼을 얼마나 많이 봤는지 모르겠다.

어색한 단어 선정으로 이해를 어려운데 개선할 수 도 없다.

> 영문 튜토리얼처럼 github로 고칠 수 있게 해달라!!

이런 상황에 누군가 웹앱의 문턱에서 엄한 삽질을 피하길 기대하며 시작해보겠다!

---

웹앱을 시작할 때 크게 2가지만 알면 된다. 첫째는 웹앱을 생성하는 것이고 둘째는 자신의 코드를 배포하는 하는 것이다.

이번에는 이 두가지를 집중하여 익혀보자.

## 웹앱 만들기

웹앱을 생성하는 것은 Azure 포털을 통한 방법과 Azure CLI를 통한 방법이 있으나 여기서는 쉽게 이해할 수 있는 Azure 포털을 이용하겠다.

> 이미지를 보고 따라하기 힘들다면 아래 비디오를 참고하면 된다.

{% include youtubePlayer.html id="2NANGcI_8iY" %}

1. [Azure 포털로 접속](https://portal.azure.com)한다.

2. `새로 만들기` 클릭한다.

    ![새로만들기](/images/webapp_a01.png)

3. `웹 + 모바일`을 선택한다.

    ![웹+모바일](/images/webapp_a02.png)

4. `웹앱`을 선택한다.

    ![웹앱](/images/webapp_a03.png)
    
5. `앱이름`을 입력하고 `리소스 그룹`을 선택한다. `만들기`를 클릭하면 생성이 시작된다.

    > 리소스 그룹은 새로 만들어도 된다.

    ![웹앱입력](/images/webapp_a04.png)
    
위 과정이 끝나면 웹앱을 확인할 수 있다.

![웹앱확인](/images/webapp_a05.png)

`웹앱`이 편리한 것은 생성만으로 접근가능한 URL이 생성되고 ftp를 통해 소스코드를 배포하고 로그를 확인할 수 있도록 설정된다는 점이다.


## 어플리케이션 배포하기

웹앱이 생성되었으면 이제 소스코드를 배포하여 자신이 원하는 형태로 가공해야한다.

웹앱에 소스코드를 배포하는 것은 ftp를 이용하거나 github의 특정 브랜치와 연결하여 연속 배포가 가능하게도 할 수 있다.

또한 각 웹앱마다 로컬 Git 레포지토리를 생성하고 git을 통해 commit하고 push하여 배포할 수 있다.

이번에는 git을 활용할 수 있는 `로컬 Git 레포지토리`를 설정하여 배포하는 방법을 설명할 것이다.

### 배포 원본 설정

말이 조금 이상하게 들리지만 배포 원본은 영문 사이트에서 Deploment Source라는 메뉴이다.

배포 시 소스를 어떤 방식으로 관리할지 결정한다고 보면 된다.

1. 생성한 웹앱에서 `설정`을 클릭한다.

    ![웹앱설정](/images/webapp_a06.png)

2. 설정 메뉴 중에서 `배포 원본`을 클릭한다.

    ![배포원본](/images/webapp_a07.png)
    
3. 배포 원본 메뉴에서 `소스 선택`클릭한다.

    ![소스선택](/images/webapp_a08.png)
    
4. 원본 선택 메뉴에서 `로컬 Git 레포지토리`를 선택한다.
    
    ![로컬Git레포지토리](/images/webapp_a09.png)

5. 선택이 완료되었으니 `확인`을 클릭하여 완료한다.
    
    ![확인](/images/webapp_a10.png)

설정이 끝나면 아래처럼 웹앱 메인 화면에서 `Git 복제 URL`을 확인할 수 있다.

![Git복제URL확인](/images/webapp_a11.png)

### 테스트 어플리케이션 배포하기(feat. CLI)

Azure 웹앱을 생성했고 배포하는 방식도 결정했다. 이제 Git을 통한 배포를 확인해보자.

먼저 커맨드 라인 인터페이스(Command Line Interface)로 하는 방법이다. 

> CLI가 익숙치않다면 다음 단계로 건너뛴다. [테스트 어플리케이션 배포하기(feat. SourceTree)](#feat-sourcetree))

1. CLI 창(OSX의 터미널이나 Windows 10의 bash)을 켜고 아래 명령을 순서대로 입력한다. 

    {% gist /161542851ecbd0276501d56e6ca42a44 downloadapp %}
    
2. Azure 포털에서 웹앱을 선택하고 `Git 복제 URL`를 클릭하여 복사한다.

    ![Git복제URL확인](/images/webapp_a11.png)
    
3. 아래 명령을 CLI에 입력한다. 단, `[웹앱 git 주소]`라고 써진 부분은 반드시 자신의 주소로 치환해야한다.

    > AzureWebAppsTutorial-0.0.1 폴더 안에서 입력해야한다.
    
    {% gist /161542851ecbd0276501d56e6ca42a44 addgitremote %}
    
 위 코드는 git 으로 `AzureWebAppsTutorial-0.0.1`폴더를 관리할 수 있도록 한 뒤 `azure`란 이름으로 remote를 등록한 것이다.
 
 그리고 commit을 한 뒤 `azure` 리모트로 push하여 코드를 반영 시킨 것이다.
 
 실제로 push하면 아래와 같은 로그가 출력된다.
 
 {% gist /161542851ecbd0276501d56e6ca42a44 deploylog %}
 
> CLI로 위 과정을 마쳤다면 다음 단계는 건너뛴다.
    

### 테스트 어플리케이션 배포하기(feat. SourceTree)

CLI가 익숙치않다면 웹 브라우저와 [SourceTree](https://www.sourcetreeapp.com/)나 [Tower](https://www.git-tower.com/)등의 프로그램을 활용해서 처리하는 방법도 있다.

> 여기서는 OSX에서 구동되는 SourceTree로 설명한다(Windows에서는 메뉴 위치등이 다를 수 있다).

1. [SourceTree](https://www.sourcetreeapp.com/)를 설치한다.

2. 아래 링크를 클릭하여 소스코드를 다운받는다.

    [AzureWebAppsTutorial-0.0.1 링크](https://github.com/totuworld/AzureWebAppsTutorial/archive/v0.0.1.zip)
    
3. 압축을 해제한다.

4. SourceTree를 실행하여 `New Repository`클릭한 뒤 `Add exist local repository` 메뉴를 선택하낟.

    > Windows에서는 File - Clone / New 선택 후 나오는 창에서 `Add Working Copy`탭으로 이동한다.
    
    ![NewRepository](/images/webapp_a12.png)

5. `Destination Path`에 압축해제한 어플리케이션 폴더를 선택한 후 `Create`를 클릭한다.

    > Windows에서는 Working Copy Path가 같은 역할을 한다.

    ![로컬레포지토리추가](/images/webapp_a13.png)
    
6. 추가된 레포지토리를 선택한다.

    ![레포지토리열기](/images/webapp_a14.png)

7. `Settings`를 클릭한다.

    ![설정](/images/webapp_a15.png)
    
8. Azure 포털에서 웹앱을 선택하고 `Git 복제 URL`를 클릭하여 복사한다.

    ![Git복제URL확인](/images/webapp_a11.png)
    
9. SourceTree로 돌아와 `Remote Name`을 azure로 입력하고 `URL`에 복사한 주소를 붙여넣고 `OK`를 클릭한다.

    ![AddAzureRemote](/images/webapp_a16.png)
    
10. `File Status`탭에서 `Staged files`를 클릭한 뒤 commit 메시지를 `init`으로 입력하고 `Push changes immdiatly to azure/master`옵션을 켜고 `Commit`한다.
    
    ![Commit](/images/webapp_a17.png)
    
11. Push 메시지에 아래와 같은 로그가 출력되는 것을 확인할 수 있다.

     ![CommitLog](/images/webapp_a18.png)

### 배포 결과 확인

배포가 완료되었으니 정상 동작하는지 결과를 확인해보자.

아래 그림처럼 Azure 포털에서 웹앱의 URL을 클릭한다.

![ClickURL](/images/webapp_a19.png)

웹 브라우저에 아래와 같은 결과가 표시된다.

![WebApp결과](/images/webapp_a20.png)

## 맺음말

보는 바와 같이 Azure 웹앱을 사용하는데는 배포 설정이 주요하다.

배포 설정까지 마무리되면 commit하고 push만해도 계속 웹앱이 업데이되기 되니 편리하다.


다음에는 배포한 웹앱의 로그를 확인하는 방법을 다루도록 하겠다.

---

## 참고자료

* [Azure 앱 서비스에서 Node.js 웹앱 시작](https://azure.microsoft.com/ko-kr/documentation/articles/app-service-web-nodejs-get-started/)