---
layout: post
title:  "아빠 왜 집에서 일해요? (feat. aws)"
date:   2019-01-09 00:00:00
categories: aws
comments: true
meta : aws beanstalk
description : AWS Beanstalk으로 node.js 어플리케이션 배포 시 npm이 망하면?
image : 
  twitter: /images/aws_live_fix.png
  facebook: /images/aws_live_fix.png
publish : true
---

* content
{:toc}

## 들어가는 말

AWS의 PaaS(Platform as a Service)인 Elastic Beanstalk(이하 Beanstalk)로 node.js 어플리케이션을 배포/운영하고 있다. 로컬이나 dev 환경에서 작동을 확인한 배포판이 production 환경으로 배포만 하면 npm install 시 문제를 발생시킨다?!. 이때 어떻게 대처했는지 기록을 남긴다.

## 어플리케이션의 미동작 확인
크론잡(cronjob)처럼 일정 주기로 정해진 일을 처리하는 어플리케이션이 있다. 해당 어플리케이션은 각각의 테스크를 수행하고 슬랙의 특정 채널에 결과를 남긴다. 그런데 production 환경의 메시지가 사라졌다.

### 어디가 문제야?
Beanstalk 환경에 접속해서 node.js 로그를 확인해봤다.

```bash
## nodejs 로그를 확인
$ tail /var/log/nodejs/nodejs.log
```

production 배포판이 특정 패키지를 찾을 수 없어서 시작단계에서 실패하고 있었다. 최근 로그 100줄을 다운받아 확인해보니 npm install 시 특정 패키지를 설치하는데 실패한 기록이 있었다. 개발 환경을 배포한 배포판과 같은 commit으로 돌리고 패키지 설치를 진행해보니 실제로 패키지가 없었다. 어떤 과정에서 누가 패키지를 삭제했는지 모르겠지만 일단 production 환경을 살리자!

## 복구 작업
우선 문제가 된 패키지의 상위 버전이 있는지 확인해서 설치했다. 그리고 npm audit 에서 문제로 지적된 몇몇 패지키지도 업데이트했다. 그리고 배포판을 만들어서 dev 환경으로 뿌렸더니 잘 된다. 그런데 production으로 배포해보니 실패를 거듭했다. 역시 패키지 설치단계에서 에러를 반환하고 특정 패키지가 없다는 에러를 내고 있었다.

### 그렇다면 내가 직접 설치해주지!
`eb cli`로 Beanstalk 환경에 접속한 뒤 아래처럼 실행했다.

```bash
## sudo 권한 획득
$ sudo su

## 설치된 nodejs 버전 보기. 10.13 버전이 있는지 확인하는 용도
$ ls /opt/elasticbeanstalk/node-install

## path 설정
$ export PATH=$PATH:/opt/elasticbeanstalk/node-install/node-v10.13.0-linux-x64/bin/

## 소스코드 있는 곳
$ cd /var/app/current

## package 설치!
$ npm install
```

직접 패키지를 설치했더니 문제없이 잘 설치되었다. 이제 남은건 node 프로세스를 재시작하는 것!

```bash
## node 프로세스 kill (sudo 권한 필요)
$ pkill -f node

## 프로세스 list 확인
$ initctl list

## nodejs 서비스 시작
$ initctl start nodejs
```

이제 Beanstalk의 health check을 지켜보거나 node.js 로그를 확인해보자.

```bash
## nodejs 로그를 확인
$ tail /var/log/nodejs/nodejs.log
```

## 마무리
출근해서 왜 문제가 되었나 살펴봤더니 dev와 production 환경의 node.js 플랫폼 버전이 달랐다. 버전을 맞추니 모든 문제가 사라졌다. 역시 기본이 제일 중요하다.

> 아빠가 집에와서 일하느라 놀아주지 못해서 미안해~ 😭