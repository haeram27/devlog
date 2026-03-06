# Java virtual thread

- https://tech.kakaopay.com/post/ro-spring-virtual-thread/
- https://0soo.tistory.com/259

## 특징

- daemon thread
  - 가상 스레드는 플랫폼 스레드(OS 스레드)와 달리 JVM이 관리하는 경량 스레드로, 프로그램 종료를 막지 않는 데몬 스레드로 설계되었으며 Java 메인 프로그램(메인 스레드=일반 스레드) 종료시 함께 즉시 종료됨
  - 가상 스레드는 일반 스레드(OS 스레드)로 변경할 수 없음
- 무제한 사용 가능
  - 수만~수백만 개를 생성할 수 있으며, I/O 작업 중 블로킹되어도 OS 스레드를 점유하지 않아 효율적

## Plain Java

### 단일 Thread 생성

```Java
Thread.startVirtualThread(Runnable task);
Thread.ofVirtual.start(Runnable task);
```

### Executor 생성

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

## Spring

### VirtualThreadTaskExecutor 생성

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

### spring내 virtual thread 사용 활성화

- https://docs.spring.io/spring-boot/reference/features/spring-application.html

```Java
spring.threads.virtual.enabled=true
```
