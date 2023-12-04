---
title: 스프링 시작하기
author: kymin
date: 2023-11-31 19:31
categories: [Spring]
tags: [spring, web]
---
# 스프링

### 스프링(Spring)

스프링 프레임워크(Spring Framework)는 자바 플랫폼을 통해 오픈소스 어플리케이션을 만들기 위한 프레임워크로 줄여서 스프링으로 불린다. 현재 대한민국 공공기관의 웹 서비스 개발 시 사용을 권장하고 있는 전자정부 표준 프레임워크의 기반 기술로서 사용되고 있다.

### 스프링의 장점

- **경량 컨테이너**

  스프링 프레임워크가 자바 객체를 직접 관리한다. 즉, 각 객체들의 생성, 소멸과 같은 라이프 사이클을 관리하며, 스프링으로부터 필요한 객체를 얻어올 수 있다.

- **POJO(Plain Old Java Object) 프로그래밍**

  순수 Java 만을 통해서 생성한 객체를 사용하여 프로그래밍이 진행되기 때문에 특정 기술이나 환경에 종속되지 않고 객체지향 설계를 제한없이 적용할 수 있으며, 코드가 단순해져 테스트와 디버깅 또한 쉬워진다.

- **제어 반전(IoC : Inversion of Control)**

  컨트롤의 제어권이 사용자가 아니라 프레임워크에 있어서 필요에 따라 스프링에서 사용자의 코드를 호출한다.

- **의존성주입(DI : Dependency Injection)**

  각각의 계층이나 서비스들 간에 의존성이 존재할 경우 프레임워크가 서로 연결시켜준다.

- **관점지향 프로그래밍(AOP: Aspect-Oriented Programming)**

  트랜잭션이나 로깅, 보안과 같이 여러 모듈에서 공통적으로 사용하는 기능의 경우 해당 기능을 분리하여 관리할 수 있다.

### 스프링 부트(Spring Boot)

스프링 프레임워크는 기능이 많은 만큼 환경설정이 복잡한 편이다. 이에 어려움을 느끼는 사용자들을 위해, 스프링 프레임워크를 사용하기 위한 설정의 많은 부분을 자동화하여 사용자가 정말 편하게 스프링을 활용할 수 있도록 돕는것이 바로 스프링 부트(Spring Boot)이다.

# VSCode에서 스프링 시작하기

먼저 편집기인 visual studio code를 아래의 링크에서 다운로드 한다.

https://code.visualstudio.com/download

다음으로 visual studio code를 실행한 뒤 아래의 extension을 설치한다.

- 자바 : Extension Pack for Java
- 스프링 : Spring Boot Extension Pack

### 스프링 프로젝트 생성

- 웹에서 생성

  스프링 부트 스타터 사이트[https://start.spring.io](https://start.spring.io)로 이동하여 스프링 프로젝트를 생성

  다운로드된 파일을 압축해제 후 IDE, 또는 VSCode에서 열기

- VSCode에서 생성

  `commend+shift+p`로 명령어 팔레트를 실행 후 spring initializr 실행

### 빌드하고 실행하기

- 빌드

  스프링 프로젝트가 존재하는 경로로 이동 후 아래의 명령어를 통해 프로젝트를 빌드명령을 실행

  ```shell
  ./gradlew (clean) build
  ```

  빌드가 성공적으로 완료되면 스프링 프로젝트의 `build/libs`에 `<프로젝트 이름>-<프로젝트 버전>.jar`파일이 생성된다.

  `clean`옵션을 추가하면 기존에 있던 빌드파일을 지우고 빌드를 진행한다.

  > `clean`명령만 단독으로도 사용 가능하다.	

- 실행

  스프링 프로젝트의 `build/libs`로 이동하여 빌드한 파일을 아래의 명령어로 실행

  ```shell
  java -jar <빌드한 파일>
  ex) java -jar spring_study-0.0.1-SNAPSHOT.jar
  ```

  서버에 배포를 하게 될 경우에는 빌드한 파일을 서버로 이동시킨 후 실행만 해주면 된다.

  > 당연히 서버에 JRE가 설치되어야 한다.

  

-----

##### 참고자료 :

[https://spring.io/guides](https://spring.io/guides)

[ wikipedia - 스프링 프레임워크](https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%94%84%EB%A7%81_%ED%94%84%EB%A0%88%EC%9E%84%EC%9B%8C%ED%81%AC)

[inflearn - 스프링 입문](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard)