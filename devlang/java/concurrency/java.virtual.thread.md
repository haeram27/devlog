# Java virtual thread

- https://tech.kakaopay.com/post/ro-spring-virtual-thread/
- https://0soo.tistory.com/259

## 사용법

- `ThreadPool`을 사용하지 말라, 가상 스레드는 생성 후 재사용 하지 말고, 필요할 때마다 신규로 생성하는 방식으로 사용한다
- Platform Thread를 사용하는 기존의 비동기 실행 메소드 API 사용을 지양하고 Blocking이 예상되는 동기 메소드를 가상 스레드로 실행시켜라

## 특징

- daemon thread
  - 가상 스레드는 플랫폼 스레드(OS 스레드)와 달리 JVM이 관리하는 경량 스레드로, 프로그램 종료를 막지 않는 `데몬 스레드`로 설계되었으며 Java 메인 프로그램(메인 스레드=일반 스레드) 종료시 함께 즉시 종료됨
  - 가상 스레드는 일반 스레드(OS 스레드)로 변경할 수 없음
- 무제한 사용 가능
  - 수만~수백만 개를 생성 가능
  - 생성 및 컨텍스트 스위칭 비용이 매우 저렴하여 I/O 작업 중 블로킹되어도 OS 스레드를 점유하지 않아 효율적
- 설계 철학 및 사용 방법
  - 별도의 `ThreadPool`을 사용하지 않는 것이 가상 스레드의 사용 목적이다.
  - Platform Thread(OS 코어 쓰레드)를 사용할 때는 대규모 동시성 작업을 위해서는 `ThreadPool`을 반드시 사용하여 실제 동시에 실행되는 스레드를 수를 제약하는 복잡한 설계(`ExecutorService` 등)를 사용했어야 했으나 가상 스레드 사용시에는 사용을 지양한다.
  - 가상 스레드의 동시 실행 개수 조절을 위해서는 `Semaphore`를 사용한다.
  - Java 코드 상 메소드 중 설계 부터 비동기를 고려한 메소드 들이 있는데(`HttpClient.sendAsync()` 등) 이들을 사용하지 말고 동기화 메소드를 호출하고 Blocking 발생 지점을 가상 스레드로 감싸는 것이 훨씬 효율적이다.

## 단일 Thread 생성

```Java
// 즉시 실행
Thread t = Thread.startVirtualThread(Runnable task);

// 설정 가능
Thread t = Thread.ofVirtual().name("my-vthread").start(Runnable task);
```

#### `Thread.ofVirtual()`에서 사용 가능한 메소드 목록

1. 속성 설정 메서드 (Configuration)
스레드의 이름, 예외 처리 등 핵심 속성을 정의합니다.

- name(String name): 스레드의 이름을 직접 지정합니다.
- name(String prefix, long start): 지정한 접두어(prefix)에 숫자를 붙여 스레드 이름을 자동 생성합니다. 여러 스레드를 만들 때 유용합니다.
- inheritInheritableThreadLocals(boolean inherit): 부모 스레드의 InheritableThreadLocal 변수 값을 상속받을지 여부를 결정합니다. (기본값: true)
- uncaughtExceptionHandler(Thread.UncaughtExceptionHandler ueh): 스레드 실행 중 예외가 발생했을 때 이를 처리할 핸들러를 등록합니다.

2. 생성 및 실행 메서드 (Terminal)
설정을 마친 후 실제로 스레드나 팩토리를 생성하는 최종 단계 메서드입니다.

- start(Runnable task): 설정한 속성으로 가상 스레드를 즉시 생성하고 시작합니다.
- unstarted(Runnable task): 설정한 속성으로 가상 스레드 객체를 생성하지만, 시작은 하지 않습니다. 나중에 thread.start()로 직접 실행해야 합니다.
- factory(): 설정한 속성을 그대로 사용하는 ThreadFactory 객체를 반환합니다. ExecutorService 등에서 스레드를 반복 생성할 때 사용합니다.

Thread 설정 예:

```java
Thread thread = Thread.ofVirtual()
    .name("my-virtual-", 0)                  // 이름 패턴 설정
    .uncaughtExceptionHandler((t, e) -> ...) // 예외 핸들러 설정
    .start(() -> {                           // 즉시 시작
        System.out.println("가상 스레드 실행 중");
    });
```

## Executor 생성

### Plain Java

```java
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

// ThreadFactory 사용
ThreadFactory factory = Thread.ofVirtual()
                              .name("vthread-pool-", 0) // 0부터 시작하는 카운터
                              .factory();
ExecutorService executor = Executors.newThreadPerTaskExecutor(factory);
```

- `Callable` 결과 취합하기

```java
    var list = new ArrayList<Integer>();
    for (int i=0; i<300; i++) list.add(i);

    // Executors.newVirtualThreadPerTaskExecutor() makes unbounded size thread 
    try (var es = Executors.newVirtualThreadPerTaskExecutor()) {
        List<Future<Integer>> futures = list.stream()
            .map(i -> es.submit(() -> {
                log.info("val: "+ i);
                Thread.sleep(1000);
                return i;
            }))
            .toList();

        // Future::isDone() is non-blocking method, just check future's done status when it invoked
        boolean allDone = futures.stream().allMatch(Future::isDone);
        log.info("alldone: " + allDone); // alldone: false here

        for (var f : futures) {
            // Future.get() block invoking current thread
            log.info("result: "+ f.get(2, java.util.concurrent.TimeUnit.SECONDS));
        }
    } catch (Exception e) {
        log.error(e.getMessage(), e);
        }
```

### Spring 전용

- VirtualThreadTaskExecutor

```Java
@EnableAsync
@Configuration
public class AsyncConfig {
    private static final String ASYNC_THREAD_PREFIX = "some-task-";
    @Bean
    public Executor someTaskExecutor() {
        var executor = new TaskExecutorAdapter(new VirtualThreadTaskExecutor(ASYNC_THREAD_PREFIX));
        executor.setTaskDecorator(new TaskDecorator());
        return executor;
    }
}
```

### spring내 virtual thread 사용 활성화 설정

- https://docs.spring.io/spring-boot/reference/features/spring-application.html

```Java
spring.threads.virtual.enabled=true
```
