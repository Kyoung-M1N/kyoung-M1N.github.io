---
title: 객체지향 프로그래밍(Object-Oriented Programming)
author: kymin
date: 2023-10-24 14:53
categories: [OOP]
tags: [oop, java]
---



## 객체지향 프로그래밍?

객체지향 프로그래밍은 컴퓨터 프로그램을 어떤 데이터를 입력받아 순서대로 처리하고 결과를 도출하는 명령어들의 목록으로 바라보는 기존의 절차지향적인 시각에서 벗어나 프로그램을 여러 독립적인 객체들의 결합과 상호작용으로 바라보는 프로그래밍의 패러다임을 말한다.

>이 때의 객체는 객체지향 프로그래밍의 가장 기본적인 단위이며, 프로그램을 구성하는 가장 작은 구성 요소이다.

>소스코드를 작성하는 사람의 정의에 따라 프로그램을 구성하는 데이터 요소뿐만 아니라 프로그램에 필요한 기능 자체나 논리만으로도 객체를 정의할 수 있다.

## 객체지향의 장점

### 유연한 변경

프로그램을 만드는 과정에서 기능의 수정, 삭제를 해야할 경우가 생기면 프로그램의 전체 흐름과 순서까지 수정해야 하는 절차지향과 달리 객체지향을 해당 기능을 하는 객체만을 찾아서 수정하거나 삭제하면 된다. 

또한 객체는 각자 독립적인 역할을 가지고 있기 때문에 수정이 필요한 상황에서도 코드의 변경이 최소화되며 유지보수에 유리하다.

### 간결한 코드

객체를 이용하면 코드의 재사용을 통해 반복적인 코드를 최소화하여 코드를 최대한 간결하게 작성할 수 있다.

### 직관적인 코드

객체지향 프로그래밍은 실제 우리가 보고 셩험하는 세계의 사물이나 현상을 최대한 프로그래밍에 반영하는 방향으로 발전을 해왔기 때문에 소스코드를 볼 때 인간에게 더 직관적으로 다가오며 이해하기도 쉽다.

## 객체지향 프로그래밍의 특징

- ### 추상화(Abstraction)

  여러가지 요소나 객체의 공통적인 특성과 기능을 추출하여 객체를 정의하는 것을 의미한다.

  자바에서 추상화를 구현할 수 있는 문법적인 요소로는 추상클래스(abstract class)와 인터페이스(interface)가 있다.

  추상클래스와 인터페이스를 이용하여 공통적인 기능을 정의하고 상속을 통해 하위클래스를 선언하여 코드의 반복을 줄이면서 여러가지 객체를 정의할 수 있다.

  > 인터페이스는 하나의 하위클래스가 여러 개의 인터페이스를 동시에 상속 받을 수 있지만, 추상클래스는 하위클래스가 하나의 추상클래스만 상속 받을 수 있다.

  ```java
  public interface Parson {
    String name;
    int age;
    
    void walk();
  }
  ```

  

- ### 상속(Inheritance)

  기존에 정의된 클래스를 재활용하여 새로운 클래스를 생성하는 것을 의미한다.

  추상화와 상속을 통해 코드의 재사용성을 높이고 반복되는 코드를 줄일 수 있다.

  > 추상클래스는 `extends`키워드를 통해, 인터페이스는 `implements`키워드를 통해 하위클래스를 생성한다.

  ```java
  public class Parson {		//추상클래스
    String name;
    int age;
    
    void walk() {
      System.out.println("걸어가는 중");
    }
  }
  ```

  ```java
  public class Student extends Parson {		//추상클래스를 상속반은 하위클래스
    String school;
  }
  ```

  ```java
  public class Main {
    public static void main(String[] args) {
      Student student = new Student();
      
      student.name = "박경민";
      student.age = 24;
      student.school = "광운대학교";
      
      System.out.println(student.school + "의 학생 " + student.age + "살 " + 						student.name);
    }
  }
  
  //결과
  광운대학교의 학생 24살 박경민
  ```

  

- ### 다형성(Polymorphism)

  객체의 기능이나 속성이 상황에 따라 여러 가지의 형태를 가질 수 있는 성질을 의미한다.

  상속 관계에 있는 상위클래스와 하위클래스가 존재할 때, 하위 클래스는 상위클래스를 참조할 수 있으며, 형변환을 통해 업캐스팅도 가능하다.(상위클래스가 하위클래스를 참조하거나 다운캐스팅은 불가능)

  또한 상위클래스가 메서드로써 제공하는 기능을 메서드 오버라이딩으로 재정의 하거나 메서드 오버로딩을 통해 다양한 기능을 제공할 수 있다.

  >- 메서드 오버라이딩
  >
  >   상위 클래스에서 선언된 메서드를 하위클래스가 재정의하는 것
  >
  >   오버라이딩 후에도 `super`예약어를 통해 상위클래스의 메서드를 불러올 수 있다.
  >
  >- 메서드 오버로딩
  >
  >   이름은 같지만 매개변수의 속성과 갯수에 차이를 두고 메서드를 선언하는 것
  >

  이와 같이 다형성을 통해 객체간의 의존성을 줄일 수 있으며 이를 기반으로 스프링에서는 제어의 역전(IoC)과 의존성 주입(DI)이라는 방법을 사용하기도 한다.

  > 예를 들어 A라는 클래스 내부에서 B라는 클래스의 인스턴스를 생성하고 해당 인스턴스의 메서드를 사용한다면 A는 B에 의존한다고 말할 수 있다.

  ```java
  public class Student extends Parson {		//추상클래스를 상속반은 하위클래스
    String school;
    
    @override
    void walk() {
      System.out.println("등교하는 중");
    }
  }
  ```

  

  

- ### 캡슐화(Encapsulation)

  클래스 내부의 데이터들을 외부로 부터 보호하는 것을 의미한다.

  즉, 클래스 내부의 데이터와 데이터를 처리할 수 있는 기능들을 한 곳에 모아 관리하는 것이다.

  >**캡슐화를 하는 이유 - 객체의 독립성과 책임영역을 지키기 위함**
  >
  >- 데이터 보호
  >
  >  외부로부터 클래스에 정의된 속성과 기능들이 변경되거나 삭제되는 것을 방지한다.
  >
  >- 데이터 은닉
  >
  >  객체 내부에서 발생하는 동작을 감추고 필요한 부분만 객체의 외부로 노출시킨다.

  캡슐화는 보통 접근제어자와 getter/setter 메서드를 이용하여 구현한다.

  - 접근제어자

    자바에는 `public`, `default`, `protected`, `private`까지 총 4가지의 접근제어자가 있다.

    | 접근제어자  | 접근 가능 범위                                     |
    | ----------- | -------------------------------------------------- |
    | `public`    | 접근 제한 없음(패키지 외부에서도 접근 가능)        |
    | `default`   | 동일 패키지와 다른 패키지의 클래스에서도 접근 가능 |
    | `protected` | 같은 패키지 내에서만 접근 가능                     |
    | `private`   | 같은 클래스 내에서만 접근 가능                     |
  
  - getter/setter 메서드
  
    `private`으로 선언된 필드에 접근하기 위한 함수이며 이를 통해 외부에서 클래스의 필드에 직접 접근하는 것을 방지한다.
  
    보통 아래와 같은 형태로 구현한다.
  
    ```java
    public class A {
      private int number;
      
      public int getNumber() {
        return number;
      }
      
      public void setNumber(int number) {
        this.number = number;
      }
    }
    ```
  
  캡슐화도 마찬가지로 다른 클래스가 신경쓸 필요가 없는 데이터나 함수에 대해 외부의 접근을 막아 데이터를 보호하고 객체 간의 의존성을 낮춰준다는 장점이 있디.
  
  하지만 결국 getter/setter로 인해 클래스 내부의 데이터에 접근과 수정을 할 수 있기 때문에 결과적으로는 데이터 보호가 이루어 지지 않는다는 한계가 있다.
  
  이를 해결하기 위해  클래스 내에 getter/setter메서드를 선언하기 보다는 DTO와 VO라는 개념을 적용하여 사용하는 추세이다.

-----

##### 참고자료 : 

[codestates.com](https://www.codestates.com/blog/content/%EA%B0%9D%EC%B2%B4-%EC%A7%80%ED%96%A5-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D-%ED%8A%B9%EC%A7%95)

