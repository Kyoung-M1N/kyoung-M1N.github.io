---
title: 스레드 동기화 매커니즘
author: kymin
date: 2026-05-13 13:58
categories: [Java]
tags: [spring, web, java]



---

## **스레드 동기화**

스레드 동기화는 여러 스레드가 동시에 공유 자원에 접근할 때 발생하는 경쟁 조건을 방지하고 데이터의 일관성을 보장하기 위한 기법이다.

대표적인 스레드 동기화 매커니즘으로는 뮤텍스, 세마포어, 모니터가 있다.

### **뮤텍스(Mutex)**

뮤텍스(Mutex)는 MUTual EXclusion의 약자로, 여러 스레드가 공유 자원에 동시에 접근하는 것을 막기 위한 동기화 기법이다.

뮤텍스는 락(Lock)의 개념을 기반으로 동작한다. 스레드가 임계 구역(Critical Section)에 진입하려면 먼저 뮤텍스 락을 획득해야 하며, 이미 다른 스레드가 락을 보유하고 있다면 해당 스레드가 락을 반환할 때까지 대기해야 한다.

> **임계 구역(Critical Section)**
>
> 임계 구역은 공유 자원에 접근하는 코드 영역으로, 동시에 하나의 스레드만 실행될 수 있어야 하는 구간을 의미한다.

뮤텍스의 핵심적인 특징은 **소유권(Ownership)**이다. 락을 획득한 스레드만이 해당 락을 해제할 수 있으며, 다른 스레드는 락을 소유한 스레드가 해제하기 전까지 임계 구역에 진입하지 못한다.

```java
ReentrantLock mutex = new ReentrantLock();

mutex.lock();
try {
    // 임계 구역: 공유 자원에 접근하는 코드
} finally {
    mutex.unlock();    // 락을 획득한 스레드만 해제 가능
}
```

Java에서는 위와 같이 `ReentrantLock`을 이용하여 뮤텍스를 구현할 수 있다. `finally` 블록에서 `unlock()`을 호출하는 것은 예외가 발생하더라도 반드시 락이 해제될 수 있도록 하기 위함이다.

이진 세마포어(Binary Semaphore)와 유사하지만, 뮤텍스는 반드시 락을 획득한 스레드만 락을 해제할 수 있다는 점에서 차이가 있다.

### **세마포어(Semaphore)**

세마포어(Semaphore)는 동시에 임계 구역에 접근할 수 있는 스레드의 수를 제어하는 동기화 기법이다.

세마포어는 공유 자원에 접근 가능한 스레드의 수를 나타내는 카운터(Counter)를 가지며, 두 가지 연산을 통해 동작한다.

- `acquire`(P 연산, wait): 카운터를 1 감소시키고, 카운터가 0 이하이면 스레드를 대기 상태로 전환한다.
- `release`(V 연산, signal): 카운터를 1 증가시키고, 대기 중인 스레드가 있으면 깨운다.

세마포어는 카운터의 초기 값에 따라 두 가지 종류로 구분된다.

> - **이진 세마포어(Binary Semaphore)**
>
>   카운터의 초기 값이 1인 세마포어로, 동시에 하나의 스레드만 임계 구역에 진입할 수 있다. 뮤텍스와 유사하게 동작하지만, 락을 획득한 스레드가 아닌 다른 스레드도 락을 해제할 수 있다는 점에서 차이가 있다.
>
> - **카운팅 세마포어(Counting Semaphore)**
>
>   카운터의 초기 값이 1보다 큰 세마포어로, 설정한 수만큼의 스레드가 동시에 임계 구역에 진입할 수 있다.

```java
Semaphore semaphore = new Semaphore(3);    // 동시에 3개의 스레드만 접근 허용

semaphore.acquire();    // 카운터 감소, 카운터가 0이면 대기
try {
    // 임계 구역: 공유 자원에 접근하는 코드
} finally {
    semaphore.release();    // 카운터 증가, 대기 중인 스레드 깨움
}
```

Java에서는 `java.util.concurrent.Semaphore`를 이용하여 동시에 접근 가능한 스레드의 수를 제한할 수 있다. 이를 활용하면 데이터베이스 커넥션 풀의 최대 연결 수를 제한하거나 외부 API에 대한 동시 호출 수를 제어하는 등의 상황에 적용할 수 있다.

### **모니터(Monitor)**

모니터(Monitor)는 뮤텍스와 컨디션 변수(Condition Variable)를 결합한 고수준의 동기화 기법이다.

뮤텍스와 세마포어는 개발자가 직접 락의 획득과 해제를 관리해야 하므로 실수가 발생하기 쉽다. 반면 모니터는 공유 자원에 접근하는 메서드에 대한 상호 배제를 자동으로 보장하여 보다 안전하게 동기화를 구현할 수 있다.

Java에서는 `synchronized` 키워드를 통해 모니터를 활용할 수 있다. Java의 모든 객체는 내부적으로 모니터를 가지고 있으며, `synchronized` 키워드가 적용된 블록이나 메서드에 진입하려는 스레드는 해당 객체의 모니터 락을 획득해야 한다.

```java
public class SharedResource {
    private int count = 0;

    public synchronized void increment() {
        count++;    // 모니터 락을 획득한 스레드만 진입 가능
    }

    public synchronized int getCount() {
        return count;
    }
}
```

또한 모니터는 컨디션 변수를 통해 특정 조건이 만족될 때까지 스레드를 대기시키거나 깨우는 기능을 제공한다. Java에서는 `wait()`, `notify()`, `notifyAll()` 메서드를 통해 이를 구현할 수 있다.

```java
public class SharedResource {
    private boolean isReady = false;

    public synchronized void waitForCondition() throws InterruptedException {
        while (!isReady) {
            wait();    // 조건이 충족될 때까지 모니터 락을 반납하고 대기
        }
        // 조건이 충족된 이후 실행할 코드
    }

    public synchronized void signalCondition() {
        isReady = true;
        notify();    // 대기 중인 스레드 중 하나를 깨움
    }
}
```

`wait()`를 호출한 스레드는 모니터 락을 반납하고 대기 상태가 된다. 이후 다른 스레드에서 `notify()` 또는 `notifyAll()`이 호출되면 대기 중인 스레드가 다시 락 획득을 시도하며 실행을 재개한다.

> **`synchronized`와 `ReentrantLock`의 차이**
>
> `synchronized`는 Java 언어 수준에서 모니터를 간단하게 사용할 수 있는 방법이지만, 락의 획득과 해제를 세밀하게 제어하기 어렵다는 단점이 있다.
>
> 반면 `ReentrantLock`은 락 획득 시도에 타임아웃을 설정하거나(`tryLock`), 공정성(fairness) 정책을 적용하거나, 복수의 컨디션 변수를 생성하는 등 `synchronized`보다 더 유연하게 동기화를 구현할 수 있다.
>
> ```java
> ReentrantLock lock = new ReentrantLock(true);    // fair=true로 설정 시 대기 시간이 오래된 스레드 우선 처리
> Condition condition = lock.newCondition();
>
> lock.lock();
> try {
>     while (!isReady) {
>         condition.await();    // wait()에 해당
>     }
> } finally {
>     lock.unlock();
> }
>
> lock.lock();
> try {
>     isReady = true;
>     condition.signal();    // notify()에 해당
> } finally {
>     lock.unlock();
> }
> ```
>
> 또한 `synchronized`는 블록을 벗어날 때 자동으로 락이 해제되므로 락을 반납하지 않는 실수를 방지할 수 있지만, `ReentrantLock`은 반드시 `finally` 블록에서 `unlock()`을 직접 호출해야 한다.

세 가지 동기화 매커니즘을 비교하면 아래와 같다.

|                     | 뮤텍스(Mutex)  | 세마포어(Semaphore)            | 모니터(Monitor)                |
| ------------------- | -------------- | ------------------------------ | ------------------------------ |
| 동시 접근 스레드 수 | 1개            | N개 (카운터 초기값)            | 1개                            |
| 락 해제 주체        | 락 획득 스레드 | 모든 스레드                    | 락 획득 스레드                 |
| 조건 대기 지원      | 미지원         | 미지원                         | 지원 (컨디션 변수)             |
| Java 구현           | ReentrantLock  | java.util.concurrent.Semaphore | synchronized, wait/notify      |


-----

##### 참고자료 :

[https://medium.com/sopt-makers](https://medium.com/sopt-makers/%EB%82%B4%EA%B0%80-%EC%96%B8%EC%A0%9C-2%EB%AA%85%EC%9D%B4-%EB%90%90%EC%A7%80-sopt-%EB%AA%A8%EC%9E%84-%EC%8B%A0%EC%B2%AD-%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9D%B4%EC%8A%88-%ED%95%B4%EA%B2%B0%EA%B8%B0-7c78105d7be4)