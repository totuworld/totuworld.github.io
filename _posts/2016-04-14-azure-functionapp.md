---
layout: post
title:  "Azure Functions로 Slack Bot 만들기"
date:   2016-04-14 00:01:00
categories: nodejs azure
comments: true
meta : nodejs azure functions
description : Azure의 Functions를 활용해 Slack Bot을 만든다. 
publish : true
---

* content
{:toc}

## 들어가는 말

빌드 2016(Build 2016) 키노트에서 [Azure Functions](https://channel9.msdn.com/Events/Build/2016/KEY02#time=44m34s)이 소개되었다.

![Azure Functions](https://msdnshared.blob.core.windows.net/media/2016/04/image162.png)
    
> Azure Function은 C#이나 Node.js를 포함하는 다양한 언어로 개발된 코드를 특정 상황에서 수행하기 위한 기능을 의미합니다.
> 특정 조건에서만 수행할 기능이 필요한 경우에 Azure Function을 사용하면, 가상머신을 구성하여 운영하는 기존의 방식과 비교했을 때 비용을 훨씬 절약하실 수 있습니다.
> 그외에도 Azure IoT Device Management Service, Azure IoT Gateway Device Service를 Public Preview 버전으로 공개합니다.  
> [MS 개발자 블로그 발췌](https://blogs.msdn.microsoft.com/eva/?p=7233)

실행된 만큼만 가격을 지불된다는 것이 매력이다. 그래서 한번 써보기로 했다.

그리고 글을 읽던 중 [AWS Lambda와 API Gateway로 Slack Bot 만들기](http://www.usefulparadigm.com/2016/04/06/creating-a-slack-bot-with-aws-lambda-and-api-gateway/)를 발견했다. 이 내용을 Azure Functions으로 진행 하겠다.


## 슬래시 명령(Slash Command) 만들기

자세한 설명은 [AWS Lambda와 API Gateway로 Slack Bot 만들기#슬래시 명령(Slash Command) 만들기](http://www.usefulparadigm.com/2016/04/06/creating-a-slack-bot-with-aws-lambda-and-api-gateway/#slash-command-)를 참고하면 된다.

여기서는 과정만 일부 발췌하여 사용하도록 한다.

1. 슬랙에서 자신의 계정에 로그인한 다음, [“Apps & Integrations”](https://slack.com/apps/build) 메뉴로 들어가 “Make a Custom Integration” 창에서 새 슬래시 명령을 만든다. 

2. 검색어를 [위키피디어](https://ko.wikipedia.org/)와 연결하는 /wiki 라는 명령을 하나 추가해 보기로 하겠다.

    ![wiki슬래시명령](https://usefulpa.s3.amazonaws.com/images/2016/create-new-slash-command.png)

3. 이어 나오는 페이지에서는 슬래시 명령에 대응하는 URL을 적어주면 된다. 아직 우리는 웹서버를 만들지 않았기 때문에 일단 이 부분은 비워 두자. 나중에 채울 것이다.

    ![슬래시명령셋팅](https://usefulpa.s3.amazonaws.com/images/2016/slash-commands-settings.png)
    
## 슬랙 봇 호스팅하기

> 이제 앞서 비워둔 URL 칸을 채워 보자. 이를 위해서는 어딘가에 웹서버를 하나 만들어 두고 슬랙에서 넘어오는 슬래시 명령을 받아서 슬랙 메시지 형태로 반환하는 코드를 작성해야 한다.

이제 Azure Functions을 발동시킬 때다!

### Azure Functions 추가

1. [Azure portal로 접속](https://portal.azure.com)해서 Functions를 생성한다(포털에서는 Function App으로 나온다).
    
    ![Functions추가](/images/azurefunctoins.png){:width="500px"}

2. 생성된 Function App을 클릭한 후 `+ New Function`를 클릭한다.

    ![신규Function추가](/images/azurefunctoins1.png)

3. Language를 Javascript로 변경하고(`1`) HttpTrigger - Node 템플릿을 선택(`2`)한 후 Function 이름을 입력(`3`)한다(이하 SlackWiki로 칭한다).
 
    ![HttpTrigger생성](/images/azurefunctoins2.png){:width="500px"}
    
4. SlackWiki의 Develop 탭 - Code 부분에 다음 소스코드를 입력한다.

{% gist totuworld/9b7c39cc78d70df9cea28a1217c875e6 index.js %}


### 슬랫 슬래시 명령 URL 입력

Azure Functions에서 HttpTrigger는 생성하면 `Function Url`을 제공한다.

![FunctionURL](/images/azurefunctions4.png)

위 URL을 비워둔 URL 칸에 넣으면 된다.

![addFunctionURL](/images/azurefunctions5.png)

## Hello, 슬랙 봇!

> 이제 이 모든 작업이 끝났으면 Slack으로 돌아오자. 슬랙의 대화창에서 다음과 같이 방금 우리가 만든 슬래시 명령을 입력하면 Slack 봇이 반갑게 우리를 맞아 줄 것이다.

![SlackBot결과](/images/slackbot.png)


## 맺음말

사용해본 결과 Azure Functions는 AWS Lambda + API gateway를 하는 것보다 간단히 사용할 수 있는 장점이 있었다.

뿐만 아니라 Azure의 Web App을 다뤄봤다면 배포 설정을 git으로 변경하는 부분도 같은 경험을 제공해서 편리했다.

여기서 다뤘던 HttpTrigger 외에도 TimerTrigger로 cron 작업 돌리는 것처럼 일정 주기로 작업을 돌릴 수 있다.


단점도 존재한다. 가장 난해한 부분은 Node.js의 package를 Web App처럼 `package.json`을 해석하는 것이었다. 

바로 해석해서 설치하는 것까진 좋은데 폴더 내부에 `package.json` 을 넣어서 폴더별로 설치되진 않았다. 이를 위해서 굳이 console에 들어가야하는 불편함이 존재했다.

~~(이에 관해서는 MS에 문의했으나 아직 답변을 받진 못했다.)~~

> MS에서 다음과 같이 답변이 왔습니다(2016. 4. 15)

> 최상위 폴더에 package.json 파일이 존재해야 하고 하위 디렉토리에는 각각의 function.json 파일이 있어야 합니다.


하지만 아직 preview 단계이니 앞으로 발전 사항을 지켜볼만하다. 정말 간단한 로직을 위해서 Web App이나 VM을 올리는 수고를 덜 수 있으니 한번 사용해보길 바란다.

## 참고자료

* [AWS Lambda와 API Gateway로 Slack Bot 만들기](http://www.usefulparadigm.com/2016/04/06/creating-a-slack-bot-with-aws-lambda-and-api-gateway/)
* [Azure Functions NodeJS developer reference](https://azure.microsoft.com/ko-kr/documentation/articles/functions-reference-node/)