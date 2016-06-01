---
layout: post
title:  "Azure 웹앱 로그조회"
date:   2016-06-01 00:01:00
categories: nodejs azure webapps
comments: true
meta : nodejs azure webapps webservice
description : Azure 앱 서비스인 웹앱의 로그를 조회하는 방법을 알아본다.  
image : 
  twitter: /images/webapp_tutorial_title_02.png
  facebook: /images/webapp_tutorial_title_02.png
publish : true
---

* content
{:toc}

## 들어가는 말

테스트코드를 돌리고 나온 모든 버그를 잡았다.

하지만 배포 후 버그가 튀어나온다.

로그가 보고 싶다 :(

![버그놈](/images/bug.png){:width="300px"}


이때 도움이 되는 기능이 뭐 없을까?

Azure 웹앱은 디버그에 도움되는 기본 기능을 제공한다. 그 중 이번에는 로그를 조회하는 방법을 알아보겠다.

## 진단 로그 설정

웹앱에서 로그를 조회하려면 먼저 로그가 기록되도록 해야한다.

웹앱을 생성하고 소스코드를 배포했다고 로그가 어딘가에 기록되는 것이 아니다.

어떤 방식으로 어느 수준의 로그를 축적할지 설정이 필요하다.

Azure에서 이를 설정하는 메뉴를 `진단 로그`(Diagnostics logs)로 부른다.

> 이름만 들어서는 이게 뭔가 싶을 수 있다.

설정법은 간단하다.

1. [Azure 포털로 접속](https://portal.azure.com)한다.

2. 웹앱을 선택하여 `설정`을 클릭한다.

    ![웹앱설정](/images/webapp_a06.png)
    
3. 기능 아래에 `진단 로그`를 클릭한다.

    ![진단로그](/images/webapp_b01.png){:width="300px"}

여기까지 진행하면 아래와 같은 `진단 로그` 메뉴가 나타난다.

![로그](/images/webapp_b02.png){:width="320px"}

크게 2가지 구역으로 나뉘는데 a 구역은 웹 응용 프로그램(즉 자신의 코드)가 만들어낸(혹은 수행중) 정보를 캡처한다. 이를 파일 시스템으로 저장할지 blob으로 저장할지 결정한다.

그 중 응용 프로그램 로깅(파일 시스템)은 설정 후 12시간 뒤에 해제된다. 로그를 오래도록 저장하고자하면 blob을 이용하자.

b 구역은 웹 서버의 로그를 제공하는 옵션이다. 이를테면 `자세한 오류 로깅`은 오류 상태를 나타내는 HTTP 상태 코드(400 이상)의 자세한 오류 정보를 기록한다.

> 각 기능별 세부 사항은 [Azure 앱 서비스에서 웹 앱에 대한 진단 로깅 설정](https://azure.microsoft.com/ko-kr/documentation/articles/web-sites-enable-diagnostic-log/)의 설명을 참고바란다.


여기서는 코드에 대한 로그를 살펴볼 예정이므로 아래와 같이 `응용 프로그램 로깅(파일 시스템)`만 설정하고 `수준`(Level)을 정보로 선택한 뒤 저장한다.

수준은 총 4가지 분류(오류, 경고, 정보, 세부 정보 표시)로 나뉜다. 수준별로 필터링하여 로그를 저장한다.

![로그설정](/images/webapp_b03.png){:width="320px"}

이로써 진단 로그 설정이 완료되었다.


> 아래 내용을 보기 전에 FTP나 CLI가 싫고 그냥 Azure 포털에서 `하!고!싶!다!`고 생각하시면 [기승전 Kudu](#kudu)로 넘어가면 된다.


## 로그 다운받기

위 설정을 통해 웹앱이 작동하면서 로그를 쌓고있다. 이를 조회하려면 로그를 다운받거나 실시간으로 스트리밍하면 된다.

2가지 방법 중 먼저 다운로드 받는 방법을 알아보자.

> 다운로드는 FTP, Azure CLI, PowerShell로 가능하지만 여기서는 FTP, Azure CLI만 커버한다. 

### FTP로 로그 다운받기

Azure 웹앱은 생성과 함께 ftp로 접근가능하다. 다들 ftp 사용법을 알겠지만 혹시나 하는 마음으로 다뤄본다.

1. [FileZilla](https://filezilla-project.org/)같은 ftp 프로그램을 다운받아 설치한다.

    > 여기서는 FileZilla로 설명하겠지만 다들 비슷할 것이다.
    
2. [Azure 포털로 접속](https://portal.azure.com)한다.

3. 웹앱을 선택하여 `게시 프로필 가져오기`를 클릭하여 게시 프로필(파일 확장자가 .PublishSettings 인 파일)을 다운받는다.
    
    ![게시 프로필](/images/webapp_b04.png)
    
4. 다운받은 게시 프로필을 [Visual Studio Code](https://code.visualstudio.com/)같은 텍스트 에디터로 확인해보면 XML 형식으로 작성된 파일 내용을 열람할 수 있다.

    그중에서 아래 그림처럼 publishMethod가 ftp인 부분을 살펴보면 ftp 접속 시 사용할 URL, username, password를 확인할 수 있다.
    
    한가지 유의해야할 점은 URL에서 뒤쪽의 상세경로는 제거하고 입력하는 것이다. 전체경로를 입력하면 소소크드가 동작중인 폴더로 이동하기 때문이다.

    ![게시 프로필확인](/images/webapp_b05.png){:width="540px"}

5. FileZilla를 실행하고 아래 표와 그림을 참조하여 URL, username, password를 입력하고 `Quickconnect`클릭하여 접속한다.

    FileZilla 입력창 | 게시 프로필의 명칭
    --- | ---
    Host | publishUrl
    Username | userName
    Password | userPWD
    
    ![FileZilla 접속](/images/webapp_b06.png)

6. FileZilla 우측 Remote site에 폴더 목록이 나타난다. 이 중에서 `LogFiles`에 들어가본다.

    ![remotesite](/images/webapp_b07.png){:width="300px"}
    
7. LogFiles 안에는 여러 폴더가 있을 수 잇다. 그중에서 앞서 진단 로그에서 설정한 응용 프로그램 로깅은 `Application`폴더에 저장된다. txt 형식의 파일을 다운받는다.

   > FileZilla에서 좌측 Local site로 드래그하거나 우클릭하여 Download를 실행하면 파일을 다운받을 수 있다.
   
   ![Application폴더](/images/webapp_b08.png){:width="300px"}
   
다운 받은 파일을 확인해보면 아래와 비슷한 형식의 내용을 볼 수 있다.
    
```
[0mGET / [32m200 [0m1.863 ms - 191[0m
[0mGET / [32m200 [0m2.008 ms - 191[0m
```

Azure 포털에서 웹앱의 URL을 여러번 클릭하거나 혹은 브라우저를 여러번 갱신하면 로그 파일의 크기가 점점 커지는 것을 확인할 수 있다.

### Azure CLI 로그인

Azure CLI는 개발, 배포, 관리를 목적으로 제작된 크로스 플랫폼(Windows, Mac, Linux) CLI이다.

커맨드 라인 인터페이스에 익숙하다면 이 내용이 훨씬 간단할 것이다.

Azure CLI를 사용하기 앞서 내가 누구인지 증명해야하니 로그인을 시작해보자.

1. [Azure CLI 설치](https://azure.microsoft.com/ko-kr/documentation/articles/xplat-cli-install/) 문서를 참고해서 Azure CLI를 설치한다.

    node.js를 활용하고 있지 않다면 설치 관리자를 사용해서 직접 다운받으면 된다.
    
2. 자신이 익숙한 터미널을 열고 다음 명령을 입력하여 Azure CLI에 권한을 획득하도록 한다.

    {% gist /1de28ccc11ecc2017b5912467aacdc8f login %}
    
3. 위 내용처럼 나왔다면 브라우저를 열어 [https://aka.ms/devicelogin](https://aka.ms/devicelogin)에 접속한 뒤 `code` 뒤에 9자리 code를 입력하고 `계속` 버튼이 나타나면 클릭한다.

    ![AzureCLI로그인](/images/webapp_b09.png)
    
4. 로그인 창이 나오면 자신의 아이디로 로그인한다.
    
    ![AzureCLI로그인](/images/webapp_b10.png)

과정이 완료되면 아래와 같은 메시지가 나온다.

![AzureCLI로그인완료](/images/webapp_b11.png)

브라우저 창을 닫으면 터미널에도 완료메시지가 나타난다

{% gist /1de28ccc11ecc2017b5912467aacdc8f logindone %}

> 기능 하나를 사용할 때마다 로그인을 해야하는 것은 아니지만 컴퓨터를 재부팅한 후 Azure CLI를 사용할 일이 발생하면 로그인이 필요하다.  


### Azure CLI로 로그 다운받기

로그인 되었으니 asm(Azure Service Management) 모드로 변경하고 다운받으면 된다.

> $ 부분이 명령어 이다. 그리고 `[웹앱이름]`이라고 적힌 부분은 반드시 자신의 웹앱 이름으로 대치해야 결과가 나온다.

{% gist /1de28ccc11ecc2017b5912467aacdc8f logdownload %}

결과를 보면 알겠지만 바로 diagnostics.zip 파일이 다운로드 된다. 압축을 해제하면 LogFiles란 폴더가 존재한다. 이 폴더의 구조는 FTP에서 살펴본것과 동일하다.


## 실시간 로그 조회하기

앞서 로그를 다운 받는 방법을 알아봤다. 내 경우는 개인 프로젝트라서 그런지 다운받아 로그를 살펴보기보단 어떤 로그가 남는지 실시간으로 출력되는 로그를 살펴보고 문제를 해결하는 경우가 많다.

웹앱도 스트림 로그를 지원하므로 실시간으로 로그를 조회할 수 있다.

Visual Studio Application Insights, PowerShell 등으로 가능하지만 여기서는 Azure CLI 내용만 커버하도록 한다.

> 어디까지나 내가 OSX 사용자이며 Visual Studio는 Unity용 MonoDevelop이 만족스럽지 않아 Visual Studio를 사용해본게 전부여서 그런것이다(앱등앱등).

이미 Azure CLI 로그인이 완료되었다면 아래 명령을 입력해보자.

> `[웹앱이름]`이라고 적힌 부분은 반드시 자신의 웹앱 이름으로 대치해야한다.

{% gist /1de28ccc11ecc2017b5912467aacdc8f streamlog %}

Azure 포털에서 웹앱의 URL을 여러번 클릭하거나 혹은 브라우저를 여러번 갱신하면 터미털에 로그가 계속 들어오는 것을 확인할 수 있다.


## 기승전 Kudu

지금까지 FTP나 Azure CLI를 이용해서 로그를 다운받고 실시간으로 조회도 했다.

하지만 이런 모든 과정이 귀찮고 브라우저에서 이런것을 처리해도 하등 문제가 없다고 생각한다면 Azure 포털에서 이 모두를 처리할 수 있다.

그 비밀의 이름은 Kudu이다.

![Kudu](/images/webapp_b13.png)

> 난~ Azure CLI 성애자니깐~~ ㅋㅋㅋ

1. [Azure 포털로 접속](https://portal.azure.com)한다.

2. 웹앱에서 `도구`를 클릭한다.

    ![도구](/images/webapp_b12.png)
    
3. 도구 메뉴에서 `Kudu`를 클릭하고 `이동`을 클릭하면 브라우저에 탭이 새로 생성되면서 Kudu를 영접할 수 있다.

    ![Kudu열기](/images/webapp_b14.png){:width="500px"}
    
4. Kudu 메인 화면에서 `Debug Console`을 클릭하여 하위 메뉴에서 `CMD`를 선택한다.

    ![DebugConsole](/images/webapp_b15.png){:width="500px"}
    
    ![CMD](/images/webapp_b16.png){:width="200px"}
    
자 이제 아래 화면이 나타난다.

![LogFiles폴더](/images/webapp_b17.png){:width="540px"}

화면만 봐도 느낌이 온다. 그렇다. LogFiles 폴더로 진입하면 FTP에서 봤던 폴더 구조가 펼쳐지고 그 안에 로그 파일을 다운받을 수 있다.

심지어 폴더를 이동하면 하단의 console 창이 알아서 폴더를 이동한다.

실시간 로그를 보고 싶다면 어떻게 해야할까?

아래 처럼 리눅스의 tail 명령을 사용하면 된다(파일명은 반드시 자신의 로그 파일로 수정해야한다).

> 파일명은 모두 타이핑하지 말고 앞쪽 몇글자를 타이핑하고 tab 버튼을 누르면 자동완성된다.

{% gist /1de28ccc11ecc2017b5912467aacdc8f kudutail %}

뭔가 허무한가? 아니다. Azure CLI를 따라해봤다면 적어도 Azure CLI를 설치하지 않았는가!!

그거면 됐다(왠지 도망가야할 듯한 느낌인데).


## 맺음말

다루면 다룰수록 Azure는 사용자가 뭔소리인지 알지못하게 기능을 잘 감춰두었다. 찾아서 쓰면 편리한 것이 많다.

옛말에 `필요는 발명의 어머니`라고 했다. Azure에 적용하면 `필요는 삽질의 어머니`쯤 될듯하다.

> 그래도 너 이 Azure 사..사...좋아한다.


다음에는 웹앱 튜토리얼 글감의 어머니(?) chiyodad님이 필요로하는 웹앱에서 SQL 데이터베이스 사용하는 방법을 다루도록 하겠다.

---

## 참고자료

* [Azure 앱 서비스에서 웹 앱에 대한 진단 로깅 설정](https://azure.microsoft.com/ko-kr/documentation/articles/web-sites-enable-diagnostic-log/)
* [Azure CLI 설치](https://azure.microsoft.com/ko-kr/documentation/articles/xplat-cli-install/)