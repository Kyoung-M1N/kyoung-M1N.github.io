---
title: 스프링빈
author: kymin
date: 2023-12-10 18:47
categories: [Spring]
tags: [spring, web, java]
---

## 스프링빈이란?

스프링빈은 스프링 컨테이너가 관리하는 자바 객체를 의미한다.

기본적으로 객체는 스프링 컨테이너에 싱글톤으로 등록되기 때문에 다른 객체들에서 스프링빈으로 등록된 객체를 호출하게 되면 스프링 컨테이너는 해당 객체 대한 인스턴스를 자동으로 넘겨주게 되고 객체들은 같은 인스턴스를 사용하게 된다.

스프링빈의 등록을 통해 의존성 주입, 제어의 역전 등의 구현을 할 수 있다.

### 의존성 주입(Dependency Inejction)

의존성 주입은 외부에서 클라이언트에게 서비스를 제공(주입)하는 것을 의미한다. 즉, 객체가 필요로 하는 어떤 것을 외부에서 전달해주는 것을 말한다.

아래의 코드는 `Member`와 `MemberRepository`라는 클래스를 이용하여 회원가입을 하는 역할의 `MemberService`클래스와 회원의 정보를 사용자에게 전달하는 `MemberController`클래스이다.

두 클래스는 `MemberRepository`를 사용하지만 해당 클래스에 대한 인스턴스는 각각의 클래스에서 선언되었기 때문에 `MemberService`와 `MemberController`는 다른 인스턴스를 사용하고 있다.

```java
public MemberService {
  private final MemberRepository memberRepository = new MemberRepository();
  
  public void signUp(Member member) {
    memberRepository.save(member);
  }
}
```

```java
public MemberController {
  private final MemberRepository memberRepository = new MemberRepository();
  
  public Member printInfomation(Member) {
    memberRepository.findId(member);
  }
}
```

이 때 `MemberRepository`에서 별도의 DB에 회원정보를 저장하는 경우가 아니라면 `MemberService`를 통해 회원가입을 한 뒤에 `MemberController`는 방금 회원가입을 한 정보를 조회할 수 없다는 문제가 생긴다.

따라서 아래와 같이 생성자를 통해 외부에서 선언된 공통된 인스턴스 파라미터로 받아 해결할 수 있고, 이를 의존성 주입이라고 한다.

```java
public MemberService {
  private final MemberRepository memberRepository;
  
  public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
  }
  
  public void signUp(Member member) {
    memberRepository.save(member);
  }
}
```

```java
public MemberController {
  private final MemberRepository memberRepository = new MemberRepository();
  
  public MemberController(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
  }
  
  public Member printInfomation(Member) {
    memberRepository.findId(member);
  }
}
```

> 의존성을 주입하는 방법에는 필드 주입, setter 주입, 생성자 주입 이렇게 3가지 방법이 있지만, 의존관계가 실행중에 동적으로 변하는 경우는 거의 없으므로 생성자 주입을 권장한다.

### 제어의 역전(Inversion Of Control)

제어의 역전이란 객체의 생성 및 제어권을 사용자가 아닌 스프링에게 맡기는 것이다.

기존에는 사용자가 new연산을 통해 객체를 생성하고 메소드를 호출했다면, 스프링빈을 등록하여 객체의 관리를 스프링 컨테이너가 담당하게 된다.

이 때 다른 객체에서 스프링빈으로 등록된 객체를 호출하게 되면 스프링 컨테이너는 해당 객체 대한 인스턴스를 자동으로 넘겨준다.

따라서 사용자는 직접 new를 이용해 생성한 객체를 사용하지 않고, 스프링에 의하여 관리되는 자바 객체를 사용하게 된다.

## 스프링빈 등록

### 컴포넌트 스캔

컴포넌트 스캔방식은 스프링이 `@Component`어노테이션을 자동으로 스캔하여 해당 객체를 스프링빈으로 등록하는 방법이다.

`@Controller`, `@Service`, `@Repository`어노테이션은 `@Component`를 포함하기 때문에  `@Component`와 마찬가지로 스프링빈으로 등록하는 데에 사용된다.

이 때, 아래와 같이 `@Autowired`가 생성자에 있으면 연관된 객체를 스프링이 스프링 컨테이너에 찾아서 넣어준다.

> @Autowired 를 통한 의존성 주입은 스프링이 관리하는 객체에서만 동작하며, 스프링 빈으로 등록하지 않고 내가 직접 생성한 객체에서는 동작하지 않는다.

```java
package com.spring_training.controller;

import com.spring_training.service.MemberService;
import org.springframework.stereotype.Controller;
import org.springframework.beans.factory.annotation.Autowired;

@Controller	// MemberController를 스프링빈으로 등록
public class MemberController {
    private final MemberService memberService;

    @Autowired  // 스프링빈으로 등록된 MemberService의 인스턴스를 자동으로 전달
    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }
}
```



### 자바 코드로 등록

스프링빈을 자바 코드를 이용하여 등록하는 방법은 `@Configuration`어노테이션을 통해 스프링빈을 등록하는 역할을 하는 클래스를 지정하고, 해당 클래스 내부에 `@Bean`어노테이션과 함께 스프링빈으로 등록할 객체의 생성자를 호출하는 방식이다.

```java
// SpringConfig.java
package com.spring_training;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import com.spring_training.service.MemberService;
import com.spring_training.repository.MemberRepository;
import com.spring_training.repository.MemoryMemberRepository;

@Configuration	//스프링빈을 등록하는 클래스로 지정
public class SpringConfig {
    @Bean	//스프링빈 등록
    public MemberService memberService() {
        return new MemberService(memberRepository());
    }

    @Bean	//스프링빈 등록
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
}
```




-----

##### 참고자료 :

[https://spring.io/guides](https://spring.io/guides)

[inflearn - 스프링 입문](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard)