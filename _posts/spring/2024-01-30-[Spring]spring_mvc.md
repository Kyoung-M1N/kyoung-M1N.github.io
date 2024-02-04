---
title: Spring MVC
author: kymin
date: 2024-01-30 16:53
categories: [Spring]
tags: [spring, web, java, mvc]

---

## 스프링 MVC

### MVC(Model-View-Controller)?

MVC는 Model, View, Controller의 약자로 소프트웨어의 비즈니스 로직과 화면을 구분하는데 중점을 두고 있는 소프트웨어 디자인 패턴이다.

- Model

  데이터를 가진 객체를 의미하며 컨트롤러가 받은 요청의 결과로써 뷰에게 전달된다.

- View

  사용자에게 시각적으로 보여지는 부분을 말하며 모델로부터 데이터를 제공받아 사용자에게 제공한다.

- Controller

  사용자의 요청을 처리하고 소프트웨어의 동작 흐름을 제어한다.

### MVC 적용하기 위한 규칙

- model은 controller와 view에 의존하지 않아야 한다.

  model에 controller와 view에 관련된 클래스를 import하여 사용하지 않는다.

- view는 model에만 의존하며 controller에 의존하지 않는다.

  view에서 model에 대한 클래스를 import하여 사용할 수 있지만 controller에 관련된 클래스는 import하여 사용하지 않는다.

- view는 model로부터 데이터를 가져올 경우에는 사용자마다 다르게 보여지는 데이터에 대해서만 가져온다.

  변하지 않는 데이터에 대해서는 view에서 자체적으로 구현하거나 상수를 따로 관리하도록 한다.

- controller는 model과 view에 의존할 수 있다.

  controller는 클라이언트의 응답을 처리한 결과로 model을 생성하고 view를 호출하여 model을 전달한다.

  따라서 controller에서 model과 view에 관련된 클래스를 import하여 사용할 수 있다.

- view가 model로부터 데이터를 전달받을 경우에는 controller를 통해 전달받아야 한다.

  model은 controller가 비즈니스로직을 호출하여 얻은 결과이므로 model을 view에서 따로 생성하지 않고 controller에서 view를 호출하여 model을 전달한다.

### Spring MVC

MVC 패턴을 이용하여 Web Application을 개발할 수 있도록 스프링이 제공하는 프레임워크이다.

따라서 Spring MVC는 디자인 패턴을 의미하는 것이 아니라 프레임워크를 의미한다.

## Spring MVC 구성과 동작

### 구성

- DispatcherServlet

  클라이언트의 모든 요청을 처리하는 Front Controller로 동작하며 클라이언트의 요청에 따라 적절한 컨트롤러를 매핑하고 뷰를 반환하는 역할을 한다.

  스프링 프로젝트가 실행될 때 자동으로 DispatcherServlet이 서블릿으로 등록되며 DispatcherServlet을 모든 경로("/")에 대해 매핑한다.

- HandlerMapping

  요청을 처리할 컨트롤러를 탐색하는 역할을 한다.

- HandlerAdapter

  클라이언트의 요청과 매핑된 컨트롤러의 실행을 요청하고 컨트롤러가 반환하는 결과값을 DispatcherServlet에게  ModelAndView 객체로 전달한다.

- Controller(handler)

  사용자의 요청을 전달받아 필요한 로직을 호출하여 실행시키는 역할을 하며 로직의 결과를 모델로써 HandlerAdapter에게 전달한다.

- ViewResolver

  DispatcherServlet에게 전달받은 내용을 통해 컨트롤러로부터 받은 로직의 처리 결과를 반영할 View를 탐색하여 반환한다.

> 서블릿(Servlet)?
>
> 서블릿은 클라이언트의 요청을 처리하도록 특정 규약에 맞춰 Java 코드로 작성하는 클래스 파일이다.
>
> Spring MVC 내부에서는 서블릿을 기반으로 웹 애플리케이션을 동작한다.

### 동작 과정

- 클라이언트가 브라우저를 통해 서버에 요청
- Dispatcher Servlet이 요청된 URL에 매핑된 핸들러(컨트롤러)를 조회
- Dispatcher Servlet이 찾은 핸들러(컨트롤러)를 실행할 핸들러 어댑터를 조회
- 핸들러 어댑터가 핸들러(컨트롤러)를 실행
- 핸들러(컨트롤러)가 작업에 필요한 로직을 호출하여 작업을 진행
- 핸들러 어댑터가 핸들러가 반환한 정보를 ModelAndView 객체로 Dispatcher Servlet에게 반환
- Dispatcher Servlet가 알맞은 View resolver를 찾아 호출
- View resolver가 View객체를 반환
- View객체가 View를 랜더링하며 화면을 표시



---

##### 참고자료 :

[https://velog.io/@shi9476](https://velog.io/@shi9476/%EC%8B%A4%EB%AC%B4%EC%97%90%EC%84%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-Spring-MVC-%EA%B5%AC%EC%A1%B0-%EC%88%9C%EC%84%9C-%EC%82%AC%EC%9A%A9-%EC%99%84%EB%B2%BD-%EC%A0%95%EB%A6%AC)

[https://velog.io/@do_dam](https://velog.io/@do_dam/Spring-MVC%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80-%EC%8A%A4%ED%94%84%EB%A7%81-MVC-%EA%B5%AC%EC%A1%B0-%EC%9D%B4%ED%95%B4)

[https://ss-o.tistory.com/160](https://ss-o.tistory.com/160)

[https://www.youtube.com](https://www.youtube.com/watch?v=ogaXW6KPc8I&t=29s)