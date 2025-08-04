---
title: 동시성 문제와 교착 상태
author: kymin
date: 2025-02-09 13:58
categories: [Spring]
tags: [spring, web, java]


---

## **동시성 문제**

### **동시성(Concurrency)**

동시성은 두 사건이 같은 시간에 발생하는 것을 의미한다.

웹 서버는 여러 명의 사용자가 보낸 요청을 동시에 처리하며, 이는 작성한 코드에 의해 구현된 기능이 동시에 작동할 수 있다는 것을 의미한다.

동시성 문제는 경쟁 조건에 놓여있는 기능이 동시에 여러 번 동작하는 경우에 발생한다.

> **경쟁 조건(Race Condition)**
>
> 경쟁 조건은 여러 개의 프로세스나 스레드가 동일한 데이터에 접근하여 값을 조작할 때 타이밍이나 접근 순서에 따라 예상하는 결과와 다른 결과가 나타날 수 있는 조건을 의미한다.

동시성 문제는 예상한 결과와 실제 결과가 다르게 나오지만 오류가 발생하지 않기 때문에 알아내기 어렵다.

또한 실제 로컬에서 POSTMAN과 같은 도구를 이용하여 테스트 할 때에는 여러 명의 사용자가 동시에 요청을 보내는 경우를 만들어내기 힘들기 때문에 알아차리기 어렵다.

### **데드 락에 의한 교착상태**

예를 들어 아래와 같이 구현된 수강신청 서비스가 있다고 가정하고 동시성 문제가 발생하는 지 알아보기 위해 여러 개의 수강신청 등록 요청을 동시에 보내보았다.

```java
// controller
public class LectureController {
	@PostMapping(path = "/{lectureId}/{studentId}")
	public ResponseEntity<String> register(
			@PathVariable(name = "lectureId") final long lectureId,
			@PathVariable(name = "studentId") final long studentId
	) {
		lectureService.registStudentToLecture(lectureId, studentId);
		return ResponseEntity.status(HttpStatus.CREATED).body("수강신청이 완료되었습니다");
	}
}

// service
public class LectureService {
	@Transactional
	public void registStudentToLecture(final long lectureId, final long studentId) {
		Lecture lecture = lectureRepository.findById(lectureId)
				.orElseThrow(() -> new NullPointerException("강의가 존재하지 않습니다"));

		Student student = studentRepository.findById(studentId)
				.orElseThrow(() -> new NullPointerException("학생 정보가 존재하지 않습니다"));

		if (lectureStudentRepository.existsByStudentAndLecture(student, lecture)) {
			throw new IllegalArgumentException("이미 수강중인 강의입니다");
		}
		
		lectureStudentRepository.save(LectureStudent.of(student, lecture));
		lecture.registStudent();
	}
}

// entity
public class Lecture {
	...
	@Column(name = "remaining_seat", nullable = false)
	private int remainingSeat;
	...

	public void registStudent() {
		if (remainingSeat <= 0) {
			throw new IllegalArgumentException("잔여 좌석이 없습니다");
		}
		this.remainingSeat -= 1;
	}
}
```

수강인원이 20명인 강의를 생성하고 jmeter를 이용하여 20명의 학생이 동시에 수강 신청을 하는 상황에 대한 요청을 발생시킨 뒤의 결과는 아래와 같다.

```sql
+----------------+------------+------------+-----------------------+
| remaining_seat | total_seat | lecture_id | title                 |
+----------------+------------+------------+-----------------------+
|              3 |         20 |          1 | 컴퓨터구조              |
+----------------+------------+------------+-----------------------+
```

수강 인원이 20명인 강의에 20명의 학생이 수강 신청 요청을 보냈지만 3개의 요청이 누락된 것을 확인할 수 있다.

엔티티에서 기록하는 값이 아닌 실제 데이터베이스의 데이터를 확인해보아도 데이터가 누락되어있는 것을 확인할 수 있었다.

위와 같이 요청이 누락된 원인은 아래와 같다.

```
SQL Error: 1213, SQLState: 40001
Deadlock found when trying to get lock; try restarting transaction
```

로그에 따르면 lock을 가져오는 과정에서 Deadlock이 발생했다고 한다.

```java
public class LectureStudent {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long lectureStudentId;

	@ManyToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "student_id")
	private Student student;

	@ManyToOne(fetch = FetchType.LAZY)
	@JoinColumn(name = "lecture_id")
	private Lecture lecture;
  ...
}
```

데드락이 발생하는 이유는 수강 신청 요청으로 인해 생성되는 `lecture_student`테이블에 외래키 제약조건이 존재하기 때문이다.

위의 엔티티는 학생 정보와 강의 정보를 다대다로 연결하기 위한 `LectureStudent`엔티티이다. 다대다 연관관계를 위해 학생 정보와 강의 정보에 각각 `@ManyToOne`의 연관관계가 설정되어 있으며 이에 따라 `lecture_student`테이블에 두 개의 외래키 제약조건이 발생한다.

MySQL은 외래키 제약조건이 존재하는 테이블에 데이터가 생성, 수정, 삭제가 발생하면 참조 무결성의 위반 여부를 파악하기 위해 외래키가 있는 원본 테이블의 레코드에 공유 락(Shared Rock)을 설정하게 된다. 이 때 여러 트랜잭션에서 공유락을 획득한 뒤 쓰기 작업을 위해 배타적 락(Exclusive Rock)으로 변경을 시도하는데, 그 과정에서 서로 다른 트랜잭션의 공유 락의 해제를 기다리다가 충돌이 발생하여 데드락이 발생한다.

> 외래키 제약조건이 존재하는 테이블의 데이터 조작이 발생하면 트랜잭션에서 해당 레코드에 대한 공유 락을 얻게 되는데, 공유 락은 읽기 전용 락이기 때문에 여러 트랜잭션이 동시에 획득 가능하다.
>
> 이후 참조 무결성의 위반 여부를 검증한 뒤에 트랜잭션은 데이터를 변경하기 위해 공유 락을 배타적 락으로 변경하려고 시도하지만 배타적 락은 쓰기 락이기 때문에 다른 트랜잭션에서 해당 레코드에 공유 락이나 배타적 락 등을 걸어놓으면 획득할 수 없다.
>
> 하지만 트랜잭션 락은 트랜잭션이 커밋되거나 롤백될 때에만 해제 가능하고 트랜잭션이 실행되는 도중에는 락의 종류를 변경할 수만 있기 때문에, 여러 트랜잭션이 공유 락에서 배타적 락으로 변경을 시도하는 과정에서 충돌이 발생하고 결국 데드락이 발생한다.

따라서 동시성 문제를 해결하기 위해서는 참조 무결성 검증 과정에 의한 데드락 문제를 방지하기 위해 외래키 제약 조건을 없애야하므로 엔티티를 아래와 같이 변경한다.

```java
public class LectureStudent {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long lectureStudentId;

	@Column(name = "student_id", nullable = false)
	private Long studentId;

	@Column(name = "lecture_id", nullable = false)
	private Long lectureId;
  ...
}
```

다대다 연관관계를 위한 엔티티를 위와 같이 변경하고 다시 한 번 jmeter를 이용하여 20명의 학생이 동시에 수강 신청을 하는 상황에 대한 요청을 발생시킨 결과 데이터베이스에 20명의 수강신청 결과가 잘 생성되어 있는 것을 확인할 수 있었다.

하지만 여전히 `Lecture`엔티티가 카운트 하는 값인 `remainingSeat`에는 아래와 같이 오류가 발생하는 것을 확인할 수 있다.

```sql
+----------------+------------+------------+-----------------------+
| remaining_seat | total_seat | lecture_id | title                 |
+----------------+------------+------------+-----------------------+
|              3 |         20 |          1 | 컴퓨터구조              |
+----------------+------------+------------+-----------------------+
```

외래키 제약조건을 없애 데드락에 의해 교착상태가 발생하는 것은 해결하였지만 여러 트랜잭션이 동일한 데이터를 수정하는 과정에서의 오류는 해결되지 않았다.

### **동시성 문제 해결**

자세한 문제를 알아보기 위해 수강 인원이 20명인 강의에 30명의 학생이 동시에 수강 신청 요청을 보내는 상황을 발생시켰다.

정상적인 동작이 수행된다면 강의 정보의 `remaining_seat`가 0이 되고 `lecture_student`테이블에 30명의 학생에 대한 컬럼이 생성되면 이후에는 예외가 발생해야 한다.

하지만 강의 정보와 학생 정보를 다대다로 연결하며 수강 인원을 나타내는 `lecture_student`테이블에 30명의 학생에 대한 컬럼이 생성되었으며 강의 정보에는 `remaining_seat`의 값이 2로 나타났다.

발생한 요청에 대한 쿼리문을 확인한 결과 30번의 요청에 대해 모든 쿼리가 정상적으로 발생한 것을 확인할 수 있었고, 이에 따라 `remaining_seat`에 대한 update 쿼리도 30번 발생했음을 확인할 수 있었다.

퀴리문이 정상적으로 발생했음에도 데이터베이스의 값이 쿼리문 만큼 변경되지 않은 이유는 아래와 같은 흐름으로 요청이 처리되었기 때문이다.

- 수강 신청 요청이 동시에 발생함에 따라, 여러 개의 트랜잭션이 `lecture`테이블의 `remaining_seat`에 접근
- 서로 다른 트랜잭션에서 같은 값을 조회 후 +1을 진행
- 트랜잭션이 커밋되며 쿼리문이 발생
- 이 때 같은 값을 읽은 뒤 1을 더한 값을 업데이트

결과적으로 update 쿼리는 30번 발생했지만, 같은 값을 읽은 여러 개의 트랜잭션이 1을 더한 값을 커밋했기 때문에 같은 값들이 업데이트 되면서 20명이라는 수강 인원 제한이 정상적으로 동작하지 않은 것이다.

### **순차적으로 요청 처리**

`synchronized` 키워드를 이용하여 동시 발생으로 인해 여러 스레드에서 처리되는 요청을 순차적으로 처리해보았다.

```java
public class LectureService {
  @Transactional
	public synchronized void registStudentToLecture(final long lectureId, final long studentId) {
		if(!studentRepository.existsById(studentId)) {
			throw new NullPointerException("학생 정보가 존재하지 않습니다");
		}

		Lecture lecture = lectureRepository.findById(lectureId)
				.orElseThrow(() -> new NullPointerException("강의가 존재하지 않습니다"));

		if (lectureStudentRepository.existsByStudentIdAndLectureId(studentId, lectureId)) {
			throw new IllegalArgumentException("이미 수강중인 강의입니다");
		}

		lectureStudentRepository.save(LectureStudent.of(lectureId, studentId));
		lecture.registStudent();
	}
}
```

위와 같이 메서드를 변경하고 실행한 결과 `synchronized`가 없을 때, 30명의 학생이 수강 신청을 보냈음에도  `remaining_seat`이 0이 되지 않고 모두 등록되던 것과 달리 `remaining_seat`가 0이 되고 이후 요청에서 의도한 대로 예외가 발생하는 것을 확인할 수 있었다.

```sql
+----------------+------------+------------+-----------------------+
| remaining_seat | total_seat | lecture_id | title                 |
+----------------+------------+------------+-----------------------+
|              0 |         20 |          1 | 컴퓨터구조              |
+----------------+------------+------------+-----------------------+
```

하지만 여전히 수강 인원인 20명 보다 많은 인원이 등록되는 것을 확인하였고, 여전히 요청이 병렬로 처리되고 있음을 알 수 있었다.

> `synchronized` 키워드를 사용하면 싱글톤으로 동작하는 스프링 빈의 메서드가 여러 스레드에서 동기화되면서 순차적으로 동작하지만, 현재 스프링 빈에 등록되어 싱글톤으로 동작하는 `LectureService`의 메서드가 병렬로 동작하는 것을 확인할 수 있었다.

`synchronized` 키워드가 있음에도 스프링 빈에 등록되어 싱글톤으로 동작하는 `LectureService`의 메서드가 순차적으로 처리되지 않는 이유는 해당 메서드가 트랜잭션 내에서 동작하기 때문이다.

`@Transactional`은 AOP로 동작하게 된다. 따라서 해당 키워드가 존재하는 메서드가 호출되면 AOP프록시가 먼저 메서드의 호출을 가로채고 트랜잭션을 시작한 뒤에 메서드를 실행하게 된다. 이 때 `synchronized`는 AOP 프록시 객체가 담당하는 트랜잭션의 제어가 아닌 해당 키워드가 존재하는 메서드에만 스레드 락을 설정하게 된다. 따라서 여러 스레드에서 동작하는 트랜잭션이 `synchronized` 키워드가 있는 메서드를 순차적으로 처리하더라도, 다른 트랜잭션에서 해당 메서드의 동작이 완료되고 트랜잭션이  커밋되기 전에 메서드가 실행되어 데이터베이스의 값을 읽어오기 때문에 스레드 동기화로 인한 순차 처리의 효과가 완전하게 나타나지 않는 것이다.

즉 A, B트랜잭션이 동작을 하게되는 경우, A 트랜잭션 시작, B 트랜잭션 시작, A가 `synchronized`메서드 호출, B는 A에서 `synchronized`메서드의 동작이 끝날 때 까지 대기 후 호출, A 트랜잭션 커밋, B 트랜잭션 커밋 순으로 동작하여 스레드 동기화를 적용해도 원하는대로 동작하지 않는다.













### **synchronized를 메서드에 적용**

`synchronized`는 자바에서 지원하는 기능으로 멀티스레드 환경에서 여러 개의 스레드간의 동기화를 진행하기 위해 사용한다.

여러 개의 스레드가 하나의 데이터에 접근하여 값을 수정하면 해당 스레드의 동작이 끝나기 전에 다른 스레드에서는 바뀐 데이터의 값을 알 수 없기 때문에 데이터에 접근하여 값을 수정하는 기능을 하는 메서드나 필드에 `synchronized`키워드를 추가하여 쓰레드 간의 데이터 동기화를 진행한다.

`synchronized`키워드를 사용하면

모니터 객체가 공유 자원에 다른 스레드가 접근할 수 없도록 락을 걸어버린다.

자바가 내부적으로 메서드나 필드에 block처리를 하여 다른 스레드에서 접근할 수 없도록 하고 작업이 끝나면 unblock처리를 하여 다른 스레드가 접근 가능하도록 한다.

동기화를 통해 특정 데이터에 block이 걸리면 해당 데이터에 접근하려는 다른 스레드들은 대기 해야하므로 동시 접근이 많아질 수록 성능이 저하될 수 있다.

동시성 문제에 대응하기 위해 서비스의 메서드에 `synchronized`키워드를 추가한다.

```java
// service
public class LectureService {
  @Transactional
  public synchronized void regist(Long lectureId) {
    // 조회 과정에서 영속성 컨텍스트의 스냅샷을 읽고 스냅샷에 걸린 락이 바로 풀려서 커밋 전에 다른 트랜잭션이 스냅샷을 읽는 것이 가능하다.
    Lecture lecture = lectureReposutory.findById(lectureId).orElseThrow(() -> new NullPointerException("강의가 존재하지 않습니다."));
    lecture.incrementRegisteredStudent();
  }
}
```

위와 같이 수정한 뒤에도 동시성 문제는 해결되지 않는 것을 확인할 수 있다.

그 이유는 synchronized로 인해 공통 데이터에 락이 걸리기 전에 먼저 transaction이 시작되고 transaction이 시작되는 과정에서 이미 공통 데이터에 접근하여 값을 읽고 수정했기 때문이다

따라서 synchronized를 이용하여 동시성 문제를 해결하기 위해서는 transaction이 사작되기 전에 synchronized 키워드가 적용된 메서드의 호출이 발생해야한다.

- controller호출에 synchronized 적용하기

  ```java
  // controller
  public class LectureController {
    @PostMapping("/lecture/regist/{lectureId}")
    // synchronized가 락을 하는 대상이 lectureService.regist()이기 때문에 동시성 문제가 해결됨
    public synchronized ResponseEntity<?> registLecture(@PathVariable Long lectureId) {
      try {
        lectureService.regist(lectureId);
        return ResponseEntity.status(HttpStatus.OK).body("수강 신청이 등록되었습니다.");
      } catch (NullPointerException e) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(e.getMessage());
      }
    }
  }
  ```

  

- 서비스 레이어에서 @Transactional어노테이션을 지우고 직접 조회, 저장 기능 구현

  ```java
  // service
  public class LectureService {
    public synchronized void regist(Long lectureId) {
      Lecture lecture = lectureReposutory.findById(lectureId).orElseThrow(() -> new NullPointerException("강의가 존재하지 않습니다."));
      lecture.incrementRegisteredStudent();
      // @Transactional을 없애고 직접 save를 하면 save에서 스냅샷과 영속성 컨텍스트의 값을 비교 후 바로 flush하는 과정으로 인해 동시성 문제가 해결된다.
      lectureReposutory.save(lecture);	// 직접 save 추가
      // 또는 lectureReposutory.flush()를 해도 된다.
    }
  }
  ```

출력되는 쿼리문을 보면 select -> select -> update -> update순으로 동작하던 기존과 달리 select -> update -> select -> update로 동작 순서가 직렬 처리된 것을 알 수 있다.

둘 다 컨트롤러, 서비스 레이어의 메서드의 동작 과정에서 동시에 발생하는 모든 요청이 하나씩 순차적으로 처리되므로 다른 스레드가 대기해야하고 이로 인해 성능 저하가 발생한다.

또한 동일한 프로세스 내의 스레드들 사이에서만 동작하기 때문에 서버가 여러 대로 분산되어잇는 경우에는 동시성 문제가 해결되지 않는다.







A요청의 읽기 B요청의 읽기 A요청의 수정 B요청의 수정 A요청의 커밋 B요청의 커밋 순으로 발생함

synchronized를 적용하여 문제를 해결하려면 트랜잭션이 열리기 이전에 synchronized가 적용되어야함



synchronized는 동일한 프로세스 내의 스레드에게만 적용되기 때문에 여러 대의 서버가 있는 상황에서는 동일한 데이터베이스에 접근하더라도 다른 프로세스 단위에서 접근하기 때문에 단일 쓰레드만 접근하도록 제한하는 방법으로는 해결할 수 없음

### **트랜잭션의 전파 속성을 이용한 동시성 문제 해결**

트랜잭션의 전파는 트랜잭션이 어떻게 동작할 것인가를 결정하는 방식을 의미한다.

앞선 설명에서 다른 스레드이므로 다른 트랜잭션이 열린다.

다른 스레드이므로 이미 존재하는 트랜잭션의 값을 읽어올 수 없다..?

다른 스레드의 트랜잭션이 변경한 값을 읽어오는 게 가능한지는 알아봐여함

- 이미 존재하는 트랜잭션에 참여

  이미 존재하는 트랜잭션에 참여하여 같은 트랜잭션 안에서 작업을 수행한다.

  하나의 작업에만 rollback이 발생해도 같은 트랜잭션 내의 모든 작업들이 rollback된다.

  

- 독립적인 트랜잭션을 생성

  이미 트랜잭션이 존래하거나 트랜잭션이 없는 상황에서 새로운 트랜잭션을 추가하여 작업을 진행한다.

  두 작업은 독립적으로 동작한다.

- 트랜잭션 없이 동작

  단순 읽기와 같이 트랜잭션이 필요하지 않은 특정 작업에 대해 트랜잭션을 걸지 않는다

### **트랜잭션의 격리 수준을 이용한 동시성 문제 해결**

- DEFAULT

  현재 사용 중인 DBMS의 데이터 접근 기술이나 DB 드라이버의 기본 설정을 적용하는 격리 수준으로 대부분의 DB는 READ_COMMITTED를 기본 격리 수준으로 가진다.

- READ_UNCOMMITTED

  가장 낮은 단계의 격리 수준으로 트랜잭션이 커밋하기 전에 미리 변경된 값을 읽을 수 있다.

  다른 스레드라면 읽을 수 없다..?

- READ_COMMITTED

  대부분의 DB에서 기본적으로 사용하는 격리 수준으로 다른 트랜잭션이 커밋하지 않은 정보를 읽을 수 없다.

- REPEATABLE_READ

  한 트랜잭션이 읽은 ROW를 수정할 수 없도록 막지만 새로운 ROW의 생성은 막지 않음

- SERIALIZABLE

  동시에 여러 개의 트랜잭션이 테이블에 접근할 수 없도록 설정, 트랜잭션이 순차적으로 실행되도록 한다.

  한 트랜잭션이 테이블에 접근하면 읽기락을 먼저 걸고 쓰기 락을 걸어버린다.

  

  데드락 에러가 발생한다.

  아마도 여러 트랜잭션이 동시에 락을 걸수 있는 상황이 발생하고 이로 인해 서로 락이 풀리기 기다리는 상황이 발생하며 접ㄱㄴ이 불가능해지는 문제가 생기기 때문?

  트랜잭션이 동시에 

### **락을 이용한 동시성 문제 해결**

- 낙관적 락(Optimistic Lock)

  대부분의 트랜잭션이 충돌하지 않는다고 가정하는 방법으로 

- 비관적 락(Pessimistic Lock)

  대부분의 트랜잭션이 충돌할 것이라고 가정하는 방법으로 

-----

##### 참고자료 :

[https://zzang9ha.tistory.com](https://zzang9ha.tistory.com/443)

[https://tecoble.techcourse.co.kr](https://tecoble.techcourse.co.kr/post/2023-08-16-concurrency-managing/)