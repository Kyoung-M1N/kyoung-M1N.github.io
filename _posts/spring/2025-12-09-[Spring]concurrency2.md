---
title: 애플리케이션에서의 동시성 처리
author: kymin
date: 2025-12-09 13:58
categories: [Spring]
tags: [spring, web, java]



---

## **동시성 문제 해결 위치**

### **데이터베이스와 애플리케이션에서의 동시성 문제**

지난번 포스팅에서 동시성 문제를 해결하기 위해 낙관적 락, 비관적 락을 비교해보았다. 두 방법은 모두 데이터베이스에서 트랜잭션이 작업을 처리하는 도중에 락을 이용하여 동일 자원에 동시 접근을 방지하거나 동시 접근을 감지하여 재시도하는 방법으로 동시성 문제를 해결했다.

하지만 데이터베이스 레벨에서의 동시성 처리는 아래와 같은 단점이 있다.

- 데이터베이스가 분산된 환경이라면 락이 적용되지 않아 동시성 처리를 해결할 수 없다.
- 데이터베이스에 요청이 발생한 뒤에 대기 또는 재시도가 발생하므로 성능이 저하될 수 있다.
- 데이터베이스 뿐만 아니라 외부 API를 호출하는 경우에 대한 중복 요청 발생을 방지할 수 없다.

결국 비관적 락과 낙관적 락은 무조건 데이터베이스에 요청이 전달되므로 서비스에 발생하는 요청이 많아지면 데이터베이스에도 요청이 많아지고, 이로 인한 트랜잭션 대기나 재시도로 인해 커넥션 사용이 증가하며 성능이 저하될 수 있다. 하지만 이를 해결하고자 데이터베이스에 수평 확장을 진행한다면 동시성 문제는 다시 발생하게 되며, 외부 API의 중복 호출은 처음부터 예방할 수 없다.

이를 해결하기 위해 애플리케이션 레벨에서의 동시성 처리 방법도 존재한다. 애플리케이션 레벨에서의 동시성 처리가 가지는 장점은 아래와 같다.

- 애플리케이션에서 락을 관리하므로 동시성 처리 과정에서 트랜잭션 대기나 재시도 로직으로 인한 인한 커넥션 점유가 적다.
- 분산 락의 경우 서버나 데이터베이스가 분산된 환경에서도 적용 가능하다.
- 락의 대상이 임계 영역이기 때문에 외부 API의 중복 호출까지 방지할 수 있다.

## **애플리케이션 레벨에서의 동시성 처리**

애플리케이션 레벨에서 동시성 문제를 해결하는 방법으로는 대표적으로 재진입 락(ReentrantLock)과 분산 락(Distributed Lock)이 있다.

### **재진입 락(ReentrantLock)**

재진입 락(ReentrantLock)은 스레드 간의 동기화를 위한 자바 표준 라이브러리의 기능이다.

코드 상에서 임계 구역을 잠금 영역으로 설정할 수 있으며, 락의 획득과 해제를 수동으로 관리할 수 있다.

락의 대상이 애플리케이션의 임계 구역이므로 낙관적 락이나 비관적 락과 달리 샤딩을 통해 데이터베이스를 분산하여 사용하는 경우에도 동시성 문제를 해결할 수 있다.

아래와 같이 락을 관리 및 획득, 해제하는 빈을 생성한다.

```java
@Component
public class ReentrantLockManager {
	// fair를 true로 설정하여 대기 시간이 오래된 요청부터 처리
	private final ReentrantLock lock = new ReentrantLock(true);

	public <T> T executeWithReentrantLock(Supplier<T> task) {
		return executeInternalWithReentrantLock(task);
	}

	public void executeWithReentrantLock(Runnable task) {
		executeInternalWithReentrantLock(() -> {
			task.run();
			return null;
		});
	}

	private <T> T executeInternalWithReentrantLock(Supplier<T> task) {
		boolean isLocked = false;
		try {
			isLocked = lock.tryLock(5, TimeUnit.SECONDS);
			if (!isLocked) {
				throw new LockAcquisitionTimeoutException(ErrorCode.LOCK_ACQUISITION_TIMEOUT);
			}
			return task.get();

		} catch (InterruptedException e) {
			// 인터럽트 플래그를 복원
			Thread.currentThread().interrupt();
			throw new ServerInternalException(ErrorCode.THREAD_INTERRUPT);
		} finally {
			if (isLocked) {
				lock.unlock();
			}
		}
	}
}
```

개인이나 팀의 스타일에 따라 다르지만 재사용성을 높이기 위해 `Supplier`나 `Runnable`로 임계 구역에 해당하는 로직을 주입받도록 구현한 뒤 외부에서 호출한다.

```java
@Service
@RequiredArgsConstructor
public class LectureFacadeService {
	private final LectureService lectureService;
	private final ReentrantLockManager reentrantLockManager;
    ...

	public void tryRegisterStudentToLecture(final long lectureId, final long studentId) {
		reentrantLockManager.executeWithReentrantLock(
				() -> lectureService.registerStudentToLecture(lectureId, studentId)
		);
    ...
	}
}
```

주의해야할 점은 임계 구역에 해당하는 메서드가 트랜잭션 내에서 동작할 경우 재진입 락으로 로직을 실행하는 부분과 클래스를 분리해야 한다.

Spring의 `@Transactional`은 프록시 기반 AOP이므로 외부 클래스에서 해당 메서드를 호출한 경우에만 트랜잭션이 정상적으로 동작하기 때문이다.

> `@Transactional`에 의한 트랜잭션은 해당 빈의 메서드를 호출했을 때 해당 빈을 프록시 객체로 감싼 뒤에 프록시가 메서드의 호출을 가로채서 동작하게 된다.
>
> 따라서 애플리케이션 내에서 락을 적용하거나 스레드간의 동기화를 처리하는 로직이 트랜잭션내에서 동작하는 로직과 같은 클래스에 존재할 경우 프록시를 거치지 않으므로 트랜잭션의 동작 전체가 동기화되지 않고, 트랜잭션의 생성과 커밋, 롤백을 제외한 내부 로직만 동기화가 적용된다.

위와 같이 재진입 락을 구현한 뒤에 30명의 학생이 수강 인원이 20명인 강의에 대해 동시에 수강 신청을 하는 경우를 테스트한 결과, `lecture`테이블의 잔여 좌석에 대한 카운트가 정상적으로 동작하였고, 수강 정보인 `lecture_student`에 등록된 학생의 수와 일치하는 것을 확인할 수 있었다.

```sql
+----------------+------------+------------+-----------------------+
| remaining_seat | total_seat | lecture_id | title                 |
+----------------+------------+------------+-----------------------+
|              0 |         20 |          1 | 컴퓨터구조              |
+----------------+------------+------------+-----------------------+
```

하지만 위의 방법에 대해 성능 지표를 측정한 결과 평균 응답 속도 67ms, 처리량 0.618/sec로 낙관적 락이나 비관적 락보다는 응답 속도가 조금 빠르지만 처리량이 매우 낮은 것을 확인할 수 있었다.

이는 락에 의한 대기가 데이터베이스가 아닌 애플리케이션에서 발생하기 때문임을 추측할 수 있다. 재진입 락은 JVM 내부에서 `AbstractQueuedSynchronzier`에 의해 대기되는 스레드가 FIFO 또는 priority queue로 관리되어 디스크 IO나 스케줄러의 동작 등이 발생하지 않는다. 따라서 재진입 락에서의 작업 스레드 전환을 위한 컨텍스트 스위칭 비용이 매우 낮지만, DB락은 트랜잭션 대기로 인한 커넥션 점유로 추가적인 대기가 발생하고 DBMS 내부적으로 스케줄러의 동작 등으로 인해 컨텍스트 스위칭 비용이 높아 상대적으로 응답 속도가 느려지게 된다.

반면에 모든 요청을 병렬로 처리한 뒤에 충돌을 감지하는 낙관적 락이나, 공유 자원에 접근하는 쿼리에 대해서만 락을 적용하는 비관적 락에 비해, 재진입 락은 임계 구역으로 지정하는 전체 로직을 순차 처리하기 때문에 처리량이 훨씬 떨어지게 된다.

따라서 이를 해결하기 위해 재진입 락이 적용될 단위를 설정하여 병렬성을 높여주는 것이 바람직하다.

아래와 같이 `ConcurrentHashMap`등을 이용하여 락을 특정 단위 기준으로 묶어주는 방법으로 병렬성을 높일 수 있다.

```java
@Component
public class ReentrantLockManager<K> {
	// private final ReentrantLock lock = new ReentrantLock(true);
	private final ConcurrentHashMap<K, LockWrapper> locks = new ConcurrentHashMap<>();

	public <T> T executeWithReentrantLock(K lockKey, Supplier<T> task) {
		return executeInternalWithReentrantLock(lockKey, task);
	}

	public void executeWithReentrantLock(K lockKey, Runnable task) {
		executeInternalWithReentrantLock(lockKey, () -> {
			task.run();
			return null;
		});
	}

	private <T> T executeInternalWithReentrantLock(K lockKey, Supplier<T> task) {
		// fair를 true로 설정하여 대기 시간이 오래된 요청부터 처리
		LockWrapper wrapper = locks.computeIfAbsent(lockKey, key -> new LockWrapper());
		boolean isLocked = false;

		wrapper.refCount.getAndIncrement();

		try {
			isLocked = wrapper.lock.tryLock(5, TimeUnit.SECONDS);
			if (!isLocked) {
				throw new RuntimeException("타임아웃으로 인한 락 획득 실패");
			}
			return task.get();

		} catch (InterruptedException e) {
			// 인터럽트 플래그를 복원
			Thread.currentThread().interrupt();
			throw new RuntimeException("스레드 인터럽트 발생", e);
		} finally {
			if (isLocked) {
				wrapper.lock.unlock();
			}
			if (wrapper.refCount.decrementAndGet() == 0
					&& !wrapper.lock.hasQueuedThreads()
					&& !wrapper.lock.isLocked()) {
				locks.remove(lockKey, wrapper);
			}
		}
	}

	private static class LockWrapper {
		final ReentrantLock lock = new ReentrantLock(true);
		final AtomicInteger refCount = new AtomicInteger(0);
	}
}
```

위와 같이 키단위로 스레드 동기화를 진행하도록 구현하는 과정에서 주의할 점은 동시성 문제가 발생할 여지가 있는 단위로 동기화 단위를 구성해야한다. 따라서 중복 호출을 제어해야하는 객체 단위, 공유 자원에 접근하는 경우를 파악한 뒤에 적절한 값을 락을 적용하는 단위로 사용해야한다.

30명의 학생이 강의1과 강의 2에 수강 신청을 동시에 요청하는 경우에 대한 성능 지표를 측정한 결과 평균 응답 속도 79ms, 처리량은 51.1/sec로 측정되었다. 재진입 락을 모든 요청에 적용했을 때보다 락의 단위를 강의 기준으로 설정한 경우에 응답 속도와 처리량이 향상됨을 알 수 있다. 특히 처리량은 약 82배 정도의 차이를 보임을 확인할 수 있다.

강의 단위로 락을 분할한 이후에는 DB 커넥션의 동시 활용도가 증가했을 뿐만 아니라, 스레드들이 단일 락에 대기하지 않게 되면서 락 획득·해제 과정에서 발생하던 컨텍스트 스위칭과 스케줄링 오버헤드가 크게 감소하는 효과가 발생한다.

이에 따라 스레드들이 단일 락에 장시간 대기하지 않게 되어 `RUNNABLE`상태와 `BLOCKED`의 상태 전환 빈도가 감소하고, `AbstractQueuedSynchronzier`와 OS에서의 스케줄링과 컨텍스트 스위칭이 감소하게 되며 락 경합에 의한 병목상태가 해소되었기 때문으로 해석할 수 있다.

> **`synchronized`와의 차이점**
>
> 재진입 락은 스레드 간의 동기화를 위해 사용된다는 점에서 `synchronized`와 유사하지만 락 획득의 제어 과정과 동작에서 차이점이 존재한다.






-----

##### 참고자료 :

[https://medium.com/sopt-makers](https://medium.com/sopt-makers/%EB%82%B4%EA%B0%80-%EC%96%B8%EC%A0%9C-2%EB%AA%85%EC%9D%B4-%EB%90%90%EC%A7%80-sopt-%EB%AA%A8%EC%9E%84-%EC%8B%A0%EC%B2%AD-%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9D%B4%EC%8A%88-%ED%95%B4%EA%B2%B0%EA%B8%B0-7c78105d7be4)

[https://miiiinju.tistory.com](https://miiiinju.tistory.com/27)