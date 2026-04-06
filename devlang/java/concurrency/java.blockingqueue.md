# BlockingQueue

BlockingQueue는 멀티스레드 환경에서 스레드 안전(Thread-safe)하게 데이터(모든 데이터가 가능하지만 보통 Runnable/Callable) 교환하는 큐로, 큐가 비었을 때 take()하면 대기하고, 가득 찼을 때 put()하면 대기하는 특징이 있습니다. 주요 구현체로는 LinkedBlockingQueue, ArrayBlockingQueue, PriorityBlockingQueue, DelayQueue 등이 있습니다.

BlockingQueue는 대게 `ThreadPoolExecutor` 생성시 사용됩니다.

- `ThreadPoolExecutor` 생성하기

```java
    private static AtomicInteger threadId = new AtomicInteger();
    public static ExecutorService customGoodUnboundedCachedThreadPool() {
        int halfNumberOfThreads = Runtime.getRuntime().availableProcessors()/2;
        int maxNumberOfThreads = Runtime.getRuntime().availableProcessors();
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                    halfNumberOfThreads, maxNumberOfThreads,
                    60L, TimeUnit.SECONDS, // keep alive duration in idle status
                    new LinkedBlockingDeque<>(), // blocking queue
                    r -> {
                        Thread t = new Thread(r);
                        t.setName("executor-worker-" + threadId.accumulateAndGet(Integer.MAX_VALUE, (current, max) -> 
                            current >= max ? 0 : current + 1
                        ));
                        // t.setDaemon(true);
                        return t;
                    });
        // allow core thread to be terminated when it is idle, default is false and core thread is never terminated even if it is idle
        executor.allowCoreThreadTimeOut(true);
        return executor;
        // @formatter:on
    }
```

## 주요 BlockingQueue 종류

- `SynchronousQueue`: 용량이 0인 큐. 하나의 데이터를 삽입하면, 다른 스레드가 그 데이터를 가져갈 때까지 삽입 스레드가 대기합니다.
- `LinkedBlockingQueue`: 가변 크기 큐(연결 리스트 기반). 용량을 제한할 수도 있고(bounded), 제한 없이(unbounded) 사용할 수도 있습니다. 대개 ArrayBlockingQueue보다 높은 처리량(throughput)을 보입니다. 일반적으로 가장 많이 사용됨
- `ArrayBlockingQueue`: 고정 크기 블록킹 큐(배열 기반). 큐 생성 시 크기를 반드시 지정해야 하며, 큐가 꽉 차면 생산자 스레드가 대기합니다. LinkedBlockingQueue대비 요소간 메모리 접근성이 조금 빠른 장점이 있음
- `PriorityBlockingQueue`: 우선순위가 높은 요소가 먼저 나오는 큐. 큐 내부 요소들은 자연 순서(Comparable) 또는 Comparator를 통해 정렬됩니다. 크기 제한이 없습니다.
- `DelayQueue`: 요소가 만료 시간(Delay)을 가진 큐. 지정된 딜레이 시간이 지나야만 요소를 꺼낼 수 있습니다.

## BlockingQueue 작동 방식

- 데이터 삽입: put(e) (꽉 차면 대기), offer(e, time, unit) (제한 시간 동안 대기).
- 데이터 추출: take() (비어있으면 대기), poll(time, unit) (제한 시간 동안 대기).