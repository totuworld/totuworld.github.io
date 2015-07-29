---
layout: post
title:  "Sequelize.js 적용"
date:   2015-07-28 22:00:00
categories: Nodejs서버강좌 Nodejs Sequelize
comments: true
meta : Node.js서버강좌 - 3
description : Sequelize.js를 이용해 MariaDB와 Node.js를 연결하여 DB에 접근하도록 한다.
publish : false
---

* content
{:toc}

## 들어가는 말

앞서 VM에 MariaDB를 설치했고 Express 프로젝트를 생성했다. 이번에는 사용자 정보나 상점, 친구, 결제 등 주요한 정보를 저장하게되는 MariaDB에 Express에서 어떻게 접근하여 다루는지 살펴보자.   

---

## ORM을 왜 사용하는가?

관계형 데이터베이스(Relational Database, RDB [^1] )를 사용하면 데이터베이스의 데이터를 조회 및 조작하기 위해서 SQL [^2] 를 작성한다. 간단한 SQL 문장을 작성하는 것이야 어렵지 않지만 사용하는 언어와 사용되는 데이터베이스에 따라서 특성이 있어서 불편한 점이 많다.

그래서 등장한 것이 ORM(Object-relational mapping)이다. ORM은 통역으로 생각하면된다. 프로그래밍 언어로 직접 데이터베이스에 말할 수 없으니 중간에 통역인 ORM이 데이터베이스가 알아들을 수 있는 언어(SQL 등)로 번역하여 전달하는 것이다.

Node.js의 ORM은 [Sequelize.js](http://docs.sequelizejs.com/en/latest/), [Bookshelf.js](http://bookshelfjs.org), [node-orm2](http://dresende.github.io/node-orm2/) 등이 있다. 여기서는 Sequelize.js를 사용하도록 한다.      

---

## Sequelize.js 설치

커맨드라인 툴로 `myapp` 프로젝트가 위치한 폴더에서 아래와 같이 입력한다.

	npm install --save sequelize
	npm install --save mysql

Sequelize.js는 ORM으로 필요하고 mysql은 Sequelize.js로 MySQL, MariaDB를 다루기 위해서 필요한 모듈이다.

---

## 준비 작업

* MariaDB에 사용할 데이터베이스를 생성한다.

1. 생성해둔 VM을 `vagrant up` 명령으로 실행한다.
2. `http://localhost:8080/phpmyadmin` 주소로 접속한다. (Username : root, Password : 1234)
3. phpMyAdmin으로 Databaes 탭을 클릭 한 후 myapp 데이터베이스 생성를 생성한다.
![MakeDB]({{"/images/make_db.jpg"}})


* `myapp` 프로젝트에 `config` 폴더를 생성 한 후 아래와 같이 `config.json`파일을 작성한다.  

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

## 테이블 정의하기

1. myapp 테이블에 추가할 UserCore 테이블 작성
2. 실행하여 생성되었는지 확인


``` json
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
```

---

[^1] : 관계형 데이터베이스는 [일련의 정형화된 테이블로 구성된 데이터 항목들의 집합체](http://www.terms.co.kr/RDB.htm)로 MSSQL, MariaDB, MySQL, PostgreSQL, CUBRID 등이 이에 속한다. EXCEL 시트를 떠올리면 된다.

[^2] : SQL은 [데이터베이스에서 정보를 얻거나 갱신하기 위한 표준화된 언어](http://www.terms.co.kr/SQL.htm)이다. 데이터를 다루는 SQL문장을 DML(Data Manipulation Language)이라고 한다. DML은 SELECT, INSERT, UPDATE, DELETE로 이뤄진다.