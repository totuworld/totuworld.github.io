---
layout: post
title:  "30인 규모팀의 커피 주문하기 - 커피투게더"
date:   2019-04-17 00:00:00
categories: etc
comments: true
meta : etc react toy
description : 30인 규모의 커피 주문은 괴롭다. 이를 해결하기 위한 서비스 개발기
image : 
  twitter: https://live.staticflickr.com/8186/8080939609_646de87c04_c.jpg
  facebook: https://live.staticflickr.com/8186/8080939609_646de87c04_c.jpg
publish : true
---

* content
{:toc}

## 들어가는 말
조직 생활을 하다보면 커피(차) 한 잔하며 휴식하는 일이 생긴다. 도란 도란 얘기도 나누고 새로온 사람과 인사도 나눌 수 있는 꼭 필요한 시간이다. 하지만 팀 규모가 10인을 넘으면 주문이 일이다. 야놀자의 범 CX실(CXPO, CXDEV)은 2019년 1/4분기에 30명이 넘어갔다. 와아~

## 커피 주문의 문제
커피 주문은 간단하다. 원하는 주문을 말하고 계산하면 된다.  하지만 사람이 많아지면 주문을 모으는 일이 힘들다. 아래 예시를 보자.

![주문예시](/images/coffeeorder/coffee_order_01.jpg){:width="40%"}

CX실의 규칙은 다음과 같다.

* 자신이 주문할 메뉴가 있는지 쓰레드를 살핀다.
* 주문이 없다면 신규 메뉴를 쓰레드에 남긴다.
* 혼자만 주문했어도 반드시 ✋ 이모지를 남겨서 카운트하기 쉽도록 한다.

위 예시에서 `아이스 아메리카노` 를 살펴보자. 샷을 추가하가한 메뉴와 `아이스아메` 라고 따로 등록한 것까지 비슷한 주문이 흩어져있다. 또 아이스아메는 ✋이모지가 없다. 신규 입사자가 늘어나면서 규칙 전파가 잘 안된다고 볼 수 있다.

주문 몇 잔인지 체크하기도 어렵다. 또 실제 주문 중인데 주문이 바뀌는 일도 가능하다. 주문에 성공해도 어떤게 내 주문인지 확인하기 어려웠다.

### 문제 정리

위 문제를 해결하는 방법을 정리해보자.

* 같은 주문을 누적해서 표시
* 총 몇 잔인지 합계 표시
* 특정 시점에서 주문을 종료하여 변경 불가능하도록 설정

## 신기능 제작!

CX실에는 [워크로그 개발기](https://yanolja.github.io/2018/09/Work-Log)가 기 개발된 상태였다. 곧 CX실 인원이 커피 주문을 하는 서비스라면 `워크로그`에 새로운 기능을 추가하면 될 터!

### 1일 차 - api 개발

커피(차) 한 잔 하는 하는 휴식을 `이벤트`로 생각했다. 이벤트에 주최자와 참석자를 특정할 수 있다. 그리고 이벤트는 `주문`을 가질 수 있다. 주문은 `음료` id를 특정하는데 이때 주문자의 - 휘핑 없이같은 - 옵션을 추가할 수 있다.

![주문db](/images/coffeeorder/coffee_order_db.png)

개략적인 구조는 저렇다. 이를 바탕으로 `이벤트`, `주문`, `참가자`, `음료`에 관한 [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) api를 제작했다.

* GET /events 이벤트 목록
* POST /events 이벤트 등록
* GET /events/:event_id 이벤트 정보
* PUT /events/:event_id 이벤트 수정
* GET /events/:event_id/guests 참가자 목록
* POST /events/:event_id/guests 참가자 등록
* GET /events/:event_id/orders 주문 목록
* POST /events/:event_id/orders 주문 추가 및 갱신
* DELETE /events/:event_id/orders/:guest_id 주문 삭제
* GET /beverages 음료 목록
* POST /beverages 음료 등록

추가로 사내 메신저인 [슬랙](https://slack.com/) 데이터를 워크로그 DB에 등록한 상태이므로 `참가자` 에게 메시지 발송하는 기능도 추가했다.

### 2일 차 - 화면 개발
개발한 화면은 총 3개다.

* 전체 `이벤트` 목록 출력 페이지
* `이벤트` 등록 페이지
* `이벤트` 상세 페이지

재사용가능한 컴포넌트가 별로 없어서 새로 만들었지만 구현이 간단한 편이라서 생각보다 빨리 완료했다.

> 워크로그는 razzle + after.js + react.js + mobx 기반

![뚝딱맨](/images/coffeeorder/quickquick.png)

## 운영 기록

2일 간의 가열찬 개발 후 대망의 서비스 오픈!!

{% include mp4player.html path="/images/coffeeorder/i_love_coffee.mp4" caption="커피 주문 동작예시" %}

CX실 채널에서 크게 환영받았다.

![서비스오픈](/images/coffeeorder/coffee_order_launch.png)

손쉽게 끝날줄 알았지만 데이터베이스(Database)가 터진다.

![서비스에러](/images/coffeeorder/coffee_order_error.jpg){:width="80%"}

돈으로 막을 수 있는건 돈으로 막는게 최고! Firebase를 유료 모델로 올려서 첫날 서비스를 마쳤다.

> 최소 기능만 가지고 서비스를 강행 오픈했을 때 참사를 경험

### 리팩토링

가만 보니 사용자 전체 목록 조회 부분과 이벤트 내 주문, 음료 목록 전체 로딩을 매번 새롭게 하고 있었다. 변경(Create, Update, Delete)발생할 때만 새롭게 데이터를 읽어서 서버 인메모리에 저장한다. 평소의 읽기는 인메모리 캐시를 먼저 확인하고 비었으면 데이터베이스로부터 데이터를 읽도록 했다.

## 마무리

이 후 자잘한 업데이트 중이다. 죽었던 다른 기능을 살리고, 불편 사항을 접수해서 개선하기도 한다. `토이 프로젝트`는 재미있다.

---

## 참고자료
* [워크로그 레포지토리](https://github.com/totuworld/time-recorder-viewer)