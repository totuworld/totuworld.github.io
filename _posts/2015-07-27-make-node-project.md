---
layout: post
title:  "Node.js + express 프로젝트 구성"
date:   2015-07-27 18:00:00
categories: Nodejs서버강좌 Nodejs
comments: true
meta : Node.js서버강좌 - 2
description : Node.js + express를 활용한 프로젝트를 생성합니다.
---

* content
{:toc}

[![video tutorial]({{"/images/nodejs_tuto_title_02.png"}}){:height="360px" width="640px"}](https://www.youtube.com/watch?v=4k34YcTDvns)

## 들어가는 말

[지난 시간에 VirtualBox와 Vagrant로 컴퓨터에 VM을 생성했다.](http://totuworld.github.io/2015/07/24/lemp/) 이제는 해당 서버에서 작동할 Node.js 프로젝트를 만들 것이다.

[따라하면서 배우는 NGUI 유니티 게임 프로그래밍](http://wikibook.co.kr/unity-ngui/)에서는 php를 활용했지만 여기서 Node.js로 변경한 이유는 예시가 많고 javascript라서 php보다 친숙할 것이라고 생각되서다.[^1]

---

## Node.js와 Express 소개

웹 프로그래밍을 경험했다면 HTML이란 말을 한번은 들어봤을 것이다. 예전에는 웹 페이지의 뼈대가되는 HTML를 나모 웹에디터, Dreamweaber 등으로 직접 편집하던 시절이 있었다. javascript는 웹 페이지의 멋을내는 용도에 불과했고 화려함은 flash가 담당했다.

그런데 기술이 차츰 발전하다보니 이제 javascript로 프론트엔드(front-end)와 백엔드(back-end)를 모두 처리할 수 있게 되었다. Node.js는 이러한 배경에서 나타난 서버사이드 javascript기술로 웹 서버나 SMTP서버 등과 같은 서버 프로그램을 작성할 수 있다.

Node.js의 많은 모듈 중에서 express는 웹과 모바일 애플리케이션을 위한 다양한 기능을 제공하는 Node.js 웹 프레임워크(framework)이다.[^2]            

---

## 필요사항

* Node.js >= 0.10.22 - [Install](https://nodejs.org/download/)

Node.js 각 운영체제별 설치파일을 제공하므로 손쉽게 설치가 가능하다.

---

## Express 어플리케이션 생성

windows와 osx에서 커맨드라인(Command Line) 툴을 사용하기 위한 방법은 아래와 같다.

* Windows 환경에서 명령프롬프트 사용하기 

> `window + r` 실행 후 `cmd` 입력하고 실행

* osx 환경에서 터미널 사용하기

> `control + space`를 누르고 `terminal`혹은 `터미널`로 입력한 후 `Enter`키


커맨드라인 툴이 준비되었다면 [express 공식 문서](http://expressjs.com/starter/generator.html)에 의거하여 npm[^3]을 통해서 express-generator를 설치한다.

	npm install express-generator -g

이제 프로젝트를 생성해보자. 프로젝트가 위치했으면하는 폴더로 이동한 후 아래와 같이 명령을 입력한다.

	express myapp

`myapp`은 프로젝트의 이름으로 자신이 원하는 이름을 입력해도 된다. 만들어진 폴더로 이동하면 아래와 같이 파일과 디렉토리가 생성된다.[^4]

	.
	├── app.js
	├── bin
	│   └── www
	├── package.json
	├── public
	│   ├── images
	│   ├── javascripts
	│   └── stylesheets
	│       └── style.css
	├── routes
	│   ├── index.js
	│   └── users.js
	└── views
	    ├── error.jade
	    ├── index.jade
	    └── layout.jade


 `myapp` 프로젝트가 필요로하는 모듈을 설치하기 위해 아래와 같이 명령을 순서대로 입력한다.

	cd myapp
	npm install
	
명령을 수행하면 node_modules이란 폴더가 생성되고 `package.json`의 `dependencies`부분에 열거된 모듈이 설치된다.

이제 잘 작동하는지 확인해보자. 커맨드라인 툴에서 아래와 같이 입력한 후 브라우저에서 `http://localhost:3000`으로 접속한다.

	node bin/www
	
![WelcomeToExpress]({{"/images/welcome_express.png"}})

위 그림처럼 브라우저에 출력되고 커맨드라인 툴에는 아래와 같은 내용이 출력될 것이다.

	GET / 200 440.741 ms - 170
	GET /stylesheets/style.css 200 69.064 ms - 110
	GET /favicon.ico 404 90.681 ms - 1246
  
---

## 맺음말

앞에서 프로젝트를 구동할 VM을 생성했고 express 프로젝트 기본 구조를 만들었다. 앞으로는 Node.js와 express를 통해서 실제 기능을 붙여가는 방법을 배워보도록 하겠다.  

---


[^1]: 제가 Node.js에 익숙해져서 그런 것입니다.(자진납세)

[^2]: [웹 프레임워크는 웹 페이지를 개발하는 과정에서 겪는 어려움을 줄이는 것이 주 목적으로 통상 데이터베이스 연동, 템플릿 형태의 표준, 세션 관리, 코드 재사용 등의 기능을 포함하고 있다.](https://ko.wikipedia.org/wiki/웹_애플리케이션_프레임워크)

[^3]: npm은 package manager로 모듈을 손쉽게 설치, 관리할 수 있도록 돕는다.

[^4]: 이렇게 기본 뼈대를 생성하는 과정을 스케폴딩(scaffolding)이라 한다.