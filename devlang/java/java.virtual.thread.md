# Java virtual thread

- https://tech.kakaopay.com/post/ro-spring-virtual-thread/
- https://0soo.tistory.com/259

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
