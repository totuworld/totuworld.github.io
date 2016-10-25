---
layout: post
title:  "Azure 웹앱에서 SQL DB사용하기"
date:   2016-10-25 21:00:00
categories: nodejs azure webapps
comments: true
meta : nodejs azure webapps webservice sql database
description : Azure 웹앱에서 SQL 데이터베이스를 사용하는 방법을 알아본다.
image : 
  twitter: /images/webapp_tutorial_title_03.png
  facebook: /images/webapp_tutorial_title_03.png
publish : true
---

* content
{:toc}

## 들어가는 말

웹 서비스를 만들면 데이터베이스를 다루게된다.

이럴 때 MEAN이나 MERN 스택처럼 NoSQL를 활용해도 좋다.

핫하며 JSON을 그대로 저장할 수 도 있는 NoSQL 데이터베이스를 선택하면 Javascript와 찰떡궁합 매칭이 가능한 이점도 있다.

그래도 나처럼 배움이 늦은 사람에게는 엑셀 표처럼 보이는 SQL이 편하다.

> SQL아, 힘내!

---

## 데이터베이스 추가

Azure는 SQL 데이터베이스, MySQL, MariaDB, PostgreSQL 등 다양한 데이터베이스를 활용할 수 있다.

> NoSQL도 다양한 것을 지원하고 있다. 써드파티가 Azure에서 힘내고 있다.

그 중 SQL 데이터베이스와 MySQL을 활용하는 방법을 다루도록 하겠다.

### SQL 데이터베이스 생성

1. [Azure 포털로 접속](https://portal.azure.com)한다.


2. `새로 만들기` 클릭한다.

    ![새로만들기](/images/webapp_a01.png)


3. `SQL Database`를 검색한다.

    ![검색](/images/webapp_c01.png)

4. `SQL Database`를 선택한다.

    ![선택](/images/webapp_c02.png)

5. `데이터베이스 이름`을 입력한다.

    ![DB명입력](/images/webapp_c03.png)

6. SQL 데이터베이스 서버를 생성 한 경우가 아니라면 전 단계 메뉴에서 `서버`를 클릭하여 `새 서버 만들기`를 선택한다.

    이미 서버를 생성했다면 선택하고 8번 단계를 진행한다.

    ![서버](/images/webapp_c04.png)

7. `서버 이름`, `서버 관리자 로그인`, `암호`를 입력하고 `선택`을 클릭한다.

    ![새서버](/images/webapp_c05.png)

8. 생성된 서버를 선택하고 `만들기`를 클릭하여 SQL 데이터베이스를 생성한다.

    ![만들기](/images/webapp_c03_1.png)

배포가 완료되면 SQL 데이터베이스를 활용할 수 있게 된다.

### SQL 데이터베이스 구조

SQL 데이터베이스 생성과정을 보면 `서버`를 만들고 `데이터베이스 이름`을 입력하는 것을 볼 수 있다. 1개의 서버에 복수의 데이터베이스가 존재할 수 있기때문에 이런 과정을 거치는 것이다. 다시 말해 1개 서버에 각 서비스별로 데이터베이스를 등록하여 사용할 수 도 있다.

> 쉽게 설명하면 1개의 서류함에 다수의 서류철이 존재할 수 있는 것과 같다.

![서류함서류철](/images/document_ cabinet.png)

이를 확장하여 설명하면 데이터베이스가 서류철이라면 하나의 문서는 테이블로 볼 수 있다.

## MySQL 생성

1. [Azure 포털로 접속](https://portal.azure.com)한다.

2. `새로 만들기` 클릭한다.

    ![새로만들기](/images/webapp_a01.png)

3. `MySQL`를 검색한 뒤 원하는 제품을 선택한다.

    > 이 글에서는 테스트 용도이므로 20MB지원하는 ClearDB 제품을 선택했다.

    ![검색](/images/webapp_c06.png)

4. `데이터베이스 이름`, `위치` 등을 모두 입력하고 `가격 정책 계층`을 무료 등급인 수성으로 설정한다.

    ![정보입력](/images/webapp_c07.png)

5. `만들기`를 클릭하여 MySQL 데이터베이스를 생성한다.

    ![만들기](/images/webapp_c07_1.png)



한가지 주의할 점이 있다. `데이터베이스 유형`을 공용으로 설정했다면 해당 서비스에서 `설정` - `속성`으로 이동해 `호스트 이름`, `사용자 이름`, `암호`를 확인해야한다.

![정보확인](/images/webapp_c08.png){:width="150px"}



## 데이터베이스 사용

> 여기서부터는 지난 시간에 다룬 웹앱을 변경하여 설명한다(Node.js + Express.js).

웹앱이 데이터베이스에게 무슨 일을 시키려면 데이터베이스가 어디에 있고 어떻게 접근할지 알려줘야한다.

> 이를테면 구글 지메일에 접속하려면 gmail.com을 알아야하고 `아이디`와 `비밀번호`를 입력하는 것과 같은 절차다.

![DB연결](/images/webapp_c09.png)

간단하게 주소, 아이디, 비밀번호만 입력하면면 MSSQL, MySQL, MariaDB 등 DBMS를 가리지않고 사용할 수 있다면 좋겠지만 각 DBMS가 차이가 있다.

거기다가 DBMS에 어떤 일(검색, 변경, 제거 등)을 시키려면 SQL 문법으로 작성된 명령어를 입력해야한다.

```
SELECT ID, NAME, SALARY FROM CUSTOMERS;

SELECT * FROM CUSTOMERS WHERE SALARY > 150;
```

여기서는 MySQL과 MSSQL을 다루므로 각 DBMS 교체 비용을 줄이고 SQL 작성을 건너뛰기위해 ORM을 사용한다.

> 선택한 ORM은 sequelize.js 이다.

> 간결하게 웹앱이 DB를 사용해야하는 예제치고 되게 불필요해보이는 짓이지만 하기로한다. 주르륵

![ORM](/images/webapp_c09_1.png)


### 업데이트된 소스코드 적용하기

1. 아래 소스코드를 다운 받아 기존에 웹앱을 설정한 폴더에 덮어쓴다.
    
    > [AzureWebAppsTutorial-0.0.2 링크](https://github.com/totuworld/AzureWebAppsTutorial/archive/v0.0.2.zip)

    기존에 아무런 변경이 없다면 대상 폴더는 `AzureWebAppsTutorial-0.0.1`이다.

2. `AzureWebAppsTutorial-0.0.1` 폴더에서 아래 명령을 입력하여 sequelize, mysql, tedious 등 필요한 모듈을 설치한다.

{% gist /aea92ba28d3d2562817d76816da5021b installmodule %}

위 명령으로 package.json 내용중 dependencies에 해당하는 모듈이 모두 설치된다.

### 설정 변경

이제 설정을 변경해야한다. `config/config.json` 파일을 열어보면 크게 3가지 object가 보인다.

위에서부터 열거하면 `development`, `devMySQL`, `production`이다.

> 필요에따라 더 늘려도되고 변경해도 무관한 내용이다. 여기서는 예를 들어야해서 3가지로 기본 등록한 것이다.

각각의 설정값과 역할은 아래 표를 참고하자.

설정 key | 데이터 타입 | 설명
--- | --- | ---
host | string | 접속해야하는 DB주소
port | number | DB 접속시 사용되는 port 번호(1433 : MSSQL, 3306 : MySQL, MariaDB)
username | string | DB 접속시 사용되는 사용자명(외부로 공개될 프로젝트의 경우 `DBUSERNAME` 등의 환경변수로 관리하길 권한다. )
password | string | DB 접속시 사용되는 비밀번호(외부로 공개될 프로젝트의 경우 `DBPASSWORD` 등의 환경변수로 관리하길 권한다. )
dialect | string | ORM(여기서는 sequelize)가 접속하게되는 DBMS 종류
 
앞서 준비한 DB는 2가지 이다. 하나는 MSSQL과 같은 SQL Database 다른 하나는 MySQL.

SQL Database에 접근하는 내용은 `development`에 적고 MySQL에 필요한 내용은 `devMySQL`에 넣는다.

MySQL로 예를 들면 대략 이렇다.

{% gist /aea92ba28d3d2562817d76816da5021b devmysql.json %}

그럼 `production`에는 어떤 내용을 넣어야할까?

`production`은 Azure 웹앱에 소스코드가 적용될 때 `NODE_ENV`로 기본 설정되는 환경변수이다.

그러므로 `production`에 넣은 내용은 Azure 웹앱의 설정이 변경되지 않으면 기본으로 적용되는 설정이 된다.

여기서는 `development`나 `devMySQL`에 넣은 내용을 복사해서 붙여넣어도 무방하다.

> 서비스활 때 이렇게 차이를 두는 이유는 로컬 환경, 배포 전 stage 환경, 배포 환경이 모두 다를때 대응하기 위함이다.  

### 접속 테스트

자 이제 설정은 모두 끝났다.

소스코드를 웹앱에 적용하기 전에 잘 작동하는지 확인해보자.

#### development 환경 테스트 

`NODE_ENV`로 `development` 환경임을 지시하고 server.js 파일을 실행시키면 된다.

> 안전을 위해 username과 password를 환경변수로 전달하는 방법까지 함께 한 명령으로 필요에 따라 환경변수를 제거하여 실행한다.

{% gist /aea92ba28d3d2562817d76816da5021b runsqldb %}


#### devMySQL 환경 테스트 

`devMySQL`도 거의 비슷하다. 환경변수만 변경하면 된다.

{% gist /aea92ba28d3d2562817d76816da5021b runmysql %}


## 맺음말

위 과정을 크게 압축하면 `DB 설정`, `접속설정`에 불과하다.

테이블 디자인, 쿼리 최적화, 기본키, 외래키 등 관계형 데이터베이스가 많은 키워드를 쏟아낼 것이다. 이러한 내용은 훌륭한 강의가 많으니 찾아보길 바란다.