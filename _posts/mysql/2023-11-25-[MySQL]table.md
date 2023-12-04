---
title: 테이블(Table)
author: kymin
date: 2023-11-25 16:20
categories: [MySQL]
tags: [database, sql]
---


### 테이블이란?

RDBMS에서 데이터를 저장하는 기본적인 구조이며 데이터베이스를 생성해도 자동으로 생성되는 것이 아니기 때문에 직접 생성해주어야 한다.

테이블은 한 개 이상의 column과 0개 이상의 row로 구성된다.

- Column

  데이터의 타입 및 크기를 가지고 있으며 특정 종류의 데이터를 나타낸다.

- Row

  여러 Column의 조합으로 구성되며 레코드라고도 불린다.

  기본키(PK)로 인해 구분되며 기보니는 중복이 허용되지 않는다.

- Field

  Row와 Column의 교차점으로 데이터가 존재하는 자리이다. 데이터가 없으면 NULL값을 가지고 있다.

### 테이블 생성

- DB 내의 테이블 목록 출력

  데이터베이스에 접속 후에 아래의 명령어를 입력

  ```sql
  show tables;
  ```

- 테이블 생성

  아래의 명령어를 통해 테이블을 생성

  ```sql
  CREATE TABLE [테이블이름](
  	[필드이름] [필드타입] [제약조건],
  	[필드이름] [필드타입] [제약조건],
  	[필드이름] [필드타입] [제약조건],
  	...
  	
  );
  ```

  제약조건은 필수가 아니며 필드 이름과 필드 타입을 한 쌍으로 하고 각 필드들을 쉼표(,)로 구분하여 테이블을 생성한다.

- 테이블 구조 출력

  아래의 명령어로 테이블의 구조를 출력

  ```sql
  describe [테이블 이름];
  ```

  `describe`는 `desc`축약어로도 사용 가능

### 테이블 삭제

- 테이블 삭제

  아래의 명령어로 테이블을 삭제

  ```sql
  drop table [테이블 이름];
  ```

  

-----

##### 참고자료 : 

[cloud.google.com/mysql](https://cloud.google.com/mysql?hl=ko)

[boostcourse.org/web326](https://www.boostcourse.org/web326/)

[tcpschool.com/mysql](https://tcpschool.com/mysql/mysql_basic_create)
