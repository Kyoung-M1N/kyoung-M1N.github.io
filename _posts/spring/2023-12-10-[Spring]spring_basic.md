---
title: 스프링 기초 정리
author: kymin
date: 2023-12-10 18:47
categories: [Spring]
tags: [spring, web, java]
---

## 스프링(Spring)

스프링 프레임워크(Spring Framework)는 자바 플랫폼을 통해 오픈소스 어플리케이션을 만들기 위한 프레임워크로 줄여서 스프링으로 불린다.

### 경량 컨테이너

스프링은 경량 컨테이너로서 객체를 직접 관리한다. 이 때 스프링 컨테이너가 관리하는 자바 객체를 스프링빈이라고 한다.

개발자는 자바 객체를 스프링 빈으로 등록할 수 있다. 다른 객체에서 스프링빈으로 등록된 객체를 호출하게 되면 스프링 컨테이너는 해당 객체 대한 인스턴스를 넘겨준다.

이 때 객체는 스프링 컨테이너에 싱글톤으로 등록되기 때문에 객체들은 하나의 스프링빈에 대해 같은 인스턴스를 사용하게 된다.

> 싱글톤?
>
> 싱글톤 패턴은 특정 클래스의 인스턴스를 1개만 생성되는 것을 보장하는 디자인 패턴이다. 즉, 생성자를 통해서 여러 번 호출이 되더라도 인스턴스를 새로 생성하지 않고 최초 호출 시에 만들어두었던 인스턴스를 재활용하는 패턴이다.

#### 스프링빈 등록

자바 객체를 스프링빈으로 등록하는 방법으로는 자바 코드를 이용하여 직접 등록하는 방법과 컴포넌트 스캔을 이용하는 방법이 있다.

- 자바 코드로 직접 등록

  `@Configuration`어노테이션을 통해 스프링빈을 등록하는 역할의 클래스를 지정하고, 해당 클래스 내부에 `@Bean`어노테이션과 함께 스프링빈으로 등록할 객체의 생성자를 호출하는 방식이다.

  ```java
  @Configuration	//스프링빈을 등록하는 클래스로 지정
  public class SpringConfig {
      @Bean	//스프링빈 등록
      public TestBean testBean() {
          return new TestBean();
      }
  }
  ```

  정형화된 계층에 해당하는 객체나 대부분의 경우 컴포넌트 스탠 방식을 통해 스프링빈으로 등록하지만 정형화되지 않거나, 상황에 따라 구현 클래스를 변경해야하는 객체의 경우 직접 코드를 통해 스프링빈으로 등록한다.

- 컴포넌트 스캔

  스프링은 `@Component`어노테이션을 자동으로 스캔하여 해당 객체를 스프링빈으로 등록한다.

  또한 `@Controller`, `@Service`, `@Repository`어노테이션은 `@Component`를 포함하기 때문에  `@Component`와 마찬가지로 스프링빈으로 등록하는 데에 사용된다.

  따라서 아래와 같이 클래스에 어노테이션만 달아주면 스프링이 알아서 해당 객체를 스프링 빈으로 등록한다.

  ```java
  @Service
  public class TestService {
    ...
  }
  ```

  컴포넌트 스캔 방식은 정형화된 계층에 해당하는 `Controller`, `Service`, `Repository`등의 객체를 스프링빈으로 등록하는 데에 주로 사용되고, 이외에도 특별한 경우가 아니라면  대부분의 경우에 사용된다.

### 의존성 주입(Dependency Inejction)

의존성 주입은 외부에서 클라이언트에게 서비스를 제공(주입)하는 것을 의미한다. 즉, 객체가 필요로 하는 어떤 것을 외부에서 전달해주는 것을 말한다.

의존성 주입으로 객체는 자신이 사용할 의존 객체를 직접 생성하지 않고, 외부에서 생성된 객체를 주입받는다. 이 때 주입되는 객체는 스프링빈으로 등록된 객체로, 의존 객체를 주입 받은 객체들은 싱글톤으로 등록된 스프링빈을 사용하게 되어 같은 인스턴스를 사용하게 된다.

의존성 주입을 통해 객체가 의존 객체의 생성에 관여하지 않도록 하여 객체 간의 결합도를 낮추고 코드의  재사용성을 높일 수 있다. 또한 코드의 객체 간의 결합도가 낮기 때문에 유지보수와 단위테스트가 용이해진다는 장점도 있다.

#### 의존성 주입 방법

의존성 주입을 하는 방법으로는 필드 주입, 수정자 주입, 생성자 주입이 있다.

또한 기본적으로 의존성 주입에는 `@Autowired`어노테이션이 사용된다.

> Autowired?
>
> 의존성 주입에 활용되는 어노테이션으로 필요한 의존 객체의 타입에 해당하는 스프링빈을 자동으로 찾아서 주입해주는 역할을 한다.

- 필드 주입(Field Injection)

  의존 객체를 객체의 필드로 선언하고 `@Autowired`어노테이션을 통해 의존성을 주입받는 방식이다.

  ```java
  @Controller
  public class TestController {
    @Autowired
    private TestService testService;
    ...
  }
  ```

  필드 주입은 `@Autowired`어노테이션에 의존하여 객체의 의존성을 주입하기 때문에 프레임워크가 없는 순수 자바 환경에서는 사용할 수 없고 이에 따라 단위테스트에 용이하지 않다.

  또한 `final`을 사용할 수 없어 객체의 불변성을 보장할 수 없으므로 잘 사용하지 않는다.

- 수정자 주입(Setter Based Injection)

  setter주입이라고도 불리며 의존 객체를 사용하는 객체의 수정자(setter)를 이용하여 의존성을 주입받는 방식이다.

  ```java
  @Controller
  public class TestController {
    private TestService testService;
    
    @Autowired
    public void setTestService(TestService testService) {
      this.testService = testService;
    }
    ...
  }
  ```

  주로 의존 관계가 변경될 가능성이 있는 객체에서 사용하며, 외부에서 해당 객체의 setter를 호출해야만 의존성 주입이 발생한다.

  의존성이 런타임에 주입되기 때문에 의존성이 누락되어 NullPointerException 등의 예외가 발생할 수도 있다.

- 생성자 주입(Constructor Based Injection)

  의존 객체를 사용하는 객체의 생성자를 통해 의존성을 주입받는 방식이다.

  ```java
  @Controller
  public class TestController {
    private TestService testService;
    
    @Autowired
    public TestController(TestService testService) {
      this.testService = testService;
    }
    ...
  }
  ```

  가장 많이 사용되는 의존성 주입 방식이며

  아래와 같이 Lombok라이브러리의 `@RequiredArgsConstructor`를 이용하여 간결하게 작성할 수 있다.

  ```java
  @Controller
  @RequiredArgsConstructor
  public class TestController {
    private TestService testService;
    ...
  }
  ```

  





### 제어의 역전(Inversion Of Control)

제어의 역전이란 객체의 생성 및 제어권을 사용자가 아닌 스프링에게 맡기는 것이다.

기존에는 사용자가 new연산을 통해 객체를 생성하고 메소드를 호출했다면, 스프링빈을 등록하여 객체의 관리를 스프링 컨테이너가 담당하게 된다.

이 때 다른 객체에서 스프링빈으로 등록된 객체를 호출하게 되면 스프링 컨테이너는 해당 객체 대한 인스턴스를 자동으로 넘겨준다.

따라서 사용자는 직접 new를 이용해 생성한 객체를 사용하지 않고, 스프링에 의하여 관리되는 자바 객체를 사용하게 된다.



-----

##### 참고자료 :

[https://spring.io/guides](https://spring.io/guides)

[inflearn - 스프링 입문](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard)

[https://velog.io/@lundy](https://velog.io/@lundy/Spring-Boot-%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B9%88-%EB%93%B1%EB%A1%9D-%EB%B0%A9%EB%B2%95-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8-%EC%8A%A4%EC%BA%94-%EC%A7%81%EC%A0%91-%EB%93%B1%EB%A1%9D)

[https://f-lab.kr/insight](https://f-lab.kr/insight/understanding-spring-framework-and-di)

[https://velog.io/@jhbae0420](https://velog.io/@jhbae0420/%EC%8B%B1%EA%B8%80%ED%86%A4-%ED%8C%A8%ED%84%B4%EC%9D%98-%EC%82%AC%EC%9A%A9-%EC%9D%B4%EC%9C%A0%EC%99%80-%EB%AC%B8%EC%A0%9C%EC%A0%90)

[https://devlog-wjdrbs96.tistory.com](https://devlog-wjdrbs96.tistory.com/166)

