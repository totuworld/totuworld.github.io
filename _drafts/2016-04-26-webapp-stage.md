---
layout: post
title:  "Azure 웹 앱 staging 환경 설정하기"
date:   2016-04-26 00:02:00
categories: nodejs azure
comments: true
meta : nodejs azure webapps
description : 배포슬롯을 사용해서 staging 환경 설정하여 배포되는 동안 서비스가 중단되지 않도록한다.
image : azure-deployment-slots.png
publish : true
---

* content
{:toc}

## 들어가는 말

웹 앱(Web Apps)를 사용하여 서비스를 운영하면 배포될 때 서비스가 중단되는 불편함이 있다.

아래 이미지에서 보는 것처럼 배포가 될 때마다 회색영역처럼 응답이 없는 시점이 발생한다.

![배포공백](/images/webappdeployslot01.png)

중단되어도 무관한 서비스라면 상관없지만 무중단 서비스를 제공하는 경우는 어떻게 해야할까? 

이제부터 알아보자.


## 배포슬롯

![배포슬롯](/images/azure-deployment-slots.png)

앞서했던 질문에 답으로 Azure가 제공하는 기능은 `배포슬롯`이다.

> 배포슬롯은 표준(Standard)나 프리미엄(Premium)이상의 웹 앱에서만 가능하다.

원리는 1개의 웹 앱에 다수의 배포슬롯을 생성하여 배포하여 검증한 것으로 production에 반영하는 것이다.

예를 들어 아래 그림처럼 a, b 슬롯이 있다고 해보자. 

![배포공백해결책](/images/webappdeployslot02.png)

b 슬롯이 production에 사용중이라면 a 슬롯에 v0.0.2a를 배포한다. a 슬롯은 배포중이지만 b 슬롯이 production이므로 서비스는 중단되지 않는다.

그리고 a 슬롯의 배포가 완료되면 a 슬롯과 b슬롯을 교환한다.

이렇게 배포하면 그림의 녹색영역처럼 서비스는 언제나 작동하게 된다. 


### 웹 앱에 staging 배포슬롯 추가

1. [Azure portal로 접속](https://portal.azure.com)에서 대상이 될 웹 앱을 선택한다.

2. 설정을 클릭한 다음 배포슬롯을 선택한다.

    ![설정](/images/webappds01.png) ![배포슬롯](/images/webappds02.png)

3. 배포슬롯에서 추가를 선택하고 `이름`과 `구성소스`를 선택한 후 추가한다.

    ![배포슬롯추가](/images/webappds03.png) ![배포슬롯추가1](/images/webappds04.png)

슬롯이 추가가된 후 슬롯을 클릭하면 웹 앱과 동일한 타일이 나타난다. 해당 타일에서 연속배포는 별도로 설정할 수 있다.

> 이 글에서는 staging과 production을 모두 로컬 git레포지토리로 연속배포하도록 설정하여 사용한다.

## 웹 앱 배포

Azure 웹 앱에 배포할 소스코드가 필요한데 여기서는 [Azure 컨테이너 서비스로 Node.js 앱 배포하기](http://totuworld.github.io/2016/04/25/acs/)에서 사용한 Node.js 웹 어플리케이션을 사용하겠다.

> 자신이 제작한 서비스로 진행해도 무방하다.

### 프로덕션 배포 설정

아래 명령을 순서대로 입력하여 Node.js 웹 어플리케이션을 다운받는다.

{% highlight shell %}

$ wget https://github.com/totuworld/NodejsExampleApp/archive/0.0.1.zip
$ unzip 0.0.1.zip
$ cd NodejsExampleApp-0.0.1

{% endhighlight %}

그리고 git을 설정한 후 모든 소스코드를 포함시켜서 커밋한다. 

{% highlight shell %}

$ git init
$ git add *
$ git commit -m "init"

{% endhighlight %}

프로덕션 배포 레포지토리를 추가한다.

> 아래 명령에서 `레포지토리주소`를 자신의 주소로 변경하여 입력한다.

{% highlight shell %}

$ git remote add azureproduction [레포지토리주소] 

{% endhighlight %}

이제 azureproduction remote로 배포한다.

{% highlight shell %}

$ git push azureproduction master
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '6381c66539'.
remote: Generating deployment script.
...중략...
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Deployment successful.

{% endhighlight %}

배포가 완료되고 브라우저에 웹 앱의 프로덕션 URL을 입력하면 아래와 같은 결과를 확인할 수 있다.

> 글에서 사용된 주소 : http://mktestyo.azurewebsites.net/ 

![ACSAppResult](/images/ACSAppResult.png)

## 코드 수정

routes폴더의 `index.js`파일의 6번째 줄을 아래처럼 수정한다.

{% highlight javascript %}

  res.render('index', { title: 'Web Apps' });

{% endhighlight %}

그리고 새로운 커밋을 작성한다.

{% highlight shell %}

$ git add *
$ git commit -m "update index.js"

{% endhighlight %}

이제 staging 배포 레포지토리를 추가한다.

{% highlight shell %}

$ git remote add azurestaging [레포지토리주소] 

{% endhighlight %}

이제 수정한 코드를 azurestaging remote로 배포한다.

{% highlight shell %}

$ git push azurestaging master
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '6381c66539'.
remote: Generating deployment script.
...중략...
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Deployment successful.

{% endhighlight %}

배포가 완료되고 브라우저에 staging URL을 입력하면 아래와 같은 결과를 확인할 수 있다.

> 글에서 사용된 주소 : http://mktestyo-staging.azurewebsites.net/

![webappsresult](/images/webappsresult.png)


## staging과 production 교환

배포가 완료되어 작동하는 staging을 production과 교환해보자. 

1. [Azure portal로 접속](https://portal.azure.com)에서 대상이 될 웹 앱을 선택한다.

2. 설정을 클릭한 다음 배포슬롯을 선택한다.

    ![설정](/images/webappds01.png) ![배포슬롯](/images/webappds02.png)
    
3. 교환을 선택한다. 

    ![배포슬롯교환](/images/webappds05.png)

4. 교환 내용을 확인하고 확인 버튼을 클릭한다.

    ![배포슬롯교환1](/images/webappds06.png)
    
교환이 되는 동안에는 아래와 같은 메시지가 나온다.

![배포슬롯교환중](/images/webappds07.png)

교환이 완료되고 브라우저에 웹 앱의 프로덕션 URL을 입력하면 다음 결과를 확인할 수 있다.

![webappsresult](/images/webappsresult.png)

이런 배포가 익숙해진다면 Azure에서는 `자동 교환`을 설정하여 편리하게 사용할 수 있다.


## 테스트

배포슬롯이 어떤 방식으로 작동하는지 알아봤다. 이제 배포슬롯을 교환하는 동안에 서비스가 이상없이 작동하는지 테스트해보자.

> 테스트에는 아래 코드를 활용했다.

{% gist f68a6447f75117290de47cb9672c114a httpRequestTest.js %}

위 코드는 1초 주기로 특정 URL에 요청을 보내서 응답을 확인한다. 이 코드를 활용해서 테스트용 웹 앱이 배포슬롯을 교환할 때도 잘 작동하는지 알아보겠다.

{% include youtubePlayer.html id="92eMExt7Tu8" %}

> 바쁘신 분들은 3분 15초부터

위 동영상은 화면이 고르지못해 날려버린 14초와 추가한 파란색 박스를 제외하고 어떤 편집도 이뤄지지않았다.

(길고 지루하지만 보시바와 같이) 교환이 일어나는 시점에도 응답이 고르다.

![올ㅋ](https://attachment.namuwikiusercontent.com/올ㅋ__hhjallk.jpg)


## 맺음말

개인적으로 제작하고 있는 서비스가 무중단이 필요해 Azure 측에 문의했더니 참고자료로 사용한 문서를 안내해주었다.

문서를 바탕으로 작동하는 방법을 익히고 동영상 녹화까지해서 테스트 해보니 문제없이 서비스가 작동할 것이란 믿음이 생겼다.

사용법도 그다지 어렵지 않았다. 롤백도 간단하게 가능하다. Azure 웹 앱을 사용하고 있다면 배포슬롯 사용해보길 권한다.

---

## 참고자료

* [Azure 앱 서비스에서 웹 앱에 대한 스테이징 환경 설정](https://azure.microsoft.com/ko-kr/documentation/articles/web-sites-staged-publishing/)