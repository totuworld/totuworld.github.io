---
layout: post
title:  "Sequelize.js 적용"
date:   2015-08-07 07:00:00
categories: Nodejs서버강좌 Nodejs Sequelize
comments: true
meta : Node.js서버강좌 - 3
description : Sequelize.js를 이용해 DB를 모델과 맵핑합니다.
---

* content
{:toc}

## 들어가는 말

앞서 VM에 MariaDB를 설치했고 Express 프로젝트를 생성했다. 이번에는 사용자 정보나 상점, 친구, 결제 등 주요한 정보를 저장하게되는 MariaDB에 Node.js에서 어떻게 접근하여 다루는지 살펴보자.   

---

## ORM을 왜 사용하는가?

관계형 데이터베이스(Relational Database, RDB)[^1]를  사용하면 데이터베이스의 데이터를 조회 및 조작하기 위해서 SQL [^2] 를 작성한다. 간단한 SQL 문장을 작성하는 것이야 어렵지 않지만 사용하는 언어와 사용되는 데이터베이스에 따라서 특성이 있어서 불편한 점이 많다.

그래서 등장한 것이 ORM(Object-relational mapping)이다. ORM은 통역으로 생각하면된다. 프로그래밍 언어로 직접 데이터베이스에 말할 수 없으니 중간에 통역인 ORM이 데이터베이스가 알아들을 수 있는 언어(SQL 등)로 번역하여 전달하는 것이다.

Node.js의 ORM은 [Sequelize.js](http://docs.sequelizejs.com/en/latest/), [Bookshelf.js](http://bookshelfjs.org), [node-orm2](http://dresende.github.io/node-orm2/) 등이 있다. 여기서는 Sequelize.js를 사용하도록 한다.      

---

## Sequelize.js 설치

커맨드라인 툴로 `myapp` 프로젝트가 위치한 폴더에서 아래와 같이 입력한다.

	npm install --save sequelize
	npm install --save mysql

Sequelize.js는 ORM으로 필요하고 mysql은 Sequelize.js로 MySQL, MariaDB를 다루기 위해서 필요한 모듈이다.

(앞에서도 말했지만 osx의 경우 인스톨이 진행되지 않으면 앞에 sudo를 붙여서 명령한다.)

---

## 준비 작업

* MariaDB에 사용할 데이터베이스를 생성한다.

1. 생성해둔 VM을 `vagrant up` 명령으로 실행한다.
2. `http://localhost:8080/phpmyadmin` 주소로 접속한다. (Username : root, Password : 1234)
3. phpMyAdmin으로 Databaes 탭을 클릭 한 후 myapp 데이터베이스 생성를 생성한다.
![MakeDB]({{"/images/make_db.jpg"}})


* `myapp` 프로젝트에 `config` 폴더를 생성 한 후 `config.json`파일을 작성한다.  

~~~
	.
	├── config
	│   └── config.json
~~~

`config.json`의 내용은 아래와 같다.

{% highlight json %}
{
  "development": {
    "username": "root",
    "password": "1234",
    "database": "myapp",
    "host": "localhost",
    "port": 6306,
    "dialect": "mariadb",
    "pool": { "max": 5, "min": 0, "idle": 10000 }
  }
}
{% endhighlight %}

위 내용은 express 웹 어플리케이션이 MariaDB에 접속할 때 사용할 설정이다.

* 프로젝트에 `models` 폴더를 추가한 후 `index.js` 파일을 생성한다.

~~~
	.
	├── models
	│   └── index.js
~~~
	
`index.js`의 내용은 아래와 같다.

{% highlight javascript %}
"use strict";

var fs        = require("fs");
var path      = require("path");
var Sequelize = require("sequelize");
var env       = process.env.NODE_ENV || "development";
var config    = require(__dirname + '/../config/config.json')[env];
var sequelize = new Sequelize(config.database, config.username, config.password, config);
var db        = {};

fs
  .readdirSync(__dirname)
  .filter(function(file) {
    return (file.indexOf(".") !== 0) && (file !== "index.js");
  })
  .forEach(function(file) {
    var model = sequelize.import(path.join(__dirname, file));
    db[model.name] = model;
  });

Object.keys(db).forEach(function(modelName) {
  if ("associate" in db[modelName]) {
    db[modelName].associate(db);
  }
});

db.sequelize = sequelize;
db.Sequelize = Sequelize;

module.exports = db;
{% endhighlight %}

`index.js`의 내용을 살펴보면 먼저 `sequelize` 변수로 데이터베이스에 접속한다. 그리고 models 폴더의 파일을 모두 읽어서 `db`변수에 연결한다. 마지막으로 `db`를 `module.exports`에 할당하여 모듈로써 models 폴더가 작동할 수 있도록 한 것이다.

말이 길었지만 정리하면 곧 models 폴더가 모듈이 된 것이다. [^3] 

---

## 테이블 정의

앞에서 데이터베이스를 활용하기 위한 모든 준비가 마쳐졌다. 이제 사용할 테이블을 정의해보도록 하자.

저장하고같은게임에서 사용자 정보를 저장하는 테이블을 만든다고 해보자. 사용할 ID와 보석, 코인, 하트같은 게임 내 정보를 담게 된다. 최상위 점수라던가 로그인 시간도 추가한다.  

Name | Type | Default | Attributes | Index | AutoIncrease
--- |:--- | --- | --- | --- | ----
no | INTEGER | - | UNSIGNED | PRIMARY | o
id | STRING | - | - | - | -
gems | INTEGER | 0 | UNSIGNED | - | -
coins | INTEGER | 0 | UNSIGNED | - | -
hearts | INTEGER | 0 | UNSIGNED | - | -
highScore | INTEGER | 0 | UNSIGNED | - | -
loginTime | DATE | 2002.06.05 | - | - | -

테이블을 정의할 때 어떤 [데이터타입(Data types)](http://docs.sequelizejs.com/en/latest/docs/models-definition/#data-types)이 있는지는 링크를 확인하기 바란다.

---

## 모델 정의

위 표와 같이 정의된 테이블을 Sequelize를 사용해서 model과 맵핑해보자. 기본 구조는 간단하다.

{% highlight javascript %}
var usercore = sequelize.define('모델명', { /* 특성 */ }, { /* 옵션 */ });
{% endhighlight %}

`usercore` 테이블에 사용될 모델명과 특성을 추가해보자.

{% highlight javascript %}
var usercore = sequelize.define('usercore', {
		no : { type : Sequelize.INTEGER.UNSIGNED, primaryKey: true, autoIncrement: true}
		, id : { type : Sequelize.STRING(14) }
		, gems : { type : Sequelize.INTEGER(5).UNSIGNED, defaultValue: 0}
		, coins : { type : Sequelize.INTEGER.UNSIGNED, defaultValue: 0}
		, hearts : { type : Sequelize.INTEGER(3).UNSIGNED, defaultValue: 0}
		, highScore : { type : Sequelize.INTEGER.UNSIGNED, defaultValue: 0}
		, loginTime : { type : Sequelize.DATE, defaultValue: '2002-06-05 00:00:00', get:function(){var convertTime=new Date(this.getDataValue('loginTime')); return convertTime.getTime();}}
	}, { /* 옵션 */ });
{% endhighlight %}

특성을 정의하면특성서 사용될 수 있는 특징과 설명은 [공식문서](http://docs.sequelizejs.com/en/latest/docs/models-definition/#definition)의 definition부분을 참고하면 된다.
`usercore`에서 사용된 특징은 아래 표를 참조하자.

특징 | 설명
--- | ---
type | Datatype을 정의한다. 정수의 경우 0이하의 숫자를 사용하지 못하게하는 UNSIGEND 옵션이나 길이를 조절하면 숫자가 함께 사용된다.
primaryKey | RDB의 기본키로 지정할 때 사용한다. 저장되는 레코드를 고유하게 식별할 수 있는 값 중에 선택하게 된다.
autoIncrement | 사용자가 생성 시 자동으로 숫자가 증가하게 된다.
defaultValue | 기본값을 정의할 때 사용한다.
get | 값을 읽을 때 가공하기 위한 목적으로 사용한다. getter와 setter가 모두 사용가능하다.

이제 옵션을 추가하고 모델 정의를 마무리하자.

{% highlight javascript %}
 var usercore = sequelize.define('usercore', {
		no : { type : Sequelize.INTEGER.UNSIGNED, primaryKey: true, autoIncrement: true}
		, id : { type : Sequelize.STRING(14) }
		, gems : { type : Sequelize.INTEGER(5).UNSIGNED, defaultValue: 0}
		, coins : { type : Sequelize.INTEGER.UNSIGNED, defaultValue: 0}
		, hearts : { type : Sequelize.INTEGER(3).UNSIGNED, defaultValue: 0}
		, highScore : { type : Sequelize.INTEGER.UNSIGNED, defaultValue: 0}
		, loginTime : { type : Sequelize.DATE, defaultValue: '2002-06-05 00:00:00', get:function(){var convertTime=new Date(this.getDataValue('loginTime')); return convertTime.getTime();}}
	}, {
		timestamps: false,
		tableName: 'usercore' 
	});
{% endhighlight %}

`timestamps`는 true가 기본 값이다. 이를 허용하면 `createdAt`과 `updatedAt` 컬럼이 자동으로 추가된다. 

테이블명은 모델명을 기준으로 자동으로 작성되나 `tableName`을 통해서 직접 지정할 수 있다.  

위와 같은 내용을 `models` 폴더의 `usercore.js`을 생성하여 추가한다.

	.
	├── models
	│   └── index.js
	│   └── usercore.js

---

## 웹 어플리케이션 시작 시 자동 실행

마지막 단계로 웹 어플리케이션이 시작할 때 `models`를 작동하도록 해야한다. 처음에 한번 연결해두면 작동하는 동안에는 추가적인 연결이 없기때문에 편리하다.

해당 내용이 추가되어야하는 곳은 `bin/www`파일이다. 해당 내용 중 추가한 부분을 요약하면 아래와 같다. [^4]

{% highlight javascript %}
#!/usr/bin/env node

/**
 * Module dependencies.
 */

---(중략)---
var models = require("../models"); //추가한 부분.

---(중략)---
/**
 * Listen on provided port, on all network interfaces.
 */

// server.listen(port); //주석처리
server.on('error', onError);
server.on('listening', onListening);

//추가한 부분.
//sequelize의 싱크 작업을 시작하고 완료되면 설정된 포트를 통해서 통신 가능하도록 한다.
models.sequelize.sync().then(function () {
  server.listen(app.get('port'), function() {
    debug('Express server listening on port ' + server.address().port);
  });
});
{% endhighlight %}
	
먼저 `models`를 읽어드린 후 `sequelize.sync()`메서드를 실행하고 마치면 설정된 포트를 통해서 통신이 가능하도록 설정한 것이다.

---

## 실행 확인

이제 잘 작동하는지 확인해보자. 커맨드라인 툴에서 아래와 같이 입력한 후 브라우저에서 `http://localhost:3000/phpmyadmin`으로 접속한다.

{% highlight shell %}
node bin/www
{% endhighlight %}

커맨드라인 툴에는 아래와 같은 내용이 표시될 것이다.

{% highlight shell %}
Executing (default): CREATE TABLE IF NOT EXISTS `usercore` (`no` INTEGER UNSIGNED , `id` VARCHAR(14), `gems` INTEGER(6) UNSIGNED DEFAULT 0, `coins` INTEGER UNSIGNED DEFAULT 0, `hearts` INTEGER(4) UNSIGNED DEFAULT 0, `highScore` INTEGER UNSIGNED DEFAULT 0, `loginTime` INTEGER(10) UNSIGNED DEFAULT 0, PRIMARY KEY (`no`)) ENGINE=InnoDB;
Executing (default): SHOW INDEX FROM `usercore` FROM `myapp`
{% endhighlight %}

해당 내용은 정의한 내용에 따라서 테이블을 생성하는 SQL 쿼리문이다.

브라우저를 통해 phpmyadmin에 접속하면 `myapp` 데이터베이스에 `usercore`테이블이 생성된 것을 확인할 수 있습니다.
![usercore확인]({{"/images/usercore_table.jpg"}})

---

## 맺음말

긴 설명을 통해 Sequelize.js를 `myapp` 웹 어플리케이션에 적용했다. 다음 시간부터는 기능을 구현하도록 하겠다.

참고로 이 내용은 Sequelize에서 공식적으로 제공하는 예제에 자세히 설명된 것을 필요한 부분만 발췌하여 적용한 것이다. 더 자세히 알고 싶다면 [express-sample](https://github.com/sequelize/express-example) 레포지토리를 확인하기 바란다.

---


[^1]: 관계형 데이터베이스는 [일련의 정형화된 테이블로 구성된 데이터 항목들의 집합체](http://www.terms.co.kr/RDB.htm)로 MSSQL, MariaDB, MySQL, PostgreSQL, CUBRID 등이 이에 속한다. EXCEL 시트를 떠올리면 된다.

[^2]: SQL은 [데이터베이스에서 정보를 얻거나 갱신하기 위한 표준화된 언어](http://www.terms.co.kr/SQL.htm)이다. 데이터를 다루는 SQL문장을 DML(Data Manipulation Language)이라고 한다. DML은 SELECT, INSERT, UPDATE, DELETE로 이뤄진다.

[^3]: 다른 파일에서 모듈을 불러다 쓸 수 있는 일종의 라이브러리 개념이다. c#의 using같다고 생각하면 된다. 

[^4]: `bin/www`파일 내용 전체를 보고 싶다면 [github의 usercore.js](https://github.com/totuworld/FarmDefence_NodeServer/blob/master/models/usercore.js)를 참고하기 바란다.