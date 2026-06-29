# Condition 인터페이스 (멀티쓰레드 동기화)

Condition은 ReentrantLock과 함께 사용되어 스레드 간의 동기화(wait/notify)를 더 세밀하게 제어하는 인터페이스입니다. lock.newCondition()으로 생성하며, await(), signal(), signalAll() 메서드를 사용하여 특정 조건이 충족될 때까지 스레드를 대기시키거나 깨웁니다

주요 메서드:

- lock.newCondition(): 새로운 Condition 객체를 생성합니다.
- await(): 쓰레드를 대기 상태로 전환합니다.
- signal(): 대기 중인 쓰레드 하나를 깨웁니다.
- signalAll(): 대기 중인 모든 쓰레드를 깨웁니다.

## Lock과 Condition의 연동

### Lock과 Condition의 역할

Lock(ReentrantLock 등)은 임계영역(Critical Section: 멀티 스레드 사이에 동시성 문제가 발생 가능한 코드 영역)을 지정하고 임계 영역에 진입하는 스레드가 단일 하도록 만드는데 사용한다.

Condition은 임계 영역 내에서 스레드를 대기 시키거나 깨우는데 사용한다.
Condition을 사용하려면 Lock의 소유권을 반드시 획득(`Lock.lock()` 호출)해야 한다.
그러므로 Condition 객체에 대한 호출(await, signal, signalAll) 등은 반드시 Lock을 이용해 지정된 임계 영역 내에서만 이루어 져야 한다.

- Lock과 Condition 사용 패턴

```java
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();

lock.lock(); // 스레드의 lock 소유권 획득, 임계 영역 시작
try {
    // try bloc의 내부가 임계 영역이다. 한번에 하나의 스레드만 접근 가능하다
    ...
    condition.await() or condition.signal()
} finally {
    lock.unlock(); // 반드시 락 해제, 임계 영역 종료
}
```

Condition.await()를 호출하려면 해당 Condition 객체를 생성한 Lock(주로 ReentrantLock)의 소유권(Monitor와 유사한 권한)을 반드시 먼저 확보해야 합니다.

Lock의 소유권 획득은 Lock의 lock() 메소드를 호출하여 획득 합니다.

- 원자적 작업: await()가 호출되면 내부적으로 "Lock을 해제"함과 동시에 "Thread를 대기 상태(Wait Set)로 전환"하는 작업이 원자적으로 일어나야 합니다. Lock을 들고 있지 않다면 Lock을 해제할 권한 자체가 없기 때문입니다.
- 상태 일관성: 보통 await()는 특정 조건(예: 큐가 비었는지 확인)을 체크한 뒤에 호출합니다. 이 조건을 체크하는 동안 다른 쓰레드가 값을 바꿔버리면 안 되기 때문에 Lock으로 보호된 임계 구역 내에서 실행되어야 합니다.

만약 Lock 없이 호출하면?

만약 lock.lock()을 통해 lock을 획득하지 않은 상태에서 condition.await()나 signal()을 호출하면, 자바에서는 실행 시점에 IllegalMonitorStateException이 발생합니다.

## 예제

### pub/sub 구조

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ConditionExample {
    private final Lock lock = new ReentrantLock();
    private final Condition condition = lock.newCondition();
    private boolean ready = false;

    public void waitForReady() throws InterruptedException {
        lock.lock(); // acquire lock - begin critical section
        try {
            while (!ready) {
                System.out.println("준비될 때까지 대기 중...");
                condition.await(); // ready가 true가 될 때까지 대기
            }
            System.out.println("준비 완료! 실행합니다.");
        } finally {
            lock.unlock(); // 5. release lock - end critical section
        }
    }

    public void makeReady() {
        lock.lock();
        try {
            ready = true;
            condition.signal(); // 대기 중인 쓰레드에게 신호를 보냄
        } finally {
            lock.unlock();
        }
    }
}
```

### pub/sub 구조

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class BoundedBuffer {
    private final Lock lock = new ReentrantLock();
    // 두 개의 대기실: '가득 찼을 때 대기하는 곳'과 '비었을 때 대기하는 곳'을 분리
    private final Condition notFull  = lock.newCondition(); 
    private final Condition notEmpty = lock.newCondition(); 

    private final Object[] items = new Object[10]; // 크기가 10인 버퍼
    private int putptr, takeptr, count;

    // producer
    public void put(Object x) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length) { // 1. 버퍼가 가득 찼으면
                System.out.println("버퍼 가득 참, 대기 중...");
                notFull.await(); // 2. '가득 차지 않을 때까지' 대기실에서 대기 (락 반납)
            }
            items[putptr] = x;
            if (++putptr == items.length) putptr = 0;
            count++;
            
            notEmpty.signal(); // 3. 이제 비어있지 않으니, 기다리는 소비자 하나 깨움
        } finally {
            lock.unlock();
        }
    }

    // consumer
    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) { // 4. 버퍼가 비었으면
                System.out.println("버퍼 비어 있음, 대기 중...");
                notEmpty.await(); // 5. '비지 않을 때까지' 대기실에서 대기 (락 반납)
            }
            Object x = items[takeptr];
            if (++takeptr == items.length) takeptr = 0;
            count--;

            notFull.signal(); // 6. 이제 공간이 생겼으니, 기다리는 생산자 하나 깨움
            return x;
        } finally {
            lock.unlock();
        }
    }
}
```