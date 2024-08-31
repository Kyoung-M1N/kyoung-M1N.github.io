---
title: [JPA]헷갈리는 개념들 정리
author: kymin
date: 2024-08-11 19:31
categories: [Spring]
tags: [spring, web, java]


---

## JPA?

JPA(Java Persistence API)는 자바의 ORM 기술 표준으로 사용되는 인터페이스의 모음을 의미한다.

말 그대로 인터페이스이기 때문에 구현된 것이 아닌 구현된 클래스와 데이터베이스의 테이블을 매핑하기 위한 프레임워크이며 JPA를 구현한 대표적인 오픈소스로는 Hibernate가 있다.

### ORM(Object-Relational Mapping)?

이름에서 알 수 있듯이 객체(Object)와 관계형 데이터베이스(Relational DataBase)에 대응관계를 부여(Mapping)한다는 의미로 객체를 데이터베이스의 테이블과 매핑하여 객체에 영속성을 부여해주는 것이다.

> 이 때, 영속성은 애플리케이션이 종료되어도 데이터가 사라지지 않고 저장되어 있는 상태를 말한다.

### Spring Data JPA?

Spring프레임워크에서 JPA를 이용하기 위한 프레임워크로 일반적인 JPA와 가끔씩 혼동되는 경우가 있다.

Spring Data JPA는 Spring Data라는 프레임워크에 JPA의 기능이 추가된 것으로 JPA가 객체와 데이터베이스의 테이블을 매핑해주는 것이라면 Spring Data는 데이터베이스에 접근하는 방식을 편리하게 제공해주는 프레임워크이다.

> 보통 자주 사용되는 Repository인터페이스를 이용하여 데이터베이스에 CRUD작업을 하는 것이 Spring Data가 제공하는 기능이다.
>
> 일반적인 JPA만 사용한다면 데이터베이스에 대한 작업을 수행할 때 `EntityManager`를 이용하게 되지만, Spring Data JPA를 사용하면 `Repository`를 이용하여 데이터베이스에 대한 작업을 수행하게 된다.

### JDBC(Java Database Connectivity)?

JDBC는 Java애플리케이션의 데이터를 데이터베이스에 저장, 업데이트하거나 데이터베이스의 데이터를 Java애플리케이션에서 사용할 수 있도록 하는 자바의 API이다.

Spring Data JDBC, Spring Data JPA 등의 프레임워크들이 생겨나며 JDBC를 직접 사용하는 경우가 많이 줄었지만 Spring Data JDBC, Spring Data JPA와 같은 프레임워크도 내부적으로는 JDBC를 사용하고 있다.

JDBC는 JDBC드라이버를 통해 자바 애플리케이션을 데이테베이스와 연결하고 통신한다. 이 때 통신 과정에서는 JDBC API를 통해 데이터베이스와 통신을 하게 된다.

JPA는 JDBC드라이버를 통해 SQL을 데이터베이스에 전달하는 방식으로 데이터베이스에 대한 작업을 수행한다.

### JPQL(Java Persistence Query Language)?

JPQL은 JPA가 제공하는 객체 지향 쿼리 언어로 SQL을 추상화한 것이다.

SQL을 추상화했기 때문에 특정 데이터베이스의 SQL에 종속되지 않으면서도 SQL의 문법과 매우 유사하며 데이터베이스의 테이블이 아닌 엔티티 객체를 대상으로 쿼리를 작성하는 것이 특징이다.

작성 시 기본 문자열로 작성되기 때문에 오류가 있더라도 컴파일 시 에러가 발생하지 않고서비스 동작 중에 런타임 에러가 발생하여 장애를 일으킬 가능성이 높다.

Spring Data JPA에서 제공하는 함수나 JPA에서 제공하는 API만으로는 세부적인 제약조건이나 검색조건이 포함된 쿼리를 사용하기 어렵기 때문에 복잡한 요구사항을 가지는 쿼리를 직접 작성하기 위해 사용된다.

### JPA의 동작

JPA는 데이터베이스의 테이블과 매핑된 엔티티 객체와 요구되는 작업에 대한 알맞은 쿼리문을 생성하거나, 개발과정에서 직접 작성된 JPQL을 분석하여 SQL로 변환하고 JDBC를 통해 DB와 통신하게 된다.

-----

##### 참고자료 :

[https://dbjh.tistory.com](https://dbjh.tistory.com/77)

[https://lealea.tistory.com](https://lealea.tistory.com/238)

[https://hstory0208.tistory.com](https://hstory0208.tistory.com/entry/JPA-JPA%EB%9E%80-%EA%B7%B8%EB%A6%AC%EA%B3%A0-Spring-Data-JPA)

[https://ittrue.tistory.com](https://ittrue.tistory.com/250)

[https://dev-coco.tistory.com](https://dev-coco.tistory.com/141)