---
title: 설치 및 시작
author: kymin
date: 2023-11-20 17:31
categories: [MySQL]
tags: [database, sql]
---
# 

## DBMS?

DBMS(DataBase Management System)는 데이터를 만들고 저장 및 관리하는 기술 또는 해당 기능을 제공하는 응용 프르그램을 의미한다.

MySQL, MariaDB, PostgreSQL등 여러가지 종류가 있다.

## MySQL?

MySQL은 가장 많이 사용되는 오픈소스 DBMS로 SQL이라고 하는 구조화된 쿼리 언어를 사용하여 데이터를 정의, 조작, 제어, 쿼리할 수 있다.

MySQL은 관계형 데이터베이스 관리 시스템(RDBMS)이라는 데이터베이스 카테고리에 속한다. 관계형 데이터베이스는 데이터가 하나 이상의 열과 행의 테이블(또는 '관계')에 저장되어 서로 다른 데이터 구조가 어떻게 관련되어 있는지 쉽게 파악하고 이해할 수 있도록 사전 정의된 관계로 데이터를 구성하는 데이터베이스다.

## 설치

- HomeBrew 설치

  https://brew.sh/에서 HomeBrew를 설치하고 아래의 명령어를 통해 버전 정보를 출력하여 설치 여부를 확인

  ```shell
  brew -v
  ```

- MySQL 설치

  아래의 명령어를 실행하여 MySQL을 설치

  ```shell
  brew install mysql
  ```

## MySQL 서버 실행

- 서버 실행

  터미널에서 아래의 명령어를 실행

  ```shell
  mysql.server start
  ```

  실행 명령이 성공적으로 작동한다면 아래와 같은 메시지가 출력

  ```shell
  Starting MySQL
  .SUCCESS!
  ```

- 데몬으로 실행

  데몬(daemon)은 운영체제 상에서 프로그램을 사용자가 직접적으로 제어하지 않고, 백그라운드에서 돌면서 여러 작업을 하는 프로그램을 말한다. 즉, 시스템의 기능을 제공하거나 백그라운드에서 항시 실행되는 프로그램이다.

  아래와 같은 명령어로 HomeBrew가 제공하는 기능을 이용하여 MySQL을 데몬 형태로 실행

  ```shell
  brew service start mysql
  ```

  아래의 명령어로 서비스를 재시작

  ```shell
  brew service restart mysql
  ```

  또한 아래의 명령어로 데몬으로 실행되고 있는 프로그램들의 목록을 출력

  ```shell
  brew services list
  ```

## MySQL 서버 종료

- 서버 종료

  서버 실행과 마찬가지로 아래의 명령어를 통해 서버를 종료

  ```shell
  mysql.server stop
  ```

  종료 명령이 성공적으로 작동한다면 아래와 같은 메시지가 출력

  ```shell
  Shutting down MySQL
  .SUCCESS!
  ```

- 데몬으로 실행 중인 서비스 종료

  HomeBrew가 제공하는 기능을 이용하여 MySQL을 데몬 형태로 실행한 상태라면 아래의 명령어를 통해 데몬 형태로 실행되고 있는 MySQL을 종료

  ```shell
  brew services stop mysql
  ```

  

-----

##### 참고자료 : 

[boostcourse.org/web326](https://www.boostcourse.org/web326/)

[cloud.google.com/mysql](https://cloud.google.com/mysql?hl=ko)

[spidyweb.tistory.com](https://spidyweb.tistory.com/222)
