---
layout: post
title: "TypeScript + Next.js로 Webview 처리하기"
date: 2018-01-24 00:00:00
categories: Github tech android webview nextjs typescript polyfill es5
comments: true
meta : Github tech android webview nextjs typescript polyfill es5
description : 모바일 어플리케이션 Webview에 TypeScript, Next.js 도입기
image :
  twitter: /images/cover3.jpg
  facebook: /images/cover3.jpg
publish : true
---

> 이 글은 2018년 1월 24일에 [야놀자 기술 블로그][https://yanolja.github.io/2018/01/Nextjs-webview]에 게시된 글을 백업한 것 입니다.

저는 야놀자 CX서비스실의 API 파트에서 백엔드(90%)와 웹 프론트엔드(10%) 프로그래머로 일하는 송요창입니다.

[야놀자 앱](https://iyq-i.tlnk.io/serve?action=click&publisher_id=354062&site_id=135377&site_id_android=135378)은 모바일 어플리케이션(Native Mobile Application)입니다. 그렇지만 몇몇 화면은 웹뷰(Webview, 이하 웹뷰)를 사용해서 하이브리드(Hybrid, 이하 하이브리드)로 처리하죠. 최근에 TypeScript와 Next.js를 활용해서 웹뷰를 처리한 경험을 얘기해보겠습니다.


## 과거
기존 [야놀자 앱](https://iyq-i.tlnk.io/serve?action=click&publisher_id=354062&site_id=135377&site_id_android=135378)에서 사용하는 하이브리드는 php + javascript(jQuery)를 통해 제작해왔습니다. 이 스택은 2017년 8월 이전까지 서비스하던 야놀자 웹 서비스에서도 사용되었습니다.

이 스택으로 지속적인 개발과 유지 보수하기에는 문제가 있었습니다. 번들링이 없어서 개발자가 직접 library를 복사 붙여넣기했기때문에 실수로 코드가 유실되어 의도하지 않은 버그가 발생하기도 했습니다. 또한 클라이언트가 사용하는 API와 웹뷰로 사용하는 부분이 함께있어 모듈화가 어렵고 재사용성도 떨어졌습니다. 비슷한 코드가 같은 프로젝트에서도 여러개 발견되었습니다.

### 대안에 관한 탐색

새로운 하이브리드를 구성할 때 API파트 TL님이 Vue.js를 사용하는 Nuxt.js 프레임워크와 React.js를 사용하는 Next.js 프레임 워크를 두고 고민했습니다. API파트 구성원이 Vue.js와 React.js 둘다 모르는 상황이라서 무엇을 선택해도 공부는 해야했습니다. Vue.js는 section별로 html, css, js가 분리되어 다수의 프로그래머가 협업하기에 유리할 것으로 예상되었죠. 그런데 이미 회사에서 2017년 8월 [야놀자 웹 2.0](https://www.yanolja.com)이 출시했을 때는 node.js + express.js + react.js + redux 스택을 사용했습니다. 회사에서도 React.js로 Frontend를 통일하려고 하고 있어 다양한 조직간 library 공유가 가능한 상황이었습니다.


## TypeScript ️️+ Next.js = 👍
React.js로 방향을 결정한 뒤 야놀자 웹 2.0 프로젝트의 경험을 바탕으로 새로 제작하는 하이브리드는 3가지를 개선하고자 했습니다.

* 코드 작성 시 코드 에러를 검출하자. 👉 TypeScript
* 편리한 서버 사이드 렌더링 + 초기 렌더링 서버 구동 속도 보장하자. 👉 Next.js
* 다수의 파일을 생성하는 일을 줄이자. 👉 Mobx

TypeScript, Next.js, Mobx 사용은 API파트 프로그래머 전원이 만족한 선택이었습니다.

![박수쳐](http://kstatic.inven.co.kr/upload/2018/01/06/bbs/i15924387052.gif)

TypeScript 도입으로 코드 레벨에서 에러가 검출되죠(이득). 파라메터와 리턴을 타입으로 명시해서 코드 구현부를 파악하지 않아도 코드 이해와 재사용에 용이했습니다(🐶이득). Mobx는 Redux를 사용할 때 의례해야하는 ActionType, Action, Reducer 작성 지옥에서 탈출하게 해주었습니다.

하지만 단점도 있었습니다. Next.js에 다이나믹 라우팅이 지원하지 않아서 `/users/:id` 형태와 같은 라우팅은 별도의 라이브러리를 사용해야했습니다(Next.js 공식 예제에도 안내된 내용). 개발중에 TypeScript가 업데이트되서 최신 버전을 적용하면 에러가 나서 API 파트 내에서 일괄 수정하는 일도 벌어졌습니다. 이런 불편이 있었지만 만족스런 선택이었습니다.

## 사건 1) babel polyfill

새로운 하이브리드 페이지를 만들고 QA는 특별하지 않은 이슈들로 무사히 진척되고 있었는데 배포 하루 전 QA실에서 거짓말 같은 버그🐞🐛를 찾아냅니다.

    [재현절차]
    * Android 4.4.2
    * 비밀번호 변경 페이지
    * 다음에 변경할께요 or 재설정하기 버튼 선택

    [재현결과]
    * 다음에 변경할께요 or 재설정하기 버튼 동작하지 않음

안드로이드 개발자와 함께 로그를 확인했습니다.

    [INFO:CONSOLE(4898)] "Uncaught SyntaxError: 다음 생략 :)

하이브리드 페이지에서 사용할 html응답은 렌더링 서버가 서버사이드에서 생성해서 전송하므로 이상없이 페이지가 노출되었지만 각 기능을 제어하는 javascript가 동작하지 못하는 상황이더군요. 혹시나 Android 4.4.2를 사용하는 사용자가 적다면 제외하려고 했는데 GA를 확인해보니 실시간 유입 사용자의 1%가 해당 버전을 사용하고 있었습니다.

### 구글신이 아실꺼야

Android 4.x, react, webview를 주요 키워드로 검색해봤는데 이렇다할 방법을 찾지 못했습니다. 그러다가 발견한 글이 `es6`로 된 것이 문제일 수 있다는 정도의 지적이었죠.

> 결과적으로 소득이 없던 것은 아닙니다.


### 시도 1, const 제거

웹뷰 프로젝트는 TypeScript를 트랜스파일한 뒤 Next.js의 build 명령으로 빌드하여 배포합니다. 빌드된 소스코드를 확인해보니 `const`처럼 es6에서 사용가능한 선언이 발견되었습니다. 우선 모든 const를 `var`로 변경해봤습니다.

결과는 실패. 웹뷰 프로젝트의 빌드된 코드에서 `Class` 선언이 발견되어 앞서 발견된 에러가 지속되었습니다.

### 시도 2, 모듈의 es5 빌드

Class 선언은 다수의 프로젝트에서 사용중인 CX서비스실 내 공통 패키지에서 발견되었습니다. 이 역시 TypeScript를 활용해서 es6로 빌드합니다. 이를 es5로 빌드하도록 변경해서 적용했습니다.

결과는 실패. 웹뷰 프로젝트의 빌드된 코드에서 `Set` 선언이 나와서 에러가 해결되지 않았습니다.

### 시도 3, polyfill

Set을 어느 패키지에서 사용하는지 확인해보니 Next.js가 사용하는 Webpack이 범인이었습니다. 인터넷에서 관련이슈를 검색했을 때 특정 메서드를 polyfill로 추가하여 회피했다는 글을 본 것이 얼핏 기억났습니다.

> 하도 검색을 많이해서 어떤 글에서 발견했는지 기억이 나지 않지만 어쨌든 뇌에 들어있었습니다.

api 파트의 TL(Tech Lead)님이 프로젝트 내에서 하이브리드로 사용하는 페이지 헤더에 [바벨 폴리필](https://cdnjs.com/libraries/babel-polyfill) 사용을 권했습니다.

> `바벨`은 트랜스파일러(transpiler)로 javascript를 사용할 때 최근 기술 규격을 준수하여 프로그래밍하고 구형 브라우저와 호환을 위해 과거 규격으로 변경해주는 도구입니다. `폴리필`은 특정 기능을 구형 브라우저에서 사용할 수 있도록한 코드 조각이나 플러그인을 말합니다. `바벨 폴리필`은 바벨이 제작한 폴리필입니다.

적용결과 대 성공!!!

정리하자면 Next.js로 하이브리드를 처리할 때 Android, iOS의 **최소 지원 버전**을 확인하고 자료 조사를 한 뒤 시작하길 권합니다.

* react나 Next.js를 써야하는가? jQuery로 하면 안되나? es5를 만족하는 javascript만 사용하면 안되나?
* Android 4.x 버전을 지원해야하는가?
  * Yes. es5 패키지 사용 + polyfill 추가
  * No. 후후후후 좋겠네요.


## 사건 2) 다수의 인스턴스로 서비스할 때 build id 문제

개발환경과 QA환경은 AWS beanstalk에 1개의 인스턴스를 만들고 배포했습니다. 아무런 문제가 없었습니다. Production 환경은 3개의 인스턴스를 만들고 배포했죠. 첫 배포에는 문제가 없었습니다. 그런데 이후 진행된 패치 작업에서 지속적인 배포 실패가 일어났습니다.

### 배포 실패 사유

AWS beanstalk의 배포 방식은 Rolling입니다.

- n개의 인스턴스 중 m개를 사용자와 접촉하지 못하는 상태로 만들고 새로운 코드를 배포합니다.
- 배포가 완료되면 다시 서비스에 투입하는데 이때 health check가 성공해야 다음 인스턴스로 넘어갑니다.

위처럼 진행되어야하는데 health check가 계속 실패해서 배포가 실패하더군요. Cloudwatch 로그를 보니 아래와 같은 에러가 쌓이고 있더군요.

    Error: Invalid Build File Hash(OhMyGod) for chunk: app.js

검색해보니 [Error: Invalid Build File Hash](https://github.com/zeit/next.js/issues/2978)라고 next.js 레포지토리에 이슈로 올라가 있었습니다. Next.js가 실행될 때 production 내의 3개 인스턴스가 각각 다른 build id를 사용하므로 a 인스턴스의 요청이 b 인스턴스로 들어가면 에러가 발생되어 health check를 실패합니다.


### 대응 1, blue/green 배포

서비스 운영을 위해 진행되는 패치가 반드시 필요한 상황이었기 때문에 production 환경을 복제하여 생성하고 CNAME을 swap하는 방식으로 blue/green 배포를 진행했습니다.

- 복제 환경에 신규 버전 배포
- CNAME을 swap하여 복제 환경이 live로 전환
- 모니터링 진행 후 기존 환경에도 신규 버전 배포

위 과정으로 진행하면 복제된 환경에 사용자 요청이 없기때문에 health check를 통과합니다. 성공적으로 배포할 수 있었으나 CloudWatch 로그를 볼 때 매번 2개의 환경을 확인해야한다는 불편한 해결책이었습니다.

### 대응 2, build id 패키지 사용 + sticky session 설정

앞서 언급한 [Error: Invalid Build File Hash](https://github.com/zeit/next.js/issues/2978) 이슈에 번들링 단계에서 build id를 특정하는 방법이 소개되어있습니다. 문제를 해결한 사람이 [next-build-id](https://www.npmjs.com/package/next-build-id)란 패키지까지 만들었더군요. 해당 패키지를 프로젝트에 적용했습니다.

그리고 문제를 살펴보면 a 인스턴스 요청이 b 인스턴스로 들어가서 생기는 문제 이므로 user 1의 요청이 a 인스턴스로 들어오면 session이 유지되는 동안은 user 1의 요청이 a 인스턴스로 유지되도록 sticky session 설정도 해주었습니다.

> 이에 관해서는 Next.js의 위키 페이지 중 [Deployment](https://github.com/zeit/next.js/wiki/Deployment)에 소개되어있습니다.

두가지 모두를 적용한 후 rolling 배포를 해도 문제없습니다. 이제 로그확인하러 2 환경을 확인하는 일은 사라졌습니다.

## 글을 마치며

![4.x폭파기원](/images/android442.png)

사고 1이 발생했을 때 안드로이드 개발자가 4.x 이하가 폭파(?)되길 기도하기도 했지만 해결되었습니다😎😎. 배포 전날 크리티컬 버그가 발견되어 아찔했지만 여러 동료들의 도움으로 배포 가능했네요.

이 작업에 많은 도움을 주신 분들에게 이 자리를 빌어 감사인사를 전하며 글을 마칩니다.

* 갑작스러운 에러에 배포 시점까지 재고해야하는 수준으로 말했지만 기다려준 기획자 허보경님.
* 비지니스 로직을 다수의 디바이스로 꼼꼼히 체크해주신 QA실 박현주님, 곽선명님, 김광선님.
* 타 파트 일이지만 세세히 로그 확인하며 일본어 검색까지 마다하지 않는 Android 파트 프로그래머 노현석님.
* 침착하게 문제를 해결할 수 있도록 해주신 API파트 TL 이병준님.

> Android 4.x, react, webview로 검색했을 때 뚜렷한 해결책이 보이지않았기에 기록을 남깁니다.

## one more thing
CX서비스실 내의 다양한 파트에서 구인중입니다.
[원티드](http://wntd.co/RMJnyd)를 보시거나 totuworld@yanolja.com 으로 문의해주세요. 
