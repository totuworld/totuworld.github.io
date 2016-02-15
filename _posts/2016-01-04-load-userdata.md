---
layout: post
title:  "사용자 기본 데이터 로드"
date:   2016-01-05 08:00:00
categories: Nodejs서버강좌 Nodejs
comments: true
meta : Node.js서버강좌 - 5
description : 사용자 데이터를 로드하는 방법을 알아본다.
publish : false
---

* content
{:toc}

## 들어가는 말

`usercore`테이블에 userID를 통해 신규 사용자 정보를 추가하는 방법에 대해서 다뤄봤다.

이번에는 사용자 정보를 로딩하는 방법을 알아보자.

---

## 사용자의 기본 데이터 로딩

사용자가 게임에 접속하면 가입 시 얻은 `usercore`테이블의 `no`값을 이용해서 기본 데이터를 데이터베이스에서 읽어와야한다.

`usercore`, `userupgrade`테이블에 생성한 정보 로딩하도록 해보자.

{% highlight javascript linenos %}

/* GET 사용자의 기본 정보를 로딩한다 */
router.get('/get/:no', function(req, res) {
  //usercore 데이터 로딩
  function GetUserCore(callback) {
    models.usercore.find({where:{no:req.params.no}})
    .then(function(findUserCore) {
      //사용자  정보가 있는지 확인
      if(findUserCore == null || findUserCore == undefined) {
        //err:존재하지 않는 사용자
        callback(true);
        return;
      }
      
      //정상적인 콜백.
      callback(null, findUserCore);
    });
  }
  
  //userupgrade 데이터 로딩
  function GetUserUpgrade(callback) {
    models.userupgrade.find({where:{no:req.params.no}})
    .then(function(findUserUpgrade) {
      //정보가 있는지 확인
      if(findUserUpgrade == null || findUserUpgrade == undefined) {
        //err:존재하지 않는 정보
        callback(true);
        return;
      }
      
      //정상적인 콜백.
      callback(null, findUserUpgrade);
    });
  }
  
  async.parallel([
    GetUserCore,
    GetUserUpgrade
  ], function(err, results) {
    if(err) {
      //err:사용자 정보가 존재하지 않는 경우.
      res.send('none0');
    }
    else {
      //TODO: 전송할 결과 생성.
    }
  });
});

{% endhighlight %}

4, 20 번 줄 : `usercore`, `userupgrade`테이블에서 정보를 로딩하여 정보가 존재하는지 확인 한 후 콜백한다.

35 번 줄 : [`async.parallel`메서드](https://github.com/caolan/async#paralleltasks-callback)로 동시에 작업(Task)을 실행한다.

44 번 줄 : 클라이언트에서 데이터를 처리할 수 있도록 전달하면 되겠다.

---

## 하트 계산

`usercore`의 hearts는 일정 시간(10분)이 지나면 1개씩 추가되어야한다. 하지만 최대수량 이상으로 보유중이라면 추가될 필요는 없다.

{% highlight javascript linenos %}

  //하트가 5개 이하인 상태이고 10분이 지났으면 하트를 채운다.
  function FillHeartsAndUpdateLoginTime(userCore, callback) {
    var now = new Date();
    var updateOptions = {};
    updateOptions['loginTime'] = ToTimeString(now);
    
    if(userCore.hearts < 5) {
      var spendTime = now.getTime() - new Date(userCore.loginTime).getTime();
      var totalHeart = userCore.hearts + Math.floor(spendTime/ 600*1000 );
      if(totalHeart > 5) {
        totalHeart = 5;
      }
      if(userCore.hearts != totalHeart) {
        updateOptions['hearts'] = totalHeart;
        userCore.hearts = totalHeart;
      }
    }

    models.usercore
      .update(updateOptions, {where:{no:userCore.no}})
      .then(function(updateResults){
        userCore.loginTime = now.getTime()/1000; 
        callback(null, userCore.loginTime);
      });
  }

{% endhighlight %}

8 번 줄 : 총 지난 시간을 계산한다.

9 번 줄 : 현재 보유한 하트에 10분 당 1개씩 새로 추가되는  


책에서 사용한 클라이언트가 XML로 데이터를 처리한다. 커맨드라인 툴에서 아래 명령을 실행하여 모듈을 추가하자.

    npm install --save xmlbuilder

모듈이 설치 되었으니 `users.js`파일에 아래 변수를 추가한다.

    var xmlbuilder = require('xmlbuilder');

그리고 결과를 전송할 로직을 추가한다.

{% highlight javascript linenos %}



{% endhighlight %}

---