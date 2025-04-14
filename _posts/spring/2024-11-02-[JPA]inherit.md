---
title: 상속 관계의 연결
author: kymin
date: 2024-11-02 21:17
categories: [JPA]
tags: [spring, web, java, jpa]


---

## **상속 관계인 클래스의 매핑**

데이터베이스에는 상속 관계가 존재하지 않는 대신에 상속 관계와 유사한 형태의 슈퍼타입-서브타입 관계라는 모델링 기법이 존재한다.

> **슈퍼타입-서브타입**
>
> 여러 가지의 엔티티들에서 공통적인 부분과 그외의 나머지 부분을 나누어 관리하는 데이터베이스 모델링 기법으로 공통된 속성을 부모 클래스로, 그외의 나머지 부분을 자식 클래스로 관리하는 상속관계와 형태가 유사하다.
>
> 서브타입 엔티티는 슈퍼타입 엔티티와 join되므로 슈퍼타입 엔티티의 pk를 fk로 가지고 동시에 해당 fk를 자신의 pk로 가진다.
>
> 따라서 자기 자신만의 유니크한 pk가 존재하지 않으며, 슈퍼 타입을 통해서만 접근 가능하다.

주로 이를 이용하여 상속관계인 클래스를 데이터베이스의 테이블과 매핑하지만 이외에도 테이블을 각각 나누어 관리하는 각자 테이블 전략과 합쳐서 관리하는 단일 테이블 전략이 존재한다.



### **Join 이용하기**

부모 클래스와 자식 클래스 모두 자신만의 테이블을 가제게 되며 슈퍼-서브 타입 관계로 테이블이 생성되게 된다.

부모 클래스와 자식 클래스가 1대1로 연결되며 부모 클래스에 는 dtype이라는 컬럼을 만들기도 한다.

엔티티의 구조와 거의 흡사하며 가상속관계의 매핑에서 가장 많이 사용된다.

join쿼리가 함께 전달 동작하기 때문에 약간의 성능저하가 발생할 수 있지만 거의 영향이 없다고 한다.

### 단일 테이블로 구현하기

하나의 테이블이 생성되며 부모 클래스와 부모 클래스를 상속받은 자식 클래스의 데이터가 모두 하나의 테이블에 저장된다.

다른 자식클래스에 해당하는 데이터에는 null값이 들어가게 된다.

테이블의 크기가 크지 않고 자식 클래스의 수가 많지 않은 경우에 거의 사용된다.

단일 테이블이므로 join을 이용하여 매핑하는 방식보다 약간의 성능 이점이 존재한다.



### 각자 테이블로 구현하기

자식 클래스마다 테이블이 생성되며 부모 클래스의 데이터가 함께 생성되어 각각의 테이블에 저장된다

상속 관계를 전혀 나타내지 않는 형태이므로 거의 사용되지 않는다.

### 

상속 관계에서 자식클래스인 엔티티를 조회한 쿼리 결과 with 페이지네이션

상속관계 매핑으로 인해 join쿼리가 추가됨

자식클래스에 해당하는 테이블에 데이터가 추가될 때 부모 클래스의 테이블을 join해야하므로 pk이자 fk로 하기 위해 select한 번, alter한 번 쿼리가 발생한다.

dtype에는 기본적으로 자식클래스의 이름이 그대로 들어가게 된다.



조회 시에는 자식클래스의 테이블에 대해 조회를 하면 join된 부모 클래스의 테이블도 같이 연결되어 조회된다.



삭제할 때에 부모 클래스와 자식 클래스 둘 다의 repository를 통해 삭제를 할 수 있지만 부모 클래스의 repository로 삭제할 경우 삭제를 위한 조회 과정에서 left join쿼리가 자식클래스의 수만큼 발생한다. 반면에 자식 클래스의 repository로 삭제할 경우 삭제를 위한 조회 과정에서 부모 클래스에 대한 join쿼리가 하나만 발생한다.

즉 부모 클래스의 repository로 조회 하면 자식 클래스의 테이블 모두를 조회해야하므로 자식 클래스의 페이블에 쿼리를 보내는 것이 성능상 약간의 이점이 발생할 수 있을 것 같다.

```mysql
// 자식클래스로의 조회쿼리
Hibernate: 
    select
        cb1_0.board_id,
        cb1_1.content,
        cb1_1.created_at,
        cb1_1.title,
        cb1_1.writer,
        cb1_0.meeting_place 
    from
        carpool_board cb1_0 
    join
        board cb1_1 
            on cb1_0.board_id=cb1_1.board_id 
    order by
        cb1_1.created_at desc 
    limit
        ?
```

```mysql
// 부모 클래스의 repository로 delete
Hibernate: 
    select
        b1_0.board_id,
        b1_0.dtype,
        b1_0.content,
        b1_0.created_at,
        b1_0.title,
        b1_0.writer,
        b1_1.meeting_place,
        b1_2.meeting_time 
    from
        board b1_0 
    left join
        carpool_board b1_1 
            on b1_0.board_id=b1_1.board_id 
    left join
        team_board b1_2 
            on b1_0.board_id=b1_2.board_id 
    where
        b1_0.board_id=?
Hibernate: 
    delete 
    from
        carpool_board 
    where
        board_id=?
Hibernate: 
    delete 
    from
        board 
    where
        board_id=?
```

```mysql
// 자식클래스의 repository로 delete
Hibernate: 
    select
        cb1_0.board_id,
        cb1_1.content,
        cb1_1.created_at,
        cb1_1.title,
        cb1_1.writer,
        cb1_0.meeting_place 
    from
        carpool_board cb1_0 
    join
        board cb1_1 
            on cb1_0.board_id=cb1_1.board_id 
    where
        cb1_0.board_id=?
Hibernate: 
    delete 
    from
        carpool_board 
    where
        board_id=?
Hibernate: -
    delete 
    from
        board 
    where
        board_id=?
```









-----

##### 참고자료 :

[https://velog.io/@leeeeeyeon](https://velog.io/@leeeeeyeon/JPA-%EC%97%B0%EA%B4%80-%EA%B4%80%EA%B3%84-%EC%A0%95%EB%A6%AC)

[https://velog.io/@imcool2551](https://velog.io/@imcool2551/JPA-%EC%83%81%EC%86%8D%EA%B4%80%EA%B3%84-%EB%A7%A4%ED%95%91)