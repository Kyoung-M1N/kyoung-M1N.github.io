---
title: 데이터베이스 연동과 JPA
author: kymin
date: 2023-12-16 19:51
categories: [Spring]
tags: [spring, web, java, jpa, database, sql]

---

## 데이터베이스 연동

### 연동 방법

아래와 같이 스프링 프로젝트에 포함된 `build.gradle`의 `dependencies`에 JPA와 MySQL의 라이브러리에 대한 의존성을 추가

```
dependencies {
	...
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'com.mysql:mysql-connector-j'
	...
}
```

> 라이브러리의 버전에 따라 라이브러리의 이름이나 그룹ID가 달라질 수 있으므로 아래의 링크에서 검색 후 `implementation 'Group ID:Artifact ID'`의 형식으로 입력하면 된다.
>
> [Spring Dependency Version](https://docs.spring.io/spring-boot/docs/current/reference/html/dependency-versions.html)

스프링 프로젝트에 MySQL을 연동하기 위해 `src/main/resources/application.properties`에 아래의 내용을 추가

```
# MySQL 드라이버 설정
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# DB Source URL
# ip주소, port번호, db이름만 알맞게 넣고 뒤의 쿼리의 형태로 오는 옵션들은 필요에 따라 추가, 생략
spring.datasource.url=jdbc:mysql://[ip주소]:[port번호]/[db이름]?useSSL=false&useUnicode=true&serverTimezone=Asia/Seoul

# DB username
spring.datasource.username=[db서버에 접근할 계정명]

# DB password
spring.datasource.password=[게정에 설정된 비밀번호]
```

> MySQL포트의 기본값은 3306이다.

이후 MySQL서버를 작동시키면 Spring에서 데이터베이스에 접근할 수 있다.

## JPA(Java Persistence API)

### JPA란?

JPA는 자바에서 ORM(Object-Relational Mapping : 객체 관계 매핑) 기술 표준으로 사용되는 인터페이스의 모음을 말하는 것으로 실제적으로 구현된 것이 아니라 자바로 구현된 클래스와 DB의 테이블을 매핑을 해주기 위해 사용되는 프레임워크이다.

JPA는 애플리케이션과 JDBC사이에서 동작하며 객체와 테이블의 매핑을 통해 패러다임의 불일치 문제를 해결해준다.

> ORM(Object-Relational Mapping)?
>
> 자바의 일반적인 Class와 RDB(Relational DataBase)의 테이블을 매핑(연결)한다는 뜻이며, 기술적으로는 어플리케이션의 객체를 RDB 테이블에 자동으로 영속화 해주는 것이라고 보면된다.

### 클래스와 테이블 매핑

`@Entity`어노테이션을 통해 현재 클래스를 테이블과 매핑한다고 JPA에 알려주며 이렇게 매핑된 클래스를 엔티티 클래스 또는 엔티티 객체라고 한다.

`@table(name = '테이블이름')`을 통해 클래스와 매핑될 테이블을 설정할 수 있으며, 데이블과 엔티티의 이름이 같을 경우에는 어노테이션을 생략해도 자동으로 매핑된다.

엔티티의 각 필드들을 어노테이션을 통해 데이터베이스의 column과 매핑할 수 있지만, 엔티티의 필드명과 데이터베이스의 column명이 일치할 경우 자동으로 매핑이 이루어진다.

```java
import jakarta.persistence.*;
import lombok.Data;

@Data	//모든 필드에 대해 getter메서드와 toString메서드 등을 자동으로 생성, non final 필드에 대해서는 자동으로 setter메서드도 생성
@Entity	//해당 클래스를 엔티티로 선언
public class Member {
    @Id	//엔티티 클래스의 필드를 테이블의 기본 키(Primary Key)에 매핑
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    private String name;
    private String email;
    private String password;
}
```

### JPA의 사용

- 엔티티매니저 생성

  JPA는 `EntityManager`를 통해 Entity를 관리하며 데이터베이스의 테이블과 매핑된 Entity를 통해 데이터를 관리한다.

  따라서 아래와 같이 엔티티매니저를 생성하고 데이터베이스의 테이블과 매핑된 엔티티를 통해 데이터를 관리한다.

  ```java
  private final EntityManager entityManager;
  
  public JpaMemberRepository(EntityManager entityManager) {
      this.entityManager = entityManager;
  }
  
  entityManager.persist(member);	//저장
  entityManager.remove(member);		//삭제
  ```

- 트랜잭션 관리

  JPA를 사용하면 항상 트랜잭션 안에서 데이터의 변경이 이루어져야 한다.

  따라서 데이터 변경을 수행하는 로직을 가진 클래스나 메서드에 `@Transactional`어노테이션을 통해 트랜잭션을 통해 데이터를 관리하도록 한다.

  ```java
  @Transactional
  public class MemberService {
      private final MemberRepository memberRepository;
  
      public MemberService(MemberRepository memberRepository) {
          this.memberRepository = memberRepository;
      }
    
    	public List<Member> findMembers() {
          return memberRepository.findAll();
      }
  
      public Optional<Member> findOne(long id) {
          return memberRepository.findById(id);
      }
  }
  ```

  > 트랜잭션(Transaction)?
  >
  > 트랜잭션은 데이터베이스의 상태를 변화시키기 해서 수행하는 작업의 단위를 뜻한다.
  >
  > 트랜잭션이 진행되는 동안에 데이터베이스가 변경되더라도 업데이트된 데이터베이스로 트랜잭션이 진행되는것이 아니라, 처음에 트랜잭션을 진행 하기 위해 참조한 데이터베이스로 진행된다.
  >
  > 트랜잭션이 정상적으로 동작을 완료한 경우 `commit`을 통해 데이터베이스에 결과를 반영하게 된다.
  >
  > 트랜잭션이 정상적으로 동작을 완료하지 못했을 경우에는 작업의 내용이 저장되지 않으며, `rollback`을 통해 트랜잭션이 실행되기 이전의 상태와 같도록 유지한다.

- JPQL

  `EntityManager`의 `createuery()`를 통해 쿼리문을 작성하여 기본적으로 JPA의 `EntityManager`가 제공하는 기능이 아닌 작업을 수행할 수 있다.

  ```java
  public List<Member> findAll() {
      return entityManager.createQuery("select m from Member m", Member.class)
              .getResultList();
      }
  ```

  JPQL은 엔티티를 대상으로 쿼리문을 작성해야하기 때문에 위의 예시에서 `Member`는 테이블의 이름이 아니라 엔티티의 이름이다.(JPQL은 데이터베이스 테이블을 전혀 알지 못한다.)

  JPA는 JPQL을 분석하여 적절한 SQL을 만들고 데이터베이스에 대한 작업이 이루어지게 된다.

  이 때 쿼리문은 Member, m.age와 같이 엔티티와 속성은 대소문자를 구분하여 작성하며, Member (as) m과 같이 별칭 설정을 필수로 해야 한다.(별칭을 정의하는 예약어인 as는 생략 가능)



## Spring Data JPA

### Spring Data JPA란?

Spring Data JPA는 스프링 프레임워크에서 JPA를 편리하게 사용할 수 있도록 지원하는 프로젝트로 데이터 접근 계층을 개발할 때 구현 클래스 없이 인터페이스만 작성해도 개발을 완료할 수 있다.(엔티티 매니저도 자동으로 관리)

Spring Data JPA는 `save()`, `delete()`, `findAll()`등의 일반적인 CRUD를 공통적으로 제공하며 메서드의 이름을 분석해서 이미 구현된 JPQL을 실행한다.

```java
import java.util.Optional;
import com.spring_training.domain.member.Member;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MemberRepository extends JpaRepository<Member, Long> {
  	// 제네릭에 엔티티 클래스와 식별자 타입을 지정
    Member findByUsername(String username);
}
```

위의 예시에서 `findByUsername(String username)`의 경우에는 스프링 데이터 JPA가 메소드 이름을 분석해서 JPQL을 자동으로 생성하여 실행하게 된다.(현재 예시의 경우 `select m from Member m where username = :username`와 같은 JPQL을 자동으로 생성하여 실행하게 된다.)

주로 `username`같이 보편적으로 사용되는 column명이 아닌 경우에 위의 예시와 같이 사용한다.

---

##### 참고자료 :

[ultrakain.gitbooks.io](https://ultrakain.gitbooks.io/jpa/content/chapter1/chapter1.3.html)

[inflearn - 스프링 입문](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%EC%9E%85%EB%AC%B8-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8/dashboard)