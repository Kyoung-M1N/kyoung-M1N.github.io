---
title: 디렉터리 구조
author: kymin
date: 2023-12-14 19:46
categories: [Etc, Architecture]
tags: [architecture]
---

## 디렉터리 구조란?

파일 시스템에서 파일 및 하위 디렉터리를 조직화하는 방식을 나타내며, 프로젝트나 소프트웨어 애플리케이션 등 다양한 컴퓨터 프로그램에서 사용된다.

자바에서는 디렉터리의 구분으로 패키지를 구분하기 때문에 패키지 구조라고도 불린다.

효율적인 파일 관리, 코드 구성, 자원 관리 등을 위해 다양한 디렉터리 구조가 사용되며, 디렉터리 구조의 설계는 프로젝트의 이해도를 높이고 유지보수성을 향상시킨다.

가장 일반적으로 사용되는 디렉터리 구조로는 계층형 구조와 도메인형 구조가 있다.

### 클래스의 분리

디렉터리 구조를 적용하기 위해서는 역할에 따라 클래스(파일)을 분리하고 디렉터리(패키지)로 묶어서 관리해야 한다.

주로 사용되는 클래스들의 이름과 역할은 아래와 같다.

- controller

  MVC패턴에 포함되는 개념으로 model과 view의 중계역할을 한다.

  주로 사용자의 요청을 처리 한 후 지정된 뷰에 모델 객체를 넘겨주는 역할을 수행하며 데이터를 주고 받거나 웹 요청을 처리한다.

- service

  비즈니스 로직의 구현을 위한 기능들을 정의하거나 구현하는 객체이다.

- impl

  인터페이스로 정의된 기능들을 세부적으로 구현하는 구현체를 의미한다.

- repository

  데이터베이스에 접근하여 데이터를 가져오고 필요한 곳(주로 서비스)에서 호출하여 데이터를 전달한다.

  주로 인터페이스로 정의되며 구현체가 따로 존재한다.

  데이터의 I/O에 대한 구현 함수가 존재한다.

- entity

  실제 데이터베이스와 mapping되는 객체를 의미한다.

- domain

  소프트웨어로 해결하고자하는 문제 영역을 의미한다.

  서비스를 구현하는 데에 필요한 객체들로 주로 정보를 가지고 있다.(ex.회원정보, 상품정보 등)

- DTO(Data Transfer Object)

  계층간에 데이터를 전달하는 역할을 하는 객체를 의미하며, 로직을 가지지 않고 전달하려는 데이터에 대한 변수와 `getter`, `setter`메서드만을 가지고 있다.

- DAO(Data Access Object)

  데이터베이스의 데이터에 접근하기 위한 객체를 의미하며, 직접 데이터베이스에 접근하여 데이터를 삽입, 삽제, 조회하는 등의 기능을 수행한다.

  데이터베이스에 접근하기 위한 로직과 비즈니스 로직을 분리하기 위해 사용한다.

- VO(Value Object)

  특정 값을 표현하기 위해 read-only속성을 가진 객체를 의미하며, 표현하려는 값에 대한 변수와 `getter`메서드만을 가지고 있다.

> **Repository와 DAO의 차이**
>
> Repository는 entity를 db의 테이블에 매핑하여 entity를 통해 데이터베이스에 대한 작업을 수행
>
> DAO는 SQL문을 매핑하여 직접 데이터베이스에 대한 작업을 수행
>
> 따라서 Repository에서는 DAO를 여러개 선언하여 Service에 필요한 데이터 종합한 뒤에 DTO를 통해 전달하기도 한다.

> **Entity와 Domain의 차이**
>
> Entity는 데이터베이스의 테이블과 매핑되어 데이터에 대한 작업을 수행하는 객체이며 별도의 로직이 존재하지 않는다.
>
> Domain은 비즈니스 로직의 구현을 위한 기능들을 정의하거나 구현하는 객체로 비즈니스 로직이 존재하며

## 계층형 구조

계층형 구조는 `controller`, `service`, `repository`등, 각 애플리케이션 계층을 대표하는 디렉터리를 기준으로 코드가 구성된다.

```
└── src
    ├── main
    │   ├── java
    │   │   └── com/example/demo
    │   │       ├── DemoApplication.java
    │   │       ├── config
    │   │       ├── controller
    │   │       ├── domain
    │   │       ├── repository
    │   │       ├── service
    │   │       ├── security
    │   │       └── exception
    │   └── resources
    │       └── application.properties
```



## 도메인형 구조

도메인이란 소프트웨어로 해결하고자하는 문제 영역을 의미하며, 도메인형 구조는 각 계층이 아닌 도메인별로 코드가 구성된다.

```
└── src
    ├── main
    │   ├── java
    │   │   └── com/example/demo
    │   │       ├── DemoApplication.java
    │   │       ├── domain
    │   │       │   ├── coupon
    │   │       │   │   ├── controller
    │   │       │   │   ├── domain
    │   │       │   │   ├── exception
    │   │       │   │   ├── repository
    │   │       │   │   └── service
    │   │       │   ├── member
    │   │       │   │   ├── controller
    │   │       │   │   ├── domain
    │   │       │   │   ├── exception
    │   │       │   │   ├── repository
    │   │       │   │   └── service
    │   │       │   └── order
    │   │       │       ├── controller
    │   │       │       ├── domain
    │   │       │       ├── exception
    │   │       │       ├── repository
    │   │       │       └── service
    │   │       └── global
    │   │						├── auth
    │   │						├── common
    │   │						├── config
    │   │						└── util
    │   └── resources
    │       └── application.properties
```



계층형 구조와 도메인형 구조는 구조만 다를뿐 동작의 흐름은 아래와 같이 비슷하게 진행된다.

```
1. 클라이언트 요청 => Controller 호출
2. Controller => Service 호출
3. 구현체를 통한 비즈니스 로직 작동 (repo의 db접근 등이 발생)
4. Service => Controller 호출
5. Controller => 클라이언트 응답
```



## 계층형vs도메인형

계층형 구조는 해당 프로젝트에 이해가 상대적으로 낮아도 전체적인 구조를 빠르게 파악할 수 있는 장점이 있지만, 디렉터리에 클래스들이 너무 많이 모이게 되는 단점이 있다.

도메인형 구조는 관련된 코드들이 응집해 있는 장점이 있지만, 프로젝트에 대한 이해도가 낮을 경우 전체적인 구조를 파악하기 어렵다는 단점이 있다.

프로젝트마다 또는 코드를 작성하는 사람들 마다 선호하는 구조에 대한 차이가 있지만 대체로 요구기능(도메인)이 적은 경우에는 계층형 구조를 사용하고 많은 도메인이 요구되는 경우나 도메인이 추가될 가능성이 있는 경우에는 도메인형 구조를 사용한다.

-----

##### 참고자료 :

[우아한 기술블로그](https://techblog.woowahan.com/2647/)

[cheese10yun.github.io](https://cheese10yun.github.io/spring-guide-directory/)

[velog.io/@jsb100800](https://velog.io/@jsb100800/Spring-boot-directory-package)

[https://velog.io/@devmizz](https://velog.io/@devmizz/Project-Package-Structure)

[velog.io/@jybin96](https://velog.io/@jybin96/Controller-Service-Repository-%EA%B0%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C)
