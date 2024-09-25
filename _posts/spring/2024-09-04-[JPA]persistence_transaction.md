---
title: JPA-엔티티 매니저와 트랜잭션
author: kymin
date: 2024-09-04 19:31
categories: [JPA]
tags: [spring, web, java, jpa]


---

## EntityManager?

EntityManager는 말그대로 Entity를 관리하는 역할을 하며, 영속성 컨텍스트에 접근하여 Entity에 대한 데이터베이스 상에서의 작업을 제공하는 역할을 한다.

EntityManager는 데이터베이스에 대한 작업요청이 발생하면 EntityManagerFactory를 통해 생성되고 JDBC가 제공하는 커넥션 풀(Connection Pool)을 사용하여 데이터베이스에 대한 작업을 수행한다.

> 커넥션 풀(Connection Pool)?
>
> 커넥션 풀은 JDBC가 자바 애플리케이션과 데이터베이스를 연결할 때 연결을 관리하기 위한 기법이다.
>
> JDBC는 커넥션 풀에 자바 애플리케이션과 데이터베이스 사이의 커넥션을 미리 생성해 놓고 애플리케이션이 데이터베이스와 통신하려고 할 때 커넥션을 가져와서 사용한 뒤에 사용이 끝나면 커넥션을 다시 커넥션 풀에 반환한다.
>
> 커넥션들은 커넥션 풀에 대기하며 애플리케이션에 의해 재사용되고 매번 애플리케이션과 데이터베이스의 연결을 생성, 종료하는 것보다 성능적인 측면에서 훨씬 효율적이다.

EntityManagerFactory는 EntityManager를 생성해주는 객체로 EntityManagerFactory가 생성되면 커넥션 풀과 연결된다.

EntityManagerFactory는 여러 쓰레드가 동시에 접근해도 안전하기 때문에 하나만 생성하여 애플리케이션 내에서 공유하며 사용한다. 하지만 EntityManager는 여러 쓰레드에서 동시에 접근할 셩우 데이터의 불일치가 발생할 수 있기 때문에 애플리케이션 내에서 공유되지 않고 DB에 대한 작업이 끝나면 바로 삭제(close)된다.

## 영속성 컨텍스트와 트랜잭션

아래의 코드를 통해 EntityManagerFactory에 의해 EntityManager가 생성될 때, 영속성 컨텍스트도 함께 생성된다.

```java
// EntityManagerFactory 생성
EntityManagerFactory emf = Persistence.createEntityManagerFactory("persistenceUnitName");

// EntityManagerFactory로 EntityManager 생성
EntityManager em = emf.createEntityManager();
```

영속성 컨텍스트(Persistence Context)는 엔티티를 영구 저장하는 환경이라는 의미로 눈에 보이지 않는 논리적인 개념이다.

엔티티 객체를 생성하고 EntityManager를 통해 데이터베이스에 대한 작업을 수행하게 되면 바로 데이터베이스에 반영이 되는 것이 아니라 영속성 컨텍스트에 먼저 반영이 된다.

영속성 컨텍스트는 1차 캐시(First-Level Cache)와 쓰기 지연 쿼리 저장소(Write-Behind Store)를 포함하고있다. 따라서 EntityManager가 데이터베이스에 대한 작업을 위해 함수를 실행하면 엔티티 객체는 1차 캐시에, 함수 호출에 의해 생성된 쿼리문은 쓰기 지연 쿼리 저장소에 먼저 저장된다.

이 때, EntityManager는 트랜잭션 내에서 실행되는데, 트랜잭션(Transaction)은 데이터베이스의 상태를 변경하는 작업의 단위라는 의미이다.

트랜잭션은 데이터베이스에 대한 작업이 성공하면 커밋(commit)을 통해 데이터베이스의 변경을 진행하고, 작업이 실패하면 롤백(roll back)을 통해 1차 캐시를 초기화하고 데이터베이스를 작업이 발생하기 이전 상태로 되돌린다. 또한 트랜잭션은 작업의 격리를 통해 다른 트랜잭션에서 현재 작업이 진행중인 데이터에 접근하지 못하게 하는 역할도 수행한다.

따라서 데이터의 일관성과 무결성을 보장하고 데이터베이스에 대한 작업이 안정적으로 이루어질 수 있도록 JPA의 모든 데이터 변경은 트랜잭션 안에서 이루어져야 하며, 트랜잭션에서 커밋이 발생할 경우에 쓰기 지연 쿼리 저장소의 쿼리문들에 대한 flush가 발생하며 1차 캐시에 존재하는 엔티티 객체들의 내용이 데이터베이스에 반영된다.

```java
EntityManagerFactory emf = Persistence.createEntityManagerFactory("persistenceUnitName");

EntityManager em = emf.createEntityManager();

EntityTransaction tx = em.getTransaction();	// 엔티티 매니저에 대한 트랜잭션 생성

tx.begin();	// 트랜잭션 시작 -> 이제 엔티티 매니저의 모든 작업은 트랜잭션 내에서 수행됨
    try {
      
        // 데이터베이스에 대한 작업 수행

        tx.commit();	// 엔티티 매니저가 필요한 작업이 모두 끝나면 트랜잭션에서 커밋을 시도
    } catch (Exception e) {
        tx.rollback();	// 예외 발생 시 트랜잭션에서 발생한 작업을 처음 상태로 다시 되돌림
    } finally {
        em.close();	// 엔티티 매니저 삭제(트랜잭션 종료도 함께 일어남)
        emf.close();
    }
```

> flush와 commit
>
> flush와 commit은 둘 다 영속성 컨텍스트를 데이터베이스에 동기화하는 데에 사용되지만 동작에서 차이점이 존재한다.
>
> - flush
>
>   영속성 컨텍스트의 변경 사항을 데이터베이스에 동기화하는 작업으로 쓰기 지연 쿼리 저장소의 SQL쿼리가 데이터베이스로 전송되어 데이터베이스에 엔티티 객테의 데이터를 반영한다.
>
>   아직 **트랜잭션이 완료되지 않은 상태**이기 때문에 트랜잭션에서 rollback이 발생하면 flush로 인한 변경사항도 취소된다.
>
> - commit
>
>   **현재 트랜잭션을 완료**하고, 트랜잭션 내에서 이루어진 모든 변경 사항을 영구적으로 데이터베이스에 반영하는 작업으로 commit 메서드를 호출하면 자동으로 flush가 먼저 발생하고 해당 트랜잭션은 종료된다.
>
>   commit이 성공적으로 실행되어 데이터베이스가 변경되면 트랜잭션이 종료되기 때문에 rollback이 불가능하다.

### 엔티티의 생명주기

- 비영속 상태(new/transient)

  - 순수한 객체 상태로 영속성 컨텍스트와 관련이 없는 상태를 의미한다.

    ```java
    Object object = new Object();
    ```

- 영속 상태(managed)

  - EntityManager를 통해 엔티티 객체가 1차 캐시에 저장된 상태로 영속성 컨텍스트가 엔티티를 관리하고 있는 상태를 의미한다.

    ```java
    entityManager.persist(object);	// 객체에 영속성 부여
    ```

- 준영속 상태(detached)

  - 엔티티 객체가 영속 상태였다가 영속성 컨텍스트와 분리된 상태를 의미한다.

    ```java
    entityManager.detach(object);	// 객체를 영속성 컨텍스트와 분리
    entityManager.close();	// 엔티티 매니저를 삭제(닫기)하여 객체와 영속성 컨텍스트를 분리
    entityManager.clear();	// 영속성 컨텍스트를 초기화하여 객체와 영속성 컨텍스트를 분리
    ```

  - 엔티티가 1차 캐시에 저장되어 있지 않다는 점에서 비영속 상태와 유사하지만, 준영속 상태인 엔티티 객체는 `entityManager.merge()`를 통해 다시 영속성 상태로 전환될 수 있다는 것이 차이점이다.

  - 또한 준영속 상태인 엔티티 객체는 영속 상태였던 적이 있기 때문에 id가 존재하고, 비영속 상태인 엔티티 객체는 영속 상태였던 적이 없기 때문에 id가 존재하지 않는다.

  - 비영속 상태인 엔티티 객체도 `entityManager.merge()`를 통해 영속상태로 변경할 수 있지만 영속성 컨텍스트에 새로운 영속 상태 엔티티가 추가되기 때문에 `entityManager.persist(object)`와 동일한 역할을 하게 된다.

- 삭제(removal)

  - 엔티티 객체가 영속성 컨텍스트에 의해 관리되지 않으며, 데이터베이스에서 삭제될 예정인 상태를 의미한다.

    ```java
    entityManager.remove(object);	// 영속성 컨텍스트에서 해당 엔티티를 제거하고 DELETE쿼리를 생성
    ```

  - 엔티티 객체는 영속성 컨텍스트의 1차 캐시에는 없는 상태가 되지만 flush가 발생하기 전에는 아직 데이터베이스 상에 해당 엔티티에 대한 데이터가 존재하며 쓰기 지연 쿼리 저장소에는 해당 엔티티에 대한 DELETE 쿼리가 존재한다.

### 요청별 엔티티매니저와 트랜잭션의 동작

- **생성**

  데이터베이스에 새로운 데이터를 생성하기 위해 새로운 엔티티 객체 생성하고 해당 객체를 영속 상태로 전환한다. 이 때, 영속성 컨텍스트의 1차 캐시에 엔티티 객체의 데이터가 저장되고 쓰기 지연 쿼리 저장소에는 INSERT쿼리가 생성된다.

  이후에 EntityManager의 작업이 모두 끝나게 되면 트랜잭션이 commit을 시도한다. 이 때 flush가 발생하며 commit이 성공적으로 끝나면 트랜잭션이 종료되고 commit이 실패하면 rollback이 발생한다.

  flush가 발생하면 쓰기 지연 쿼리 저장소의 SQL쿼리가 데이터베이스에 전달되어 실행된다.

  ```java
  EntityManagerFactory emf = Persistence.createEntityManagerFactory("persistenceUnitName");
  
  EntityManager em = emf.createEntityManager();
  
  EntityTransaction tx = em.getTransaction();	// 엔티티 매니저에 대한 트랜잭션 생성
  
  tx.begin();	// 트랜잭션 시작 -> 이제 엔티티 매니저의 모든 작업은 트랜잭션 내에서 수행됨
      try {
          Object object = new Object();	// 새로운 엔티티 객체 생성(비영속 상태)
          
          em.persist(object);	// 엔티티 객체에 영속성 부여(영속 상태, 1차 캐시에 저장, INSERT쿼리 생성)
        
          tx.commit();	// 엔티티 매니저가 필요한 작업이 모두 끝나면 트랜잭션에서 커밋을 시도
      } catch (Exception e) {
          tx.rollback();	// 예외 발생 시 트랜잭션에서 발생한 작업을 처음 상태로 다시 되돌림
      } finally {
          em.close();	// 엔티티 매니저 삭제(트랜잭션 종료도 함께 일어남)
          emf.close();
      }
  ```

  

- **조회**

  데이터베이스에서 데이터를 불러오기 위해 EntityManager는 전달반은 엔티티와 primary key등의 정보를 이용하여 SELECT문을 실행하여 조회를 수행한다. 이 때 조회하고자 JPA는 1차 캐시를 먼저 조회해보고 데이터베이스를 조회하게 된다. 따라서 1차 캐시에 조회하려는 엔티티 객체가 존재하면 SELECT쿼리는 전송되지 않는다.

  또한 이 때 반환되는 엔티티는 모두 같은 인스턴스이므로 동일성이 보장되어 `==`로 객체를 비교하면 true가 반환된다.

  이후에 EntityManager의 작업이 모두 끝나게 되면 트랜잭션이 commit을 시도한다. 이 때 flush가 발생하며 commit이 성공적으로 끝나면 트랜잭션이 종료되고 commit이 실패하면 rollback이 발생한다.

  조회를 위한 SELECT쿼리는 JPA가 1차 캐시에 존재하지 않는 데이터를 조회할 때마다 실행되므로 flush가 발생할 때 까지 쓰기 지연 쿼리 저장소에 등록되어 실행을 대기하지 않는다.

  ```java
  EntityManagerFactory emf = Persistence.createEntityManagerFactory("persistenceUnitName");
  
  EntityManager em = emf.createEntityManager();
  
  EntityTransaction tx = em.getTransaction();	// 엔티티 매니저에 대한 트랜잭션 생성
  
  tx.begin();	// 트랜잭션 시작 -> 이제 엔티티 매니저의 모든 작업은 트랜잭션 내에서 수행됨
      try {
          Object object = em.find(Object.class, 1L)	// primary key와 mapping된 엔티티 클래스로 조회(영속 상태)
        
          tx.commit();	// 엔티티 매니저가 필요한 작업이 모두 끝나면 트랜잭션에서 커밋을 시도
      } catch (Exception e) {
          tx.rollback();	// 예외 발생 시 트랜잭션에서 발생한 작업을 처음 상태로 다시 되돌림
      } finally {
          em.close();	// 엔티티 매니저 삭제(트랜잭션 종료도 함께 일어남)
          emf.close();
      }
  ```

- **수정**

  JPA를 이용하여 엔티티를 수정하게 되면 JPA는 변경 감지(dirty checking)를 통해 UPDATE쿼리를 생성한다. 엔티티를 조회하게 되면 1차 캐시에 최초 엔티티 조회시 상태를 스냅샷으로 저장하고 데이터 변경 작업이 이루어진 뒤에 flush가 발생하면, 1차 캐시의 스냅샷과 엔티티를 비교하여 데이터가 다르면 update쿼리를 생성하여 쓰기 지연 SQL 저장, 데이터베이스에 전달한다.

  변경 감지는 영속 상태의 객체에서만 일어나며 비영속, 준영속 상태의 객체는 값을 변경해도 변경 감지가 발생하지 않아 데이터베이스에 변경이 반영되지 않는다.

  ```java
  EntityManagerFactory emf = Persistence.createEntityManagerFactory("persistenceUnitName");
  
  EntityManager em = emf.createEntityManager();
  
  EntityTransaction tx = em.getTransaction();	// 엔티티 매니저에 대한 트랜잭션 생성
  
  tx.begin();	// 트랜잭션 시작 -> 이제 엔티티 매니저의 모든 작업은 트랜잭션 내에서 수행됨
      try {
          Object object = em.find(Object.class, 1L)	// primary key와 mapping된 엔티티 클래스로 조회(영속 상태)
          object.setName("changed");	// 영속 상태의 객체의 값을 변경 -> JPA가 자동으로 변경을 감지하여 flush가 발생하면 UPDATE쿼리를 생성
        
          tx.commit();	// 엔티티 매니저가 필요한 작업이 모두 끝나면 트랜잭션에서 커밋을 시도
      } catch (Exception e) {
          tx.rollback();	// 예외 발생 시 트랜잭션에서 발생한 작업을 처음 상태로 다시 되돌림
      } finally {
          em.close();	// 엔티티 매니저 삭제(트랜잭션 종료도 함께 일어남)
          emf.close();
      }
  ```

- **삭제**

  엔티티를 데이터베이스에서 삭제하기 위해서는 삭제하려는 엔티티를 조회해야 한다. 즉, 삭제하려는 엔티티가 데이터베이스에 존재하며, 영속 상태여야 한다. 하지만 데이터베이스에 대상이 존재하지 않더라도 결과적으로 해당 데이터가 데이터베이스에 존재하지 않게 되는 것은 같기 때문에  DELETE쿼리는 정상적으로 실행되며 에러는 발생하지 않는다.

  엔티티를 삭제하는 코드가 실행되면 해당 엔티티는 1차 캐시에서 즉시 삭제되며, 해당 엔티티에 대한 DELETE쿼리가 쓰기 지연 쿼리 저장소에 등록된다.

  이후에 EntityManager의 작업이 모두 끝나게 되면 트랜잭션이 commit을 시도한다. 이 때 flush가 발생하며 commit이 성공적으로 끝나면 트랜잭션이 종료되고 commit이 실패하면 rollback이 발생한다.

  ```java
  EntityManagerFactory emf = Persistence.createEntityManagerFactory("persistenceUnitName");
  
  EntityManager em = emf.createEntityManager();
  
  EntityTransaction tx = em.getTransaction();	// 엔티티 매니저에 대한 트랜잭션 생성
  
  tx.begin();	// 트랜잭션 시작 -> 이제 엔티티 매니저의 모든 작업은 트랜잭션 내에서 수행됨
      try {
          Object object = em.find(Object.class, 1L)	// primary key와 mapping된 엔티티 클래스로 조회(영속 상태)
          em.remove(object);	// 영속 상태의 엔티티를 삭제, 엔티티가 1차 캐시에서 바로 삭제되며 쓰기 지연 쿼리 저장소에 object에 대한 DELETE쿼리 등록, object는 가비지 컬렉터에 의해 제거
        
          tx.commit();	// 엔티티 매니저가 필요한 작업이 모두 끝나면 트랜잭션에서 커밋을 시도
      } catch (Exception e) {
          tx.rollback();	// 예외 발생 시 트랜잭션에서 발생한 작업을 처음 상태로 다시 되돌림
      } finally {
          em.close();	// 엔티티 매니저 삭제(트랜잭션 종료도 함께 일어남)
          emf.close();
      }
  ```

  이 때 삭제된 엔티티에 대한 인스턴스는 가비지 컬렉터에 의해 제거된다.

-----

##### 참고자료 :

[https://dev-troh.tistory.com](https://dev-troh.tistory.com/151)

[https://lng1982.tistory.com](https://lng1982.tistory.com/275)

[https://siyoon210.tistory.com](https://siyoon210.tistory.com/138)

[https://velog.io/@shasha](https://velog.io/@shasha/Database-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EC%A0%95%EB%A6%AC)

[https://brightstarit.tistory.com](https://brightstarit.tistory.com/24)

