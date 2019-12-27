---
layout: post
title:  yalive 구현소개
date:   2019-12-27 00:00:00
categories: etc
comments: true
meta : etc
description : 1달만에 만든 yalive의 기획?과 개발을 해부합니다
image : 
  twitter: /images/yalive-data-00.jpg
  facebook: /images/yalive-data-00.jpg
publish : true
---

* content
{:toc}

## 들어가는 말
[yalive 제작 후기](https://blog.totu.dev/2019/12/24/yalive-postmortem/)에서 숨가빴던 1달간의 개발 과정을 소개했다면 이번에는 yalive를 살펴보며 동작방식을 설명하기로 한다.

## 라이브 퀴즈쇼의 핵심 기능
라이브 퀴즈쇼는 참여자와 진행자가 모두 같은 문제를 보고 같은 결과를 봐야한다. 대시보드와 관리자 페이지 그리고 많은 클라이언트가 같은 데이터를 비슷한 시점에 수신해야한다.

![화면간데이터공유](/images/yalive-data-01.jpg){:width="50%"}

그림에서 보는 것처럼 상황에 따라 관리자페이지에서 데이터를 수정하면 대시보드와 클라이언트는 이 정보를 반영해서 각자 화면을 제어한다. 데이터 동기화만 해결하면 8할은 개발이 끝난 셈이다.

그럼 어떻게 데이터를 동기화 할 수 있을까? 내 선택은 Firebase의 Cloud Firestore 였다. Cloud Firestore 공식 문서에 따르면 확장한도가 동시 연결 수 약 1,000,000개, 초당 쓰기 1,000회이다. 기획 자체가 실시간 1만명 연결이 아니라 최대 1,000명이 접속할 수 있는 환경이므로 충분하다. 보너스로 인증(authentication)을 사용할 수 있으니 프론트엔드와 백엔드 개발에서 초기 비용을 낮출 수 있다.

### 다른 대안은 없었나?
socket이나 gRPC도 생각해보긴 했다. 그런데 yalive 제작에는 무리라고 봤다. 일단 제작 착수 시점이 배포까지 1달 남은 시점이었다. socket이나 gRPC를 사용하려면 이미 어느정도 제작된 솔루션을 가지고 있어야했다. 개발에 참여하는 멤버 중에 적어도 1명이라도 이 경험을 가지고 있어야 가능하다. 하지만 난 맨손이었다. socket.io로 채팅을 만들었던게 언제인지 기억도 가물가물하다.

Firebase쪽이 상황이 훨씬 좋았다. 다른 토이 프로젝트용으로 Firebase, React.js, Next.js를 이용해서 인증과 회원가입을 처리해뒀다. 덤으로 yalive 개발에 참여한 멤버가 모두 이 스택에 익숙하므로 초기 비용이 낮아졌다.

## 데이터 동기화 살펴보기
이제 데이터 동기화를 구체화시켜보자. 잼 라이브를 떠올려보면 된다.

퀴즈쇼 시작에 앞서 참가자들이 쇼에 들어온다. 진행자가 애드립을 한다.

![잼누나](https://cdn.clien.net/web/api/file/F01/7618286/9253b739d10fa.png){:width="30%"}

문제를 내고 카운트다운을 한다.

![문제출제](https://ext.fmkorea.com/files/attach/new/20190415/486616/1531092467/1739955072/1e95a2e7d501cdb415876542dd9e7906.png){:width="30%"}

참가자 중 몇 퍼센트가 뭘 선택했는지 보여주고 정답을 공개한다.

![정답공개](http://db.kookje.co.kr/news2000/photo/2018/0807/L20180807.99099003342i1.jpg){:width="30%"}

탈락자가 생기고 다시 애드립이 진행된다. 이 과장을 반복하다가 끝난다.

이 흐름을 바탕으로 몇가지 상태를 정의할 수 있었다.

|상태이름|설명|
|PREPARE|준비중. 참가자가 입장할 수 있다|
|IDLE|대기 상태. 진행자의 애드립이 이때 나온다|
|QUIZ|문제가 공개된다|
|COUNTDOWN|각 참가자가 답을 제출해야하는 제한시간|
|CALCULATE|집계. 오답자를 가려서 탈락이 몇 명인지 등을 체크한다|
|SHOW_RESULT|문제의 정답을 공개한다|
|FINISH|모든 퀴즈가 종료된 상태|

각 상태는 정해진 흐름에 따라 다른 상태로 변경될 수 있다.

![상태 흐름](/images/yalive-data-03.jpg){:width="50%"}

### 동기화 데이터 구조
앞서 설명한 상태 정보외에 무엇이 더 필요할까? 먼저 퀴즈에 관한 정보가 필요하다. 전체 참가자와 생존자 숫자도 필요하다.  실제로 사용한 `퀴즈 정보` 인터페이스는 아래 코드와 같다.

```javascript
export interface QuizOperation {
  /** 상태 */
  status: EN_QUIZ_STATUS;
  /** 제목? 노출할 글자? */
  title?: string;

  /** 참가 가능한 이메일 주소 */
  possibleEmailAddress?: string;

  /** 퀴즈 id(각 퀴즈를 구분하는 값) */
  quiz_id?: string;
  /** 퀴즈 타입 */
  quiz_type?: EN_QUIZ_TYPE;
  /** 퀴즈 설명 */
  quiz_desc?: string;
  /** 퀴즈 이미지 url */
  quiz_image_url?: string;
  /** 사용자가 선택할 수 있는 객관식 문항 */
  quiz_selector?: SelectorItem[];
  /** 퀴즈의 정답(반드시 SHOW_RESULT 상태에서 넣어줘야한다.) */
  quiz_correct_answer?: number;

  /** 전체 참가자 숫자 */
  total_participants: number;
  /** 생존한 참가자 숫자 */
  alive_participants: number;
}
```

퀴즈 정답(quiz_correct_answer)를 퀴즈 설정 시 넣지 않고, 정답 공개(SHOW_RESULT) 전에 넣도록 했다. 아무래도 개발자 도구를 열어 미리 데이터를 확인하는 사람도 있을 수 있기 때문이었다. 

### 퀴즈 정보 수정
데이터 수정은 관리자 페이지에서만 가능하도록 했다. 크게 3가지 기능을 수행한다.
* 상태 변경
* 퀴즈 문제 설정
* 오답자 체크 후 생존 참가자 숫자 변경

상태 변경은 현재 상태에 따라 다음 상태로 올 수 있는 버튼을 노출했다.

![IDLE일때버튼](/images/yalive-data-04.png)

예외처리를 한 부분은 COUNTDOWN 상태이다. 화면에는 10초 타이머가 진행되지만 관리자 페이지에서는 15초 타이머가 실행된다. 타이머가 종료하면 자동으로 CALCULATE 상태로 변경하고, `정답 공개` 버튼과 `오답자 계산` 버튼이 차례로 나타난다. 이 두 단계를 마치고 SHOW_RESULT 상태로 변경해야한다. 이 시스템을 잘 이해하지 못하는 사람이 운영했다면 어떤 버튼을 눌러야할지 몰라서 사고 나기 딱 좋은 부분이다. 행사 당일 관리자 페이지 운영을 직접 했기때문에 큰 문제가 없었지만 현재는 아래 처럼 개선했다.

```
* CALCURATE 상태에서 정답 공개와 오답자 계산이 순차적으로 자동 진행
* 각 단계중 실패 시 다시 시도가능하도록 버튼이 모두 노출됨
```

그리고 이런 수정은 관라자 페이지에서 api 호출을 통해서 처리한다. Firebase SDK를 활용해서 관리자 페이지에서 직접 수정할 수 있지만 이후에 정합성 처리 등을 넣으려면 이게 유리하다.

> 그래놓고 실제로 정합성 체크하는 로직은 없다😱  
> [서버 사이드에 JSON Schema 도입 · Issue #31 · ya-live/ya-live · GitHub](https://github.com/ya-live/ya-live/issues/31)  

### 퀴즈 정보 읽기
데이터를 클라이언트와 대시보드, 관리자 페이지에서 사용할 때는 Firebase SDK 기능을 지원받으면 된다. Cloud  Firestore의 문서(document) 변경 사항을 수신하는 onSnapShot 메서드를 사용했다. 프론트엔드 라이브러리가 React.js이니 함수형 컴포넌트(Functional Component)에서 편리하게 사용할 수 있게 Custom Hook을 만들었다.

[ya-live/firestore_hooks.ts at master · ya-live/ya-live · GitHub](https://github.com/ya-live/ya-live/blob/master/components/auth/hooks/firestore_hooks.ts)

내가 처리한 부분은 여기까지인데 클라이언트와 대시보드를 만든 @alattalatta, @0901sj님이 react.js의 context api를 활용해서 각 컴포넌트에서 데이터를 활용할 수 있도록 했다. 관리자 페이지는 하위 컴포넌트를 가지지 않지만 클라이언트와 대시보드는 상황에 따라 컴포넌트가 변경되니 매번 데이터를 흘리는게 귀찮았겠다.

> yalive만들면서 프론트엔드 코드를 볼 수 있어 좋았다.  

## 클라이언트 제출 데이터 처리
참가자 정보는 퀴즈 정보 하위에 위치한다. 생존 여부와 제출한 답안을 가지는 형태다.

```javascript
export interface QuizParticipant {
  /** ISO 8601 */
  join: string;
  /** 생존 여부 */
  alive: boolean;
  currentQuizID?: string;
  /** 선택한 번호 */
  select?: number;
  /** 사용자 고유 id */
  id: string;
  /** 사용자 출력 명 */
  displayName: string;
}
```

![참가자데이터구조](/images/yalive-data-05.png)

각 클라이언트에서 api를 통해 답안을 제출하면 개별 참가자 정보(Participant)에 선택한 답안과 퀴즈 고유 id를 기록한다. 제한 시간이 종료되면 관리자 페이지에서 오답자를 체크하여 alive 값을 false로 바꿔서 더이상 쇼에 참여할 수 없도록 한다.

![답안 제출 과정](/images/yalive-data-02.jpg){:width="50%"}

위 과정에서 api를 통하면 응답을 수신한 시간을 특정할 수 있고 정합성도 체크할 수 있어서 사후에 문제가 발생했을 때 증거를 남길 수 있다. 무엇보다 Firebase SDK를 사용해서 바로 업데이트 하도록하면 정답이 아니더라도 생존 여부를 조작할 수 있다. 어뷰징 요소는 줄이는게 좋다.

뭔가 공통점이 보이지 않는가? Firebase SDK를 사용하지만 데이터 읽기에만 사용하고 수정은 모두 api를 통하고 있다. 이는 일전에 진행한 Firebase 스터디에서 들은 내용을 바탕으로 정한 정책이다. Firebase를 사용해서 게임을 운영하는 분들은 사용자 어뷰징을 걱정한다. 그래서 개별 클라이언트는 데이터 수신만하고 데이터 변경만 서버사이드에 요청하는 방식으로 운영하여 DB접근을 차단한다고 들었다.

> 그래놓고 실제로 정합성 체크하는 로직은 없다😱 (2)  

## next.js v9.x + now = ❤️
데이터에 대한 설명은 이정도로하고 next.js와 now에 관해 얘기해보겠다.

프론트엔드 라이브러리는 react.js를 사용했지만 프레임워크는 next.js를 사용했다. 많은 기능을 지원하지만 서버사이드 렌더링(Server Side Rendering)과 파일 시스템 라우팅(File-System Routing)이 편리해서 손에 익어버렸다. 덤으로 프로젝트에 참여한 멤버들이 모두 현업에서 next.js를 사용한다.

next.js v9.x 버전에서 반가운 2가지 기능이 있다.
첫째는 파일 시스템 기반의 동적 라우팅([File system-Based Dynamic Routing](https://nextjs.org/blog/next-9#dynamic-route-segments)) 이다. [nuxt.js](https://ko.nuxtjs.org/guide/routing/)에서 가장 부러운 기능이 동적 라우팅이었는데 이제 된다. 파일 이름에 대괄호([ , ])를 추가하면 url path를 해석해서 해당 정보를 query로 전달해준다.

둘째는 API 라우트([API Routes](https://nextjs.org/docs#api-routes))다. 기존에 next.js를 사용해도 서버 사이드에서 처리하는 api는 직접 express.js 등을 활용해서 제작했다. 이제는 `/pages/api` 폴더 아래에 파일을 생성해서 백엔드 api를 사용할 수 있다.

now는 next.js를 만든 zeit사가 운영 중인 서버리스(serverless) 플랫폼이다. 여기에 next.js를 사용하면 아주 기간막히게 잘 달라붙는다. 예를들어 next.js의 API 라우팅을 사용하면 각 파일을 별도의 서버리스 서버로 동작하게 한다 - 내부에서는 aws의 lamba로 1:1 매핑하는 듯하다). aws에서 beanstalk 하나 등록하는 수고를 생각하면 배포 설정도 정말 간단하다.