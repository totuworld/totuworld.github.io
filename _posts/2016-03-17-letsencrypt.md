---
layout: post
title:  "Azure Web Apps에 커스텀도메인 설정하기(feat.LetsEncrypt)"
date:   2016-03-17 01:00:00
categories: etc encrypt azure
comments: true
meta : Auzre WebApps ssl letsencrypt
description : Web Apps에 커스텀도메인과 SSL 인증서를 적용한다.
publish : true
---

* content
{:toc}

![titeimage](/images/castles-616573_960_720.jpg){:width="400px"}

## 들어가는 말

Azure에서 Web Apps(웹앱)을 사용해서 Node.js로 만든 웹 서비스를 등록했다. 이를 도메인과 연결하고 HTTPS를 사용해보고 싶었다.

아주 간단한 건줄알고 시작했는데 지식이 없어서 꽤나 오래 걸렸다(Azure의 한없이 불친절한 메뉴얼도 한몫했다).

부디 이글을 보고 있다면 손쉽게 이겨내길 바란다.

## Web Apps

### Web Apps 생성

[Azure portal로 접속](https://portal.azure.com)해서 Azure 서비스에서 웹앱을 생성할 때 혹은 생성 후 Custom domains(커스텀도메인)과 SSL을 설정할 수 있는 tier로 설정한다.

![S1](/images/webapp_s1.png)

    이하 웹앱은 `yotest`로 명명하겠다.

### 배포 설정

배포는 여러방법으로 가능하지만 여기서는 Local Git Repository로 설정한다.

1. [Azure portal](https://portal.azure.com)에서 yotest 웹앱을 선택한 후 설정(Settings)에 들어간다. 아래쪽에 PUBLISHING에 Deployment Source 메뉴를 클릭한다.

    ![배포설정1](/images/webappdeploy1.png)
    
2. 필수구성 요소를 클릭하고 Local git Repository를 선택한다.
    
    ![배포설정2](/images/webappdeploy2.png)
    
3. yotest 웹앱 설정창에서 git repository 주소와 URL를 함께 확인한다.

    ![git 주소](/images/webappdeploy3.png)
    
        git 주소 https://totuworld@yotest.scm.azurewebsites.net:443/yotest.git
        url yotest.azurewebsites.net

### 커스텀도메인 설정

1. [Azure portal](https://portal.azure.com)에서 yotest 웹앱을 선택한 후 설정(Settings)에 들어간다. 아래쪽으로 살펴보면 ROUTING에 Custom domains and SSL 메뉴를 클릭한다.

    ![커스텀도메인 메뉴](/images/customdomains1.png)

2. 커스텀도메인 메뉴에서 Bring External Domains 버튼을 클릭한다.

    ![외부도메인가져오기](/images/customdomains2.png)
    
3. 아래 그림 2번 위치에서 yotest 웹앱의 IP 주소를 확인한다(여기서는 40.7.1.1로 예를 든다).

    ![웹앱IP확인](/images/customdomains3.png)
    
4. 자신의 도메인에서 A 레코드와 CNAME 레코드를 앞서 확인한 정보(40.7.1.1과 yotest.azurewebsites.net)바탕으로 각각 등록한다(반드시 자신의 설정값과 동일하게 입력하여야한다).

    Type | Name | Value
    --- | --- | ---
    A | @ | 40.7.1.1
    CNAME | www | yotest.azurewebsites.net

    변경된 설정이 반영되려면 20분정도 걸렸다. 차를 마시고 오자(적용 여부를 확인하는 것은 [http://www.digwebinterface.com](http://www.digwebinterface.com)에서 확인하자).
    
        보통은 위와 같이 연결하나 필자의 경우 www 대신에 yotest란 서브 도메인을 사용했기때문에 CNAME의 Name이 yotest로 하였다. 

5. 이제 아래 그림 1번 위치에 구입한 도메인을 입력한다(아래에서는 yotest.magmakick.io라고 예를 든다).

    ![도메인입력](/images/customdomains3.png)
    
    설정이 정상적으로 진행되었으면 상단의 SAVE 버튼을 눌러 반영한다.
    
    이제 yotest.magmakick.io로 접속하면 Azure의 yotest 웹앱으로 이동한다.
    
## SSL 인증서 얻기

yotest 웹앱의 설정이 모두 마쳐진것은 아니다. 실제로 사용할 인증서가 없기 때문이다.

SSL을 구매해서 사용해도 되지만 여기서는 [Let's Encrypt](https://letsencrypt.org)를 사용해서 무료로 SSL을 얻도록 하겠다.

### SSL 인증서 발급

mac에서 진행행하다가 어려움을 겪어서 vagrant로 ubuntu 14.04 lts하나 띄우고 진행했음을 밝힌다.

참고로 이 과정은 [Lets' Encrypt로 무료로 HTTPS 지원하기](https://blog.outsider.ne.kr/1178)글을 참조했다. 더 자세히 알고 싶다면 해당 블로그를 방문해보길 권한다.
    
1. 클라이언트를 클론받는다.

        $ git clone https://github.com/letsencrypt/letsencrypt
        $ cd letsencrypt

2. 관련 의존성을 해결하기 위해서 아래 명령을 입력한다.

        $ ./letsencrypt-auto --help
        
    생각보다 시간이 걸린다.

3. 인증서 발급을 위해서 아래 명령을 입력한다. -d 뒤에는 도메인을 --email 뒤에는 자신의 email을 입력한다.

        $ ./letsencrypt-auto certonly --manual -d yotest.magmakick.io --email totuworld@gmail.com
        
    다수의 도메인을 활용하려면 -d 도메인 -d 도메인 형태로 추가하면 된다.
    
    ![ip로그확인](/images/encryptip.png)
    
    위와 같은 이미지가 나오면 Yes를 선택한다.
        
4. 3번 과정을 통하면 아래와 같은 결과가 나온다.

        http://yotest.magmakick.io/.well-known/acme-challenge/hjcbhLr2TTcbTywrWesONhOoiNbKi3GpyUYksHDFRjA before continuing:

        hjcbhLr2TTcbTywrWesONhOoiNbKi3GpyUYksHDFRjA.IBkQXVLbjkDk12yEvZSRNwSrUgZx3j23HgqTrqwP1KU

        If you don't have HTTP server configured, you can run the following
        command on the target server (as root):

        mkdir -p /tmp/letsencrypt/public_html/.well-known/acme-challenge
        cd /tmp/letsencrypt/public_html
        printf "%s" hjcbhLr2TTcbTywrWesONhOoiNbKi3GpyUYksHDFRjA.IBkQXVLbjkDk12yEvZSRNwSrUgZx3j23HgqTrqwP1KU > .well-known/acme-challenge/hjcbhLr2TTcbTywrWesONhOoiNbKi3GpyUYksHDFRjA
        # run only once per server:
        $(command -v python2 || command -v python2.7 || command -v python2.6) -c \
        "import BaseHTTPServer, SimpleHTTPServer; \
        s = BaseHTTPServer.HTTPServer(('', 80), SimpleHTTPServer.SimpleHTTPRequestHandler); \
        s.serve_forever()"
        Press ENTER to continue

    엔터를 바로 누르지말고 일단 내용을 해석해보자.
    
    브라우저로 아래 주소에 접속한다.
    
        http://yotest.magmakick.io/.well-known/acme-challenge/hjcbhLr2TTcbTywrWesONhOoiNbKi3GpyUYksHDFRjA 
    
    다음 값이 반환되어야한다.
    
        hjcbhLr2TTcbTywrWesONhOoiNbKi3GpyUYksHDFRjA.IBkQXVLbjkDk12yEvZSRNwSrUgZx3j23HgqTrqwP1KU
        
    이를 위해서 아래 과정을 진행한다.

### 인증서 발급 마무리

위에서 살펴본 것처럼 특정 주소지로 접속했을 때 값을 반환하기 위해서 다음 과정을 진행한다.

    단계가 여러개지만 결론은 아래 값을 반환하는 것이므로 자신이 편한 방법으로 진행해도 무방하다.
    
    hjcbhLr2TTcbTywrWesONhOoiNbKi3GpyUYksHDFRjA.IBkQXVLbjkDk12yEvZSRNwSrUgZx3j23HgqTrqwP1KU

1. 아래 프로젝트를 클론 받는다(클론은 vagrant 환경에서 받지 않아도 된다).

        $ git clone https://github.com/totuworld/returnserver.git
        $ cd returnserver
    
2. `server.js` 파일에서 `targetPath`와 `returnValue`를 변경한다.

    위의 결과대로 받아야하기때문에 여기서는 다음과 같이 수정했다.

        let targetPath = '/.well-known/acme-challenge/hjcbhLr2TTcbTywrWesONhOoiNbKi3GpyUYksHDFRjA';
        let returnValue = 'hjcbhLr2TTcbTywrWesONhOoiNbKi3GpyUYksHDFRjA.IBkQXVLbjkDk12yEvZSRNwSrUgZx3j23HgqTrqwP1KU';
    
3. 이제 Azure yotest 웹앱의 git 레포지토리와 연결한다(azure 뒤쪽은 자신의 git 레포지토리를 입력한다).

        $ git remote add azure https://totuworld@yotest.scm.azurewebsites.net:443/yotest.git

4. 커밋을 하나 만들고 azure 리모트로 push한다.

        $ git add *
        $ git commit -m 'update targetpath&returnValue'
        $ git push azure master

5. ubuntu 환경에서 enter를 입력한다.

        IMPORTANT NOTES:
        - Congratulations! Your certificate and chain have been saved at
        /etc/letsencrypt/live/yotest.magmakick.io/fullchain.pem. Your
        cert will expire on 2016-06-15. To obtain a new version of the
        certificate in the future, simply run Let's Encrypt again.
        - If you like Let's Encrypt, please consider supporting our work by:

        Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
        Donating to EFF:                    https://eff.org/donate-le
        
    위에 보는 것처럼 SSL 인증서가 생성되었다.
    
## SSL 인증서 사용

인증서를 바로 사용할 수 있으면 좋으련만 Azure 웹앱은 PFX 파일 형태로 인증서를 업로드할 수 있어서 인증서 변경을 해야한다.

### PFX 확장자 인증서 만들기

1. ubuntu 환경에서 다음 명령을 입력하여 root 권한을 얻는다.
        
        $ sudo su

2. 해당 폴더로 이동한다(자신의 도메인 명과 뒤쪽 폴더명이 같이 해야한다). 그리고 변경 명령을 입력한다.

        $ cd /etc/letsencrypt/live/yotest.magmakick.io/
        $ openssl pkcs12 -export -out cert.pfx -inkey privkey.pem -in cert.pem
        
    비밀번호하도록 나오면 암호를 입력하고 기억해둔다.
    
3. 파일을 확인한다.

        $ ls
        cert.pem  cert.pfx  chain.pem  fullchain.pem  privkey.pem
    
    `cert.pfx`가 생성된 것을 확인할 수 있다. vagrant 환경이라면 ftp 등을 이용해서 cert.pfx 파일을 복사한다.

PFX 확장자의 인증서를 얻었으니 이제 Azure 포털에서 등록하면 된다.

### 인증서 적용하기

Azure 포털에서 웹앱에 SSL 인증서를 등록하고 커스텀도메인과 연결하는 방법을 알아보자.

1. [Azure portal로 접속](https://portal.azure.com) yotest 웹앱 설정 - Custom domains and SSL 메뉴를 클릭한다.

    ![커스텀도메인 메뉴](/images/customdomains1.png)

2. Upload Certificates 메뉴를 클릭한다.

    ![인증서업로드1](/images/sslcert1.png)
    
3. pfx 확장자 인증서를 선택하고 비밀번호를 입력한 뒤 Save버튼을 클릭한다.

    ![인증서업로드2](/images/sslcert2.png)
    
4. 업로드가 완료되면 아래 그림의 1처럼 등록된 인증서를 확인할 수 있다.

    2에서 자신이 등록한 도메인을 선택하고 3에서 인증서를 선택하면 4처럼 bind된 것을 확인할 수 있다.
    
    ![인증서업로드3](/images/sslcert3.png){:width="300px"}
    
5. Save버튼을 클릭해서 마무리한다.

    ![인증서업로드완료](/images/sslcert1.png)
    
적용되기까지 조금 기다린 후 브라우저로 자신의 URL(여기서는 [https://yotest.magmakick.io](https://yotest.magmakick.io))을 입력하고 접속하면 인증서를 확인할 수 있다.

![인증서확인](/images/httpsconfirm.png){:width="400px"}


## 맺음말

이 시간에는 Azure 웹앱에 커스텀도메인과 SSL을 적용하는 방법을 알아봤다.

비록 와일드카드를 지원하지 않고 90일마다 갱신해야하는 인증서지만 인증서도 얻었다.

그리고 도메인에 레코드를 등록하는 일도 해봤다.

앞으로 비슷한 일이 발생할 때 어려움없이 진행하길 기대해본다.

---

참고 링크 

* [https://blog.outsider.ne.kr/1178](https://blog.outsider.ne.kr/1178)
* [https://royaljay.com/development/free-ssl-cert-for-azure-web-apps/](https://royaljay.com/development/free-ssl-cert-for-azure-web-apps/)
