---
layout: post
title:  "이세계에 진입한 게임 서버 - 1"
date:   2016-12-21 00:01:00
categories: nodejs azure webapps unity
comments: true
meta : nodejs azure webapps webservice
description : Azure 웹앱 시작하기
image : 
  twitter: /images/rvwendy_tutorial_01.jpg
  facebook: /images/rvwendy_tutorial_01.jpg
publish : true
---

* content
{:toc}

{% include youtubePlayer.html id="Evwb2jwat6M" %}

## 들어가는 말
레벨제로 카페에 올라오는 신규 게임 소식만 접해도 소규모로 모바일 게임을 만드는 사람이 얼마나 많은지 알 수 있다.

![소규모게임](/images/lvzerogames.png)

그런데 많은 경우 클라이언트만 제작하고 출시해서 운영에 어려움이 있다. 특히 문제가 되는 것은 크랙된 게임이 유통되어 그나마 있던 수익마저 다른 누군가가 가져가는 것이다.

이런 문제를 해결하려면 출시단계부터 게임 서버를 운영하는 것이다. 하지만 게임 클라이언트 만들기도 바쁜데 언제 할 수 있을까?

> 소잃기 전에 한번 배우시길 추천한다.

막상 배우려고 해도 어디서부터 해야할지 막막할 수 있다.

길잡이라도 할 수 있게 소규모 게임 개발자들을 위한 `Azure 웹앱으로 만드는 게임 서버` 강좌를 시작하고자 한다.

## 왜 Azure 웹앱인가?
요리가 왜 어려운지 고민해봤는가? 결혼 전 계란후라이와 라면밖에 끓이지 못하던 사람이었다.
아내따라서 뭔가 만들다보니 이제는 가끔 주말 아침 식사를 만들 수 있는 초보가 되었다.
배우면서 느낀 어려운 점은 이렇다.

### 처음 요리할 때 요리가 힘든이유?
갖추어야할 장비, 재료가 많다.
> 아니 집에 두반장이 있을리가 있나! 레몬 과즙을 숟가락짜면 너무 힘들다!

레시피가 있어도 그대로 따라하기가 어렵다. 해본적이 없으니 `데치세요` 라고 써있는걸 `풀죽`으로 만든다.

이런 점은 프로그래밍도 비슷하다. 알아야할 것이 많고 누군가 방법을 알려줘도 그것을 해석하기 위한 노력이 필요하다.

> 설령 c나 c++ 수업을 들었다고 원하는걸 뚝딱뚝딱 만들 수 있는 수준일리 없다.

### 쉐프에게 레시피만 전달하면 요리를 받을 수 있다면?
그런데 쉐프에게 레시피만 전달하면 요리를 받을 수 있게 된다면 어떨까? 안정적인 요리를 늘 먹을 수 있게된다.

### Azure 웹앱은 준비된 쉐프
Azure 웹앱은 준비된 쉐프다. 레시피만 전달하면 무쇠 냄비나 스텐리스 프라이팬을 관리해주고 재료도 척척 준비한다.
해야할 일이 레시피 작성에서 끝나게 된다.

즉, 인프라 관리는 모두 Azure가 담당하니 소스코드만 만들어서 보내면 되는 것이다.

## Azure 웹앱 만들기

### 사전 준비
바로 Azure를 활용하고 싶지만 가입은 해야한다. 아래 동영상을 참고해도 좋고 인터넷에 자료를 검색해도 좋다.

{% include youtubePlayer.html id="q380IeIsmPc" %}

꼭 살아서 다음 단계에서 만나자!

### 진짜 Azure 웹앱 만들기
처음으로 우리가 할 일은 쉐프를 섭외하는 일이다.

멋진 전문가처럼 커맨드 라인 인터페이스(CLI)를 켜서 작업하고 싶겠지만 우리는 쉽게 이해할 수 있는 Azure 포털을 이용하겠다.

1. [Azure 포털(https://portal.azure.com)로 접속](https://portal.azure.com)한다.

2. `새로 만들기` 클릭한다.

    ![새로만들기](/images/webapp_a01.png)

3. `Web App`으로 검색하거나 `웹 + 모바일`을 선택한 뒤

    ![웹+모바일](/images/webapp_a02.png)

4. `웹앱`을 선택한다.

    ![웹앱](/images/webapp_a03.png)
    
5. `앱이름`을 입력하고 `리소스 그룹`을 생성한다. 입력완료 후 `만들기`를 클릭하여 생성한다.

    ![웹앱입력](/images/rvwendy01_01.jpg)
    
위 과정이 끝나면 웹앱을 확인할 수 있다.

![웹앱확인](/images/rvwendy01_02.jpg){:width="500px"}

`웹앱`이 편리한 것은 생성만으로 접근가능한 URL이 생성되고 ftp를 통해 소스코드를 배포하고 로그를 확인할 수 있도록 설정된다는 점이다.
> 그냥 만들기만하면 친구나 연인(A.K.A. 희귀생명체)에게 링크를 쏴줄 수 있다는 얘기다.

## 소스코드 배포
> 와 생성이 뭐이렇게 쉽나? 라고 생각하면 천만다행이지만 쉬운것도 어렵게 보이도록 만드는 것이 장기인만큼 걱정이 살짝된다.

웹앱이 생성되었으면 이제 소스코드 - 앞서 예를 든 레시피 - 를 배포하여 자신이 원하는 형태로 가공해야한다.

다양한 방법으로 소스코드를 배포할 수 있지만 여기서는 `로컬 Git 레포지토리`를 설정하여 배포하는 방법을 설명할 것이다.

> git을 사용하면 ~~상상속에서~~ 핫하고 섹시해보이까 배워두자.

### 배포 옵션 설정

말이 조금 이상하게 들리지만 배포 옵션은 영문 사이트에서 Deploment Source라는 메뉴이다.

배포 시 소스코드를 어떤 방식으로 관리할지 정하는 것이다.

1. 메뉴 중 `배포 옵션`을 클릭한다.

    ![배포옵션](/images/rvwendy01_03.jpg)
    
2. 배포 옵션 메뉴에서 `소스 선택`클릭한다.

    ![소스선택](/images/rvwendy01_04.jpg)
    
3. 원본 선택 메뉴에서 `로컬 Git 레포지토리`를 선택한다.
    
    ![로컬Git레포지토리](/images/rvwendy01_05.jpg)

4. `기본 인증 설정`을 클릭한 뒤 배포시 사용할 아이디와 패스워드를 입력한 뒤 `확인`을 클릭한다.
    
    ![기본 인증 설정1](/images/rvwendy01_06_1.jpg)

    ![기본 인증 설정1](/images/rvwendy01_06_2.jpg)

5. 입력이 완료되면 `확인`을 클릭한다.
    
    ![확인](/images/rvwendy01_07.jpg)

설정이 끝나면 아래처럼 웹앱 개요 화면에서 `Git 복제 URL`을 확인할 수 있다.

![Git복제URL확인](/images/rvwendy01_08.jpg){:width="500px"}

### 테스트 어플리케이션 배포하기(feat. SourceTree)

git을 다룰 때 CLI로 하면 멋지지만 멋을 조금 포기하면 굉장히 편한 방법이 있다. 클릭클릭으로 git을 다룰 수 있는 SourceTree라는 프로그램으로 말이다.

SourceTree를 설치하고 소스코드를 배포하는 방법을 알아보겠다.

1. [SourceTree](https://www.sourcetreeapp.com/)를 설치한다.

2. 아래 링크를 클릭하여 소스코드를 다운받는다.

    [Wendy-0.0.1 링크](https://github.com/totuworld/Wendy/archive/0.0.1.zip)
    
3. 압축을 해제한다.

4. SourceTree를 실행하여 `New Repository`클릭한 뒤 `Add exist local repository` 메뉴를 선택하낟.

    > Windows에서는 File - Clone / New 선택 후 나오는 창에서 `Add Working Copy`탭으로 이동한다.
    
    ![NewRepository](/images/webapp_a12.png)

5. `Destination Path`에 압축해제한 어플리케이션 폴더를 선택한 후 `Create`를 클릭한다.

    > Windows에서는 Working Copy Path가 같은 역할을 한다.

    ![로컬레포지토리추가](/images/rvwendy01_09.jpg)
    
6. 추가된 레포지토리를 선택한다.

    ![레포지토리열기](/images/rvwendy01_10.jpg)

7. `Settings`를 클릭한다.

    ![설정](/images/rvwendy01_11.jpg)
    
8. Azure 포털에서 웹앱을 선택하고 `Git 복제 URL`를 클릭하여 복사한다.

    ![Git복제URL확인](/images/rvwendy01_08.jpg){:width="500px"}
    
9. SourceTree로 돌아와 `Remote Name`을 azure로 입력하고 `URL`에 복사한 주소를 붙여넣고 `OK`를 클릭한다.

    ![AddAzureRemote](/images/rvwendy01_12.jpg)
    
10. `File Status`탭에서 `Staged files`를 클릭한 뒤 commit 메시지를 `init`으로 입력하고 `Push changes immdiatly to azure/master`옵션을 켜고 `Commit`한다.
    
    ![Commit](/images/rvwendy01_13.jpg)
    
11. Push 메시지에 아래와 같은 로그가 출력되는 것을 확인할 수 있다.

     ![CommitLog](/images/rvwendy01_14.jpg)

## 결과 확인

꽤 오래 걸렸지만 배포가 되었다. 이제 확인해볼 차례다.

아래 그림처럼 Azure 포털에서 웹앱의 URL을 클릭한다.

![ClickURL](/images/rvwendy01_15.jpg)

웹 브라우저에 아래와 같은 결과가 표시된다.

![WebApp결과](/images/rvwendy01_16.png)

## 맺는 말

지금 무슨 일이 일어났는지 어리둥절 할 수 있다. Azure도 생소한데 git에다가 SourceTree 프로그램까지 설치하고 난리난리했으니 그럴 수 있다.

이것도 계속하면 익숙해진다.

꼭 성공해서 다음 강좌를 만나도록하자!

> 첫 걸음을 걸었으니 이제 반을 지났다 :)