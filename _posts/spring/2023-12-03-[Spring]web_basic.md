---
title: 스프링 웹 기초
author: kymin
date: 2023-12-03 17:18
categories: [Spring]
tags: [spring, web, java]
---

## welcome page

### Index.html 리턴

welcome page는 스프링부트가 기본적으로 제공하는 기능으로 스프링 프로젝트를 실행했을 때 가장 먼저 보이게 되는 페이지이다.

기본적으로 `resource/static`의 `index.html`파일을 탐색하여 자동으로 welcome page를 띄우지만 없을 경우 `resource/templates`의 `index.html`을 welcome page로 띄운다.

## 정적 컨텐츠

### 정적 컨텐츠 리턴

정적컨텐츠로 리턴할 html파일은 보통 `resource/static`에 존재한다.

url을 통해 페이지를 요청 받았을 때 스프링 컨테이너에 관련 컨트롤러가 존재하지 않을 경우 스프링에 내장된 톰캣이 html파일을 불러와서 보여준다.

컨트롤러가 없고 단순히 html파일만 페이지에 출력하기 때문에 페이지를 구성하는 요소들은 변경될 수 없다.

## 템플릿 엔진과 html

### 템플릿엔진

템플릿엔진은 지정된 템플릿 양식과 데이터가 합쳐져 HTML 문서를 출력하는 소프트웨어이다.

웹페이지를 호출하면 html로 작성되어있는 템플릿에 템플릿엔진이 데이터를 전달하여 데이터가 반영된 html파일을 리턴한다.

Freemarker, Thymeleaf, JSP 등이 있다.

### Templated페이지 리턴

템플릿이 되는 html파일은 보통 `resource/templates`에 존재한다.

url을 통해 페이지를 요청 받았을 때 스프링 켄테이너가 요청된 url에 mapping된 컨트롤러를 탐색하고, 컨트롤러는 model을 통해 view를 구성하기 위해 필요한 값을 전달한 뒤 해당 페이지를 리턴한다.

이 때 컨트롤러에서 해당 페이지의 이름을 문자열로 리턴하면 Thymeleaf의 viewResolver가 알아서 해당 html파일을 찾아서 페이지에 띄우게 된다.

컨트롤러가 전달하는 model에 따라 페이지의 내용이 변경될 수 있다.

## API

### ResponseBody 리턴

함수에 `@ResponseBody` 어노테이션을 사용하면 스프링 컨테이너가 viewResolver를 사용하지 않고 HttpMessageConverter가 동작하게 된다. 

이 때 문자열을 그대로 리턴하면 `StringHttpMessageConverter`가 동작하여 http의 body에 문자열을 그대로 리턴한다.

### JSON 리턴

함수에 `@ResponseBody` 어노테이션을 사용하고 객체를 리턴하면 `MappingJackson2HttpMessageConverter`가 동작하여 http의 body에 json파일을 리턴한다.

-----

##### 참고자료 :

[https://spring.io/guides](https://spring.io/guides)

[inflearn - 스프링 입문](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard)