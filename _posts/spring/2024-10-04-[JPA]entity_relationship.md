---
title: 엔티티의 연관 관계
author: kymin
date: 2024-10-04 18:23
categories: [JPA]
tags: [spring, web, java, jpa]


---

## **연관관계**

엔티티 객체는 대부분 다른 엔티티와 연결되어 동작하게 된다.

이러한 엔티티의 연결을 연관관계라고 하며 엔티티의 연관관계를 데이터베이스의 테이블과 매핑하는 것을 연관관계 매핑이라고 한다.

객체는 코드상에서 참조를 통해 연관관계를 형성하고 데이터베이스에서는 다른 테이블의 키를 외래키로 설정하여 연관관계를 형성하게 된다.

JPA는 객체의 참조와 데이터베이스의 외래키를 매핑하여 두 패러다임의 불일치를 해결하는 방식으로 ORM을 제공한다.



### **연관관계의 주인**

데이터베이스에서의 연관관계는 테이블이 다른 테이블의 기본키를 외래키로 참조하는 방식으로 형성되고, 이 때 다른 테이블의 외래키를 가지고 있는 테이블과 해당 테이블에 매핑된 엔티티 객체를 연관관계의 주인이라고 한다.

객체의 연관관계에는 단반향과 양방향이 존재하지만 데이터베이스의 연관관계는 어느 한쪽만 외래키를 가지면 어느 테이블에서나 다른 테이블을 join할 수 있기 때문에 항상 양방향으로 연결된다. 이러한 이유로 연관관계를 설정할 때에 어느 테이블에서 외래키를 저장할지(어느 테이블 또는 객체를 연관관계의 주인으로 설정할지) 결정해야 한다.

이 때 연관관계로 연결된 두 객체들 중에서 연관관계의 주인이 아닌 객체는 자신과 연관된 객체에 대해 조회만 가능하다. 반대로 연관관계의 주인인 객체는 자신과 연결된 객체의 참조값을 이용하여 연관관계에 있는 객체를 수정하여 변경할 수 있다.

```java
@Entity
@Table(name = "team")
public class Team {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @Column(name = "team_id", nullable = false)
  private Long teamId;
  ...
  // 연관관계의 주인을 정의(Member가 연관관계의 주인이며 Member.team에 의해 참조됨을 의미)
  @OneToMany(mappedBy = "team")
  private List<Member> members = new ArrayList<>();    // 일대다 관계이므로 객체의 리스트를 참조
  ...
}

@Entity
@Table(name = "member")
public class Member {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  @Column(name = "member_id", nullable = false)
  private Long memberId;
  ...
  @ManyToOne(fetch = FetchType.LAZY)
  @JoinColumn(name = "team_id")    // 아래의 객체 참조와 매핑될 외래키를 정의
  private Team team;    // 객체 참조
  ...
}
```

위의 코드는 Team과 Member가 일대다로 매핑되어있는 형태로, 팀에는 여러 명의 멤버가 소속될 수 있지만 멤버는 하나의 팀에만 소속되도록 설정한 연관관계이다.

두 객체는 `@OneToMany`와 `@ManyToOne`을 통해 양방향으로 연결이 되어있지만 연관관계의 주인은 Member이므로 아래와 같이 동작하게 된다.

```java
Member member = new Member("이름");
Team team = new team("팀이름");

// 연관관계의 주인이 참조하는 객체를 전달하여 데이터베이스에 저장시 외래키가 저장됨
member.setTeam(team);

// 아래의 코드는 비영속상태일 때는 동작하지만 데이터베이스에 커밋해도 수정된 정보가 반영되지 않음
// 하지만 데이터베이스에 커밋되기 전에 team을 통해 member를 조회를 하는 경우에 setTeam의 결과가 반영되지 않은 상태이므로 커밋 전에도 변경사항을 반영하기 위해 연관관계 편의 메서드에서 사용됨
team.getMembers().add(member);
```

위의 예시에서 Team이 연관관계의 주인일 경우 `team.getMember().setTeam(team)`처럼 접근이 복잡해지고 Member의 정보를 수정하기 위해 Team에 먼저 접근해야하는 패러다임 불일치 문제가 발생하므로 대부분의 경우에는 다수에 속하는 테이블을 연관관계의 주인으로 설정한다. 일대일 연관관계에서도 마찬가지로 패러다임의 일치 여부를 고려하여 비즈니스 로직에 따라 적절한 테이블을 연관관계의 주인으로 설정한다. 

>연관관계 편의 메서드
>
>연관관계 편의 메서드는 객체의 연관관계 설정하는 과정에서 양방향 연관관계를 올바르게 유지할 수 있도록 하는 메서드를 의미합니다.
>
>```java
>@Entity
>@Table(name = "team")
>public class Team {
>  @Id
>  @GeneratedValue(strategy = GenerationType.IDENTITY)
>  @Column(name = "team_id", nullable = false)
>  private Long teamId;
>  ...
>  // 연관관계의 주인을 정의(Member가 연관관계의 주인이며 Member.team에 의해 참조됨을 의미)
>  @OneToMany(mappedBy = "team")
>  private List<Member> members = new ArrayList<>();    // 일대다 관계이므로 객체의 리스트를 참조
>  
>  // 연관관계 편의 메서드 - member의 team에 대한 연관관계를 설정하는 동시에 team과 연결된 member의 목록에도 member의 정보를 추가
>  public void setTeam(Team team) {
>        this.team = team;
>        if (!team.getMembers().contains(this)) {
>            team.getMembers().add(this);
>        }
>    }
>  ...
>}
>
>@Entity
>@Table(name = "member")
>public class Member {
>  @Id
>  @GeneratedValue(strategy = GenerationType.IDENTITY)
>  @Column(name = "member_id", nullable = false)
>  private Long memberId;
>  ...
>  @ManyToOne(fetch = FetchType.LAZY)
>  @JoinColumn(name = "team_id")    // 아래의 객체 참조와 매핑될 외래키를 정의
>  private Team team;    // 객체 참조
>  
>  // 연관관계 편의 메서드 - team과 연결된 member의 목록에도 member의 정보를 추가하는 동시에 member의 team에 대한 연관관계를 설정
>    public void addMember(Member member) {
>        members.add(member);
>        member.setTeam(this);
>    }
>  ...
>}
>```
>
>연관관계 편의 메서드는 패러다임의 일치 여부와 비즈니스 로직을 고려하여 연관관계에 있는 두 객체 중에 한 쪽에만 설정해도 된다.



### **연관관계의 방향성**

데이터베이스의 연관관계는 어느 한쪽만 외래키를 가지면 어느 테이블에서나 다른 테이블을 join할 수 있기 때문에 항상 양방향으로 연결되지만, 객체의 연관관계는 참조를 통해 설정되기 때문에 단방향과 양방향 연관관계가 각각 존재한다.

연관관계에 있는 두 객체 중 한 쪽에만 참조용 필드가 존재하는 경우가 단방향, 두 객체 모두 서로에 대한 참조용 필드가 존재하는 경우가 양방향이다. 객체의 양방향 연결은 두 객체의 서로에 대한 단방향 연결을 통해 설정된다.

무조건적으로 양방향 연결을 설정하게 되면 불필요한 연관관계가 발생하거나 무한 루프가 발생할 수 있기 때문에, 가급적이면 단방향으로 설정한 뒤에 구현하는 과정에서 역방향으로 객체 탐색이 필요하다고 느낄 때 양방향 연결을 추가하는 방법을 사용한다.

- **단방향 연결**

  ```java
  @Entity
  @Table(name = "team")
  public class Team {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "team_id", nullable = false)
    private Long teamId;
    ...
  }
  
  @Entity
  @Table(name = "member")
  public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id", nullable = false)
    private Long memberId;
    ...
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")    // 아래의 객체 참조와 매핑될 외래키를 정의
    private Team team;    // 객체 참조
    ...
  }
  ```

  연관관계의 주인인 엔티티 객체에 연관관계에 있는 객체의 참조를 선언한다. 객체의 연관관계가 생성되거나 변경되는 경우에는 member에 존재하는 참조를 통해 설정할 수 있으며, 연관관계의 주인이 아닌 객체는 다른 객체를 참조할 수 없다.

  연관관계에 있는 객체에 대해 데이터베이스상에서 삭제를 할 때에는 참조되고있는 객체의 연관관계를 먼저 제거한 후에 삭제를 진행해야한다.

- **양방향 연결**

  ```java
  @Entity
  @Table(name = "team")
  public class Team {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "team_id", nullable = false)
    private Long teamId;
    ...
    // 연관관계의 주인을 정의(Member가 연관관계의 주인이며 Member.team에 의해 참조됨을 의미)
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();    // 일대다 관계이므로 객체의 리스트를 참조
    ...
  }
  
  @Entity
  @Table(name = "member")
  public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "member_id", nullable = false)
    private Long memberId;
    ...
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")    // 아래의 객체 참조와 매핑될 외래키를 정의
    private Team team;    // 객체 참조
    ...
  }
  ```

  mappedBy를 통해 연관관계의 주인이 아닌 객체의 참조에 연관관계의 주인이 되는 객체에서 참조를 위해 선언한 필드의 이름을 설정하여 양방향 연결을 진행한다.

  단방향과 달리 연관관계 주인이 아닌 객체에서도 연관관계에 있는 객체의 값을 불러올 수 있지만, 값을 새로 추가하거나 수정할 수 없다. 연관관계에 의해 참조되고 있는 객체에 수정이 발생하고 트랜잭션이 발생하지 않은 상황에서 연관관계 주인이 아닌 객체가 다른 객체를 참조를 통해 조회하는 경우 수정사항이 반영되지 않은 결과가 조회되기 때문에 연관관계 편의 메서드를 설정해주는 것이 좋다.

  Lombok이나 Jackson과 같은 라이브러리의 경우 객체의 인스턴스를 호출하는 기능이 존재하는데, 이를 통해 연관관계의 두 객체가 서로의 인스턴스를 호출하게 되는 경우에 무한루프가 발생할 수 있다. 이를 방지하기 위해 Lombok의 `toString()`을 오버라이딩하여 구현하거나 엔티티를 DTO로 변환하여 반환하는 방법 등을 사용한다.



### **연관관계의 종류**

엔티티 객체의 연결 형태에 따라 다양한 연관관계가 존재하며 각 연관관계의 종류에 맞는 어노테이션을 통해 연관관계를 설정할 수 있다.

연관관계에서 주체가 되는 테이블을 주 테이블, 주체에 의해 관계가 설정되는 테이블을 대상 테이블이라고 한다.

- **일대일(1:1)**

  두 엔티티가 일대일로 연결되어있는 형태의 연관관계를 의미하며 두 엔티티가 서로를 하나씩만 참조해야하는 경우에만 사용한다.

  ```java
  @Entity
  @Table(name = "user")
  public class User {
    @Id
    @Column(name = "user_id", nullable = false, unique = true)
    private String userId;
    
    @Column(name = "password", nullable = false)
    private String password;
    
    // 연관관계의 주인을 정의(UserInfo가 연관관계의 주인이며 UserInfo.user 의해 참조됨을 의미)
    @OneToOne(mappedBy = "user")
    private UserInfo userInfo;    // 객체 참조
    ...
  }
  
  @Entity
  public class UserInfo {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "user_info_id", nullable = false)
    private Long id;
    ...
    @OneToOne
  	@JoinColumn(name = "user_id")    // 아래의 객체 참조와 매핑될 외래키를 정의
    private User user;    // 객체 참조
    ...
  }
  ```

  일대일 연관관계는 외래키의 값이 중복될 일이 없기 때문에 연관관계의 주인이 되는 테이블의 외래키 컬럼에 UNIQUE제약 조건이 추가된다.

  주 테이블과 대상 테이블 모두 외래키를 가질 수 있기 때문에 두 테이블이 다 연관관계의 주인이 될 수 있다. 하지만 주 테이블에 외래키를 저장하여 연관관계의 주인으로 지정할 경우에 대상 테이블이 존재하지 않는 경우, 외래키 값에 NULL이 사용되어 데이터의 무결성을 해칠 수 있고, 대상 테이블을 연관관계의 주인으로 지정할 경우에는 연관관계에 있는 테이블에 대해 즉시 로딩이 발생하기 때문에 성능 저하가 발생할 수 있다. 따라서 이를 해결하기 위해 일대일 관계에서는 양방향 연결을 주로 사용하며 대상 테이블을 연관관계의 주인으로 설정한 뒤에 `FetchType.LAZY`를 통해 지연 로딩을 설정하는 것이 바람직하다.

  

- **일대다(1:N)**

  `@OneToMany`와 `@ManyToOne`이 있으며 이를 일대다와 다대일로 구분하여 사용한다.

  객체와 데이터베이스의 패러다임을 일치시키기 위해 거의 대부분 다수에 해당하는 테이블에 외래키를 저장하여 연관관계의 주인으로 설정한다.

  ```java
  @Entity
  public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "board_id", nullable = false)
    private Long boardId;
    ...
    // 연관관계의 주인을 정의(Comment가 연관관계의 주인이며 Comment.board 의해 참조됨을 의미)
    @OneToMany(mappedBy = "board")
    private List<Comment> comments = new ArrayList<>();    // 일대다 관계이므로 객체의 리스트를 참조
    ...
  }
  
  @Entity
  public class Comment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "comment_id", nullable = false)
    private Long commentId;
    ...
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "board_id")    // 아래의 객체 참조와 매핑될 외래키를 정의
    private Board board;    // 객체 참조
    ...
  }
  ```

  일대일과 달리 불필요한 연관관계가 발생하거나 순환 참조로 인한 무한루프가 발생하는 것을 방지하기 위해 가급적이면 단방향으로 설정한 뒤에 구현하는 과정에서 역방향으로 객체 탐색이 필요하다고 느낄 때 양방향 연결을 추가한다. 또한 트랜잭션이 커밋되기 전에 데이터베이스에 반영할 결과와 다른 결과가 조회되는 것을 방지하기 위해 연관관계 편의 메서드를 구성하는 것이 바람직하다.

  

- **다대다(N:N)**

  주 테이블과 대상 테이블이 서로를 여러 개씩 참조할 수 있는 경우에 사용하는 연관관계이다.

  다대다 연결을 설정하기 위한 `@ManyToMany`어노테이션이 존재하지만 잘 사용하지 않는다.

  객체는 연관관계에 있는 다른 객체를 컬렉션으로 참조하여 다대다 연결을 구현할 수 있지만, 데이터베이스는 테이블을 이용하여 다대다를 구현할 수 없기 때문에 중간 테이블을 생성하여 일대다와 다대일 관계로 연결하는 과정을 통해 다대다 연결을 구현한다.

  `@ManyToMany`는 연관관계의 주인이 되는 객체에 `@JoinTable`을 통해 중간 테이블을 설정하고 중간 테이블이 참조할 외래키들을 설정한다.

  manytomany가 있지만 ~~문제가 있어 잘 사용하지 않음, 중간 테이블을 통해 구현

  ```java
  // @ManyToMany를 이용하여 다대다 양방향 연결
  @Entity
  public class User {
    @Id
    @Column(name = "user_id", nullable = false, unique = true)
    private String userId;
    
    // 연관관계의 주인을 정의(List<ChatRoom>이 연관관계의 주인이며 ChatRoom.users 의해 참조됨을 의미)
    @ManyToMany(mappedBy = "users")
    private List<ChatRoom> ChatRooms = new ArrayList<>();
    ...
  }
  
  @Entity
  public class ChatRoom {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "chat_room_id", nullable = false)
    private Long chatRoomId;
    
    @ManyToMany
    // JoinTable은 연관관계의 주인이 되는 객체에만 작성해주면 된다.
    @JoinTable(name = "partcipant",    // 다대다 연결을 위해 생성할 중간 테이블의 이름
        joinColumns = @JoinColumn(name = "chat_room_id"),    // 중간 테이블이 참조할 외래키 정의
        // 중간테이블이 참조할 반대편 테이블의 외래키 정의
        inverseJoinColumn = @JoinColumn(name = "user_id"))
    private List<User> users = new ArrayList<>();
    ...
  }
  ```

  

  ```java
  // 직접 중간 테이블을 생성하고 @ManyToOne과 @OneToMany로 다대다 연결
  @Entity
  public class User {
    @Id
    @Column(name = "user_id", nullable = false, unique = true)
    private String userId;
    ...
  }
  
  @Entity
  public class ChatRoom {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "chat_room_id", nullable = false)
    private Long chatRoomId;
    ...
  }
  
  @Entity
  public class Participant {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "participant_id", nullable = false)
    private Long participantId;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")    // 중간 테이블이 참조할 user의 외래키 정의
    private User user;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "chat_room_id")    // 중간 테이블이 참조할 chatroom의 외래키 정의
    private ChatRoom chatRoom;
    ...
  }
  ```

  `@ManyToMany`를 이용하면 객체는 컬렉션을 통해 다대다로 연결되고 데이터에비스에는 중간 테이블이 생성된다. 이 과정에서 객체와 테이블 간의 패러다임의 불일치가 발생하는 문제가 있다. 이에 따라 로직을 실행하기 위한 코드의 동작과 데이터베이스의 쿼리문의 괴리가 커지는 문제가 발생할 수 있다. 또한 중간 테이블이 단순히 다대다 관계에 있는 두 객체를 연결하는 역할뿐만 아니라 별도의 데이터를 가질 수 있기 때문에 `@ManyToMany`는 잘 사용되지 않는다.

  주로 `@ManyToMany`대신에 직접 중간 테이블을 엔티티로 선언하고 `@ManyToOne`과 `@OneToMany`를 이용하여 양방향 연결을 구현한다.

  두 외래키를 중간테이블의 pk로 사용하는 경우도 있지만 컬럼의 아이디가 다른 테이블에 종속적이게 되므로 유연한 수정이 어려워져서 거의 사용하지 않는다.



-----

##### 참고자료 :

[https://velog.io/@leeeeeyeon](https://velog.io/@leeeeeyeon/JPA-%EC%97%B0%EA%B4%80-%EA%B4%80%EA%B3%84-%EC%A0%95%EB%A6%AC)

[https://ocwokocw.tistory.com](https://ocwokocw.tistory.com/128)

[https://colabear754.tistory.com](https://colabear754.tistory.com/142)

[https://curiousjinan.tistory.com](https://curiousjinan.tistory.com/entry/Spring-JPA-%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-%EB%A7%A4%ED%95%91)

