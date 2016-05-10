---
layout: post
title:  "Docker Machine으로 Azure 다루기"
date:   2016-05-09 00:02:00
categories: nodejs azure
comments: true
meta : nodejs azure docker
description : Docker Machine의 Azure driver를 사용하여 Azure에 컨테이너를 배포한다. 
image : /images/logo.png
publish : true
---

* content
{:toc}

## 들어가는 말

Docker를 사용해서 웹 어플리케이션 등을 배포하면 환경을 맞추는 수고를 대폭 줄일 수 있고 확장할 때도 편리하다.

그런데 이를 다수의 리모트 호스트에 설치하고 각각을 관리하는건 귀찮은 일이다.

이런 수고를 덜어주는 것이 [Docker Machine](https://docs.docker.com/machine/overview/)이다.

![DockerMachineLogo](/images/logo.png)

Docker Machine은 로컬 시스템뿐 아니라 다양한 클라우드 서비스 제공자(AWS, Digital Ocean 등)의 환경도 제어할 수 있게 드라이버를 제공한다.

![DockerMachine](https://docs.docker.com/machine/img/machine.png)

그럼 Docker Machine의 Azure 드라이버를 이용해서 환경 설정과 컨테이너 배포를 살펴보도록 하자. 

> 모든 과정은 OSX에서 진행된다. Windows 10의 bash를 사용해도 가능하지만 이 글에서 해당 과정을 모두 커버하지 않는다.

---

## Docker Machine 설치

Docker Engine은 이미 설치되어있다고 치고 Docker Machine을 설치해보도록 하자. 터미널에 아래 명령을 입력한다.

{% highlight shell %}

$ curl -L https://github.com/docker/machine/releases/download/v0.7.0/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine && \
  chmod +x /usr/local/bin/docker-machine

{% endhighlight %}

    주의 : v0.7.0이상으로 설치.
    
설치가 완료되었다면 아래 명령으로 버전을 확인할 수 있다.

{% highlight shell %}

$ docker-machine -v
docker-machine version 0.7.0, build a650a40

{% endhighlight %}

Docker Machine이 설치 되었다.

## 로컬 가상 머신 다루기

이제 로컬 환경에 가상 머신을 생성하고 컨테이너를 배포해보자.

> 여기서는 dev라는 이름을 사용한다.

### dev 머신 생성

생성은 간단하다. virtualbox 드라이버를 이용해서 명령만 입력하면 된다.

{% highlight shell %}

$ docker-machine create --driver virtualbox dev
(dev) Creating VirtualBox VM...
(dev) Creating SSH key...
(dev) Starting the VM...
(dev) Check network to re-create if needed...
(dev) Waiting for an IP...
...(이하생략)...

{% endhighlight %}

위 명령이 실행되면 dev라는 가상머신이 생성된다.

## dev 머신에 Node.js 웹 어플리케이션 배포

docker명령으로 dev 머신을 제어할 수 있도록 환경을 먼저 구성한 후 이미지를 생성하여 배포해보자.


###  dev 머신 환경 변수 설정

우선 아래 명령으로 dev 머신의 환경 변수를 확인한다.

{% highlight shell %}
$ docker-machine env dev
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/yo/.docker/machine/machines/dev"
export DOCKER_MACHINE_NAME="dev"
# Run this command to configure your shell:
# eval $(docker-machine env dev)

{% endhighlight %}

docker 명령으로 dev 머신을 제어할 것이므로 위 커맨드에 마지막에 안내한 명령을 입력한다.

{% highlight shell %}

$ eval $(docker-machine env dev)

{% endhighlight %}


### 이미지 생성

아래 명령을 순서대로 입력하여 Node.js 웹 어플리케이션을 다운받는다.

{% highlight shell %}

$ wget https://github.com/totuworld/NodejsExampleApp/archive/0.0.1.zip
$ unzip 0.0.1.zip
$ cd NodejsExampleApp-0.0.1

{% endhighlight %}

아래 명령으로 이미지를 생성한다.

{% highlight shell %}

$ docker build --tag=nodeapp:0.0.1 .
Sending build context to Docker daemon 17.92 kB
Step 1 : FROM node:4-slim
...(이하생략)...

{% endhighlight %}


### 컨네이너 생성

이미지를 생성했으니 배포해보자.

{% highlight shell %}

$ docker run --name=nodeapp -d -p 80:3000 nodeapp:0.0.1
78ea5a20bb83121157235dc7104aae639054c3cd37d6ff6650d27875e188863e

{% endhighlight %}

위 명령은 nodeapp이란 이름으로 nodeapp:0.0.1 이미지를 배포한 것이다. 그리고 컨테이너의 3000번 포트를 외부 포트 80번에 연결했다.

결과를 확인하려면 환경 변수의 `DOCKER_HOST`부분에 있던 주소를 브라우저에 입력해보자.

![ACSAppResult](/images/ACSAppResult.png)


### dev 머신 정리

일단 아래 그림을 보자. Docker가 컨테이너를 실행하는 것은 다음과 같은 구조를 가진다.

![Docker](https://www.docker.com/sites/default/files/what-is-vm-diagram.png)

여기서 진행한 것은 Docker Engine 위에 올라갈 이미지를 생성한 후 이미지를 컨테이너로 생성한 것이다.


## Azure 드라이버 활용하기

위와 동일한 행동을 Azure 환경에서 진행하려면 어떻게 해야할지 알아보자.

> 여기서는 totuworld란 이름을 사용한다.

### totuworld 머신 생성

Docker Machine을 활용해서 `totuworld`란 머신을 생성해보자. dev 머신을 생성하는 것과 크게 다르지는 않다.

> 자신의 Azure 계정에서 구독 ID(Subscription ID)를 확인해두어야한다. Azure 포털에서 구독 부분을 클릭하여 확인할 수 있다.

{% highlight shell %}

$ export AZURE_SUBSCRIPTION_ID=[구독 ID]
$ docker-machine create --driver azure --azure-open-port 80 totuworld
Running pre-create checks...
(totuworld) Stored Azure credentials expired. Please reauthenticate.
(totuworld) Microsoft Azure: To sign in, use a web browser to open the page https://aka.ms/devicelogin. Enter the code GQKWVKL58 to authenticate.

{% endhighlight %}

자 위와 같이 나오면 반드시 [https://aka.ms/devicelogin](https://aka.ms/devicelogin) 다음 주소로 접속하여 `code`를 입력하여 권한을 준다.

![dockermachineazure](/images/dockermachine.png)

권한이 확인되면 아래와 같은 내용이 터미널에 출력되면서 머신이 생성될 것이다.

{% highlight shell %}

(totuworld) Completed machine pre-create checks.
Creating machine...
(totuworld) Querying existing resource group.  name="docker-machine"
(totuworld) Creating resource group.  name="docker-machine" location="westus"
...(이하생략)...

{% endhighlight %}

모든 과정이 마쳐지면 Azure 포털에서 리소스 그룹 중에 docker-machine이란 그룹이 생성된 것을 확인할 수 있다. 그 안에는 다양한 리소스가 있다.

![dockermachine리소스](/images/dockermachine_resource.png)


### totuworld 머신 환경 변수 설정

환경 변수를 설정하는 것은 앞서 본것처럼 간단하다. env 뒤에 자신의 머신 이름을 입력하면 끝이다.

{% highlight shell %}

$ eval $(docker-machine env totuworld)

{% endhighlight %}


### 이미지 생성

앞서 생성한 이미지는 dev 머신에 있다. Docker Hub로 이미지를 공유하거나 개인 레지스터를 사용하지 않는한 totuworld 머신에도 이미지를 생성해야한다.

NodejsExampleApp-0.0.1 폴더에서 이미지를 생성하는 명령을 입력한다.

{% highlight shell %}

$ docker build --tag=nodeapp:0.0.1 .
Sending build context to Docker daemon 17.92 kB
Step 1 : FROM node:4-slim
...(이하생략)...

{% endhighlight %}


### 컨네이너 생성

배포도 같은 방법으로 가능하다.

{% highlight shell %}

$ docker run --name=nodeapp -d -p 80:3000 nodeapp:0.0.1
0fbfa68f9f97b742759317a5846a084d7efaa370535cf9342150402374cf3067

{% endhighlight %}

아래 명령을 입력해서 확인한 ip 주소를 브라우저에 입력하면 같은 결과를 확인할 수 있다.

{% highlight shell %}

$ docker-machine ip totuworld
13.91.254.246

{% endhighlight %}

![ACSAppResult](/images/ACSAppResult.png)



## 맺음말

Docker Machine 사용은 이번이 처음이다(전에는 다수의 머신을 다룰 일이 없었다). 

사용해보니 머신마다 터미널로 접속하는 일이 없어서 좋았다.

무엇보다 Azure 드라이버를 사용하니 환경을 셋업하는 것이 편리했다.


요즘 MS의 행보가 눈에 자주 들어온다. 또 어떤 것으로 놀래켜줄지 지켜보는 재미가 있다. 

> 이상의 내용은 참고 자료에 링크한 [Episode 204: Using Docker Machine with Azure with Ahmet Balkan](https://channel9.msdn.com/Shows/Cloud+Cover/Episode-204-Using-Docker-Machine-with-Azure-with-Ahmet-Balkan)을 바탕으로 했다. 자세한 진행이 궁금하다면 영상을 한번 살펴보길 바란다. 영상에서 내용을 진행하는  Ahmet Alp Balkans는 Azure 리눅스팀에서 오픈소스 엔지니어로 활동한다. 실제 Docker Machine의 Azure 드라이버 관련 커밋에서 그의 이름을 발견할 수 있다(MS직원인데 Google이 만든 Go 언어로 Docker Machine의 Azure 드라이버를 만들고 있다). 

---

## 참고자료

* [Episode 204: Using Docker Machine with Azure with Ahmet Balkan](https://channel9.msdn.com/Shows/Cloud+Cover/Episode-204-Using-Docker-Machine-with-Azure-with-Ahmet-Balkan)