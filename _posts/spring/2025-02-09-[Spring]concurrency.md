---
title: 동시성 문제와 교착 상태
author: kymin
date: 2025-04-09 13:58
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
    
		lecture.registStudent();
		
		lectureStudentRepository.save(LectureStudent.of(student, lecture));
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

하지만 여전히 `Lecture`엔티티가 카운트 하는 값인 `remainingSeat`에는 아래와 같이 오류가 발생하는 것을 확인할 수 있다. 즉, 현재 수강이 가능한 인원보다 더 많은 인원이 수강 등록될 수 있는 문제가 발생하게 된다.

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
  ...
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
    
		lecture.registStudent();

		lectureStudentRepository.save(LectureStudent.of(lectureId, studentId));
	}
  ...
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

`@Transactional`은 AOP로 동작하게 된다. 따라서 해당 키워드가 존재하는 메서드가 호출되면 AOP프록시가 먼저 메서드의 호출을 가로채고 트랜잭션을 시작한 뒤에 메서드를 실행하게 된다. 이 때 `synchronized`는 AOP 프록시 객체가 담당하는 트랜잭션의 제어가 아닌 해당 키워드가 존재하는 메서드에만 모니터 락을 설정하게 된다. 따라서 여러 스레드에서 동작하는 트랜잭션이 `synchronized` 키워드가 있는 메서드를 순차적으로 처리하더라도, 다른 트랜잭션에서 해당 메서드의 동작이 완료되고 트랜잭션이  커밋되기 전에 메서드가 실행되어 데이터베이스의 값을 읽어오기 때문에 스레드 동기화로 인한 순차 처리의 효과가 완전하게 나타나지 않는 것이다.

> `synchronized` 키워드를 사용하면 공유 자원에 대한 접근을 제어하는 객체인 모니터(Monitor)에 의해 스레드가 모니터 락을 획득 및 반환하며 스레드 동기화가 이루어진다.

즉 A, B트랜잭션이 동작을 하게되는 경우, A 트랜잭션 시작, B 트랜잭션 시작, A가 `synchronized`메서드 호출, B는 A에서 `synchronized`메서드의 동작이 끝날 때 까지 대기 후 호출, A 트랜잭션 커밋, B 트랜잭션 커밋 순으로 동작하여 스레드 동기화를 적용해도 원하는대로 동작하지 않는다.

따라서 `synchronized`를 통해 요청을 순차적으로 처리하기 위해서는 아래와 같이 트랜잭션이 제어되는 부분까지 `synchronized`의 범위에 포함시켜야 한다.

- service의 메서드를 호출하는 controller에서 synchronized 적용하기

  ```java
  public class LectureController {
  	private final LectureService lectureService;
    ...
    @PostMapping(path = "/{lectureId}/{studentId}")
  	public synchronized ResponseEntity<String> register(
  			@PathVariable(name = "lectureId") final long lectureId,
  			@PathVariable(name = "studentId") final long studentId
  	) {
  		lectureService.registStudentToLecture(lectureId, studentId);
  		return ResponseEntity.status(HttpStatus.CREATED).body("수강신청이 완료되었습니다");
  	}
  }
  ```

- 내부적으로 `@Transactional`을 사용하는 메서드로 직접 트랜잭션 제어

  ```java
  public class LectureService {
    ...
  	public synchronized void registStudentToLecture(final long lectureId, final long studentId) {
  		if(!studentRepository.existsById(studentId)) {
  			throw new NullPointerException("학생 정보가 존재하지 않습니다");
  		}
  
  		Lecture lecture = lectureRepository.findById(lectureId)
  				.orElseThrow(() -> new NullPointerException("강의가 존재하지 않습니다"));
  
  		if (lectureStudentRepository.existsByStudentIdAndLectureId(studentId, lectureId)) {
  			throw new IllegalArgumentException("이미 수강중인 강의입니다");
  		}
      
  		lecture.registStudent();
      // 내부적으로 @Transactional을 사용하는 save를 사용하여 더티체크하고 저장
      lectureRepository.save(lecture);
      
  		lectureStudentRepository.save(LectureStudent.of(lectureId, studentId));
  	}
    ...
  }
  ```

위의 두 방식으로 변경한 뒤에 30명의 학생이 수강 인원이 20명인 강의에 대해 동시에 수강 신청을 하는 경우를 테스트한 결과, 의도한대로 20명의 학생만 수강 신청이 이루어지고 이후에는 예외가 발생하는 것을 확인할 수 있다.

하지만 위와 같이 동시에 발생한 요청을 순차적으로 처리하는 방법에는 몇가지 단점이 존재한다.

`synchronized`는 스레드 동기화를 위해 메서드에 스레드 락을 설정하므로 다른 스레드에서 해당 메서드를 호출하여 동작 중인 경우에 다른 스레드들은 대기해야 한다. 따라서 동시 요청이 많을 수록 스레드의 대기 시간이 늘어나기 때문에 성능 저하가 발생할 수 있다.

또한 `synchronized`는 동일한 프로세스에서의 스레드에서만 동기화가 적용되기 때문에 서버가 여러 대로 분산된 환경에서는 데이터의 정합성이 깨지며 동시성 문제를 해결할 수 없게 된다.

### **트랜잭션 격리 수준 이용**

다음으로 트랜잭션 격리 수준을 SERIALIZABLE로 설정하여 분산 서버 환경에서도 동시성 문제를 해결할 수 있는지 확인해보기 위해 아래와 같이 변경해보았다.

```java
public class LectureService {
    ...
    @Transactional(isolation = Isolation.SERIALIZABLE)
	public void registStudentToLecture(final long lectureId, final long studentId) {
		if(!studentRepository.existsById(studentId)) {
			throw new NullPointerException("학생 정보가 존재하지 않습니다");
		}

		Lecture lecture = lectureRepository.findById(lectureId)
				.orElseThrow(() -> new NullPointerException("강의가 존재하지 않습니다"));

		if (lectureStudentRepository.existsByStudentIdAndLectureId(studentId, lectureId)) {
			throw new IllegalArgumentException("이미 수강중인 강의입니다");
		}
    
		lecture.registStudent();

		lectureStudentRepository.save(LectureStudent.of(lectureId, studentId));
	}
    ...
}
```

위와 같이 변경한 뒤에 30명의 학생이 수강 인원이 20명인 강의에 대해 동시에 수강 신청을 하는 경우를 테스트한 결과, 비교적 동시성 처리가 잘 되는듯 하지만 중간에 데드락이 발생한다.

데드락이 발생하는 이유는 SERIALIZABLE이 실제 트랜잭션의 실행을 직렬로 처리하는 것이 아니라 내부적으로 락이나 트랜잭션 스케줄링 등을 이용하여 트랜잭션들이 순차적으로 실행된 것처럼 보이도록 결과를 직렬화하기 때문이다.

따라서 데이터베이스에 접근하는 과정에서 레코드에 공유락을 설정하고 외래키 제약조건이 설정된 경우와 같이 공유락을 획득한 여러 트랜잭션이 배타적 락으로의 전환을 대기하며 데드락이 발생하여 교착 상태가 발생한다.

또한 SERIALIZABLE은 데이터베이스에서 발생하는 읽기와 쓰기의 충돌을 방지하기 위해 다른 격리 수준보다 락은 빈번하게 사용할 뿐만 아니라 범위 락까지 사용하기 때문에 다른 격리 수준에 비해 성능이 떨어진다는 문제가 있다.

### **데이터베이스 락 적용**

데이터베이스의 락 전략에는 비관적 락, 낙관적 락 두 가지가 있다.

>- 낙관적 락(Optimistic Lock)
>
>  대부분의 트랜잭션이 충돌하지 않는다고 가정하는 방법이다.
>
>  트랜잭션에 의한 데이터 조작이 시작되기 전에 버전이나 타임스탬프를 기록하고 커밋이 발생하는 시점에서의 버전이나 타임스탬프를 비교하여 충돌을 감지하고, 충돌이 발생하면 트랜잭션의 동작을 재실행한다.
>
>- 비관적 락(Pessimistic Lock)
>
>  대부분의 트랜잭션이 충돌할 것이라고 가정하는 방법이다.
>
>  개발자가 획득할 락을 직접 설정하고 트랜잭션이 시작과 동시에 개발자가 설정한 종류의 락을 획득하려고 시도하여 트랜잭션 간의 충돌을 방지한다.
>

- 낙관적 락 이용

  ```java
  // entity
  public class Lecture {
  	...
  	@Column(name = "remaining_seat", nullable = false)
  	private int remainingSeat;
      
      @Version
  	private Integer version;
  	...
  
  	public void registStudent() {
  		if (remainingSeat <= 0) {
  			throw new IllegalArgumentException("잔여 좌석이 없습니다");
  		}
  		this.remainingSeat -= 1;
  	}
  }
  ```

  위와 같이 엔티티에 `@Version`을 추가하여 낙관적 락을 적용한 뒤에 30명의 학생이 수강 인원이 20명인 강의에 대해 동시에 수강 신청을 하는 경우를 테스트한 결과, 중간에 버전 불일치에 의한 예외가 발생하는 것을 확인할 수 있었다.

  ```
  Resolved [org.springframework.orm.ObjectOptimisticLockingFailureException: Row was updated or deleted by another transaction (or unsaved-value mapping was incorrect)
  ```

  이를 해결하기 위해 아래와 같이 낙관적 락에서의 버전 불일치로 인한 트랜잭션 커밋 실패에 대해 재시도 로직을 추가하였다.

  ```java
  public class LectureService {
      ...
      @Retryable(
  			retryFor = ObjectOptimisticLockingFailureException.class,
  			maxAttempts = 5,
  			backoff = @Backoff(delay = 500),
  			recover = "recoverRegist"
  	)
      @Transactional
  	public void registStudentToLecture(final long lectureId, final long studentId) {
  		...
  	}
      ...
      @Recover
  	public void recoverRegist(final ObjectOptimisticLockingFailureException e, final long lectureId, final long studentId){
          ...
  	}
  }
  ```

  재시도 로직을 추가한 뒤에 30명의 학생이 수강 인원이 20명인 강의에 대해 동시에 수강 신청을 하는 경우를 테스트한 결과, 버전 불일치에 의한 커밋 실패에 정상적으로 재시도 로직이 실행되며 동시성 문제가 해결되는 것을 확인할 수 있었다.

  하지만 이는 요청에 대한 처리 결과가 요청 순서에 따라 보장되지 않고, 재시도의 성공 시점까지 포함하기 때문에 요청을 선착순으로 처리한다는 보장을 해주지 못 한다.

- 비관적 락 이용

  ```java
  public interface LectureRepository extends JpaRepository<Lecture, Long> {
  	@Lock(LockModeType.PESSIMISTIC_WRITE)
  	Optional<Lecture> findByLectureId(long id);
  }
  ```

  위와 같이 repository의 메서드에 비관적 락 설정을 추가한 뒤에 30명의 학생이 수강 인원이 20명인 강의에 대해 동시에 수강 신청을 하는 경우를 테스트한 결과, `select * from lecture where lecture_id = ? for update`쿼리가 실행되고 이후 트랜잭션이 커밋되기 전까지의 로직이 순차적으로 실행되며 요청 순서에 다른 로직의 실행을 보장하면서 동시성 문제가 해결되는 것을 확인할 수 있었다.

30개의 동시 요청이 발생하는 상황에서 낙관적 락을 사용했을 때와 비관적 락을 사용했을 때의 성능 측정 결과는 아래와 같다.

|                                | 낙관적 락(Optimistic Lock) | 비관적 락(Pessimistic Lock) |
| ------------------------------ | -------------------------- | --------------------------- |
| 요청당 평균 응답 속도(Average) | 368ms                      | 89ms                        |
| 단위 사간당 처리량(Throughput) | 4.0/sec                    | 1.1/sec                     |

측정 결과를 비교해보니 요청 하나당 처리 속도는 비관적 락이 빠른 편이지만 단위 시간당 처리량은 낙관적 락이 더 높은 것을 확인할 수 있다.
낙관적 락은 배타적 락에 의한 트랜잭션 대기가 적게 발생하는 대신 버전 불일치로 인한 트랜잭션 롤백 및 재시도에 의한 시간 지연이 존재하기 때문에 평균 응답 시간과 처리량이 높게 나타나는 것을 알 수 있다.
반면에 비관적 락은 충돌에 의한 롤백이 발생하지 않는 대신에 배타적 락에 의한 트랜잭션 대기가 빈번하게 발생하여 처리량이 낮고 평균 응답 속도가 빠른 것을 확인할 수 있었다.




-----

##### 참고자료 :

[https://zzang9ha.tistory.com](https://zzang9ha.tistory.com/443)

[https://mangkyu.tistory.com](https://mangkyu.tistory.com/299)

[https://tecoble.techcourse.co.kr](https://tecoble.techcourse.co.kr/post/2023-08-16-concurrency-managing/)

