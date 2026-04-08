# ThreadPoolExecutor RejectedxceptionHandler

- BlockingQueue의 add(), offer(), put()은 큐가 가득 찬 상황(Capacity Full)에서 실패를 처리하는 방식이 각각 다름

| 메서드 | 큐가 가득 찼을 때의 동작 | 리턴 타입 | 관련 예외 |
|---|---|---|---|
| add(e) | 예외 발생 | boolean (항상 true) | IllegalStateException |
| offer(e) | 즉시 false 반환 | boolean | 없음 |
| put(e) | 공간 생길 때까지 무한 대기 | void | InterruptedException |

**BlockingQueue 종류**

| 구현체 | 크기 제한 여부 | 특징 |
|---|---|---|
| LinkedBlockingQueue | 선택 가능 (Optional) | 가장 범용적, 보통 크기 제한해서 사용 |
| ArrayBlockingQueue | 필수 (Fixed) | 성능 중시, 고정된 자원 관리 |
| SynchronousQueue | 없음 (0) | 핸드오프(직접 전달) 방식,<br> maxPoolSize가 무제한인 cachedThreadPool에서 사용|
| PriorityBlockingQueue | 무제한 | 우선순위 기반 처리 |
| DelayQueue | 무제한 | 시간 지연 후 처리 |


- ThreadPoolExecutor에 BlockingQueue를 사용했을때 BlockingQueue에 task를 전달하는 method는 `offer()`이다.
  - 이유: execute() 메서드가 호출되었을 때 스레드가 무한정 대기(Blocking) 상태에 빠지면, 시스템 전체의 응답성이 심각하게 떨어질 수 있기 때문입니다.
  - 해결책: 만약 큐가 찼을 때 대기하게 만들고 싶다면, 커스텀 RejectedxceptionHandler 내에서 queue.put()을 호출하도록 직접 구현해야 합니다.

- ThreadPoolExecutor의 BlockingQueue가 가득차게 되면, 다음의 순서로 처리됨(신규 스레드를 생성해 보고 안되면 RejectedExecutionHandler가 호출)
  1. Core 스레드 체크: 현재 실행 중인 스레드가 corePoolSize보다 적으면 새 스레드를 생성해 즉시 실행합니다.
  2. 큐(Queue) 삽입: corePoolSize가 다 찼다면 작업을 BlockingQueue에 담으려고 시도합니다.
  3. Max 스레드 확장: 큐가 가득 찼다면, 현재 스레드 수가 maximumPoolSize보다 적은지 확인합니다. 적다면 새 스레드를 추가로 생성하여 작업을 즉시 실행합니다.
  4. 거절(Rejection): 큐도 꽉 차고, 스레드 수도 maximumPoolSize에 도달했다면 그때 비로소 RejectedExecutionHandler가 호출됩니다.

- 거절(Rejection) 시 발생하는 일 (Default & Custom)
  - RejectedExecutionHandler 설정에 따라 결과가 달라집니다.
    - AbortPolicy (기본값): RejectedExecutionException 예외를 던집니다. 작업은 버려집니다.
    - CallerRunsPolicy: 작업을 제출한 스레드(예: Main 스레드)가 직접 그 작업을 실행합니다. 이 과정에서 제출 속도가 자연스럽게 늦춰집니다(Back-pressure).
    - DiscardPolicy: 아무 일도 하지 않고 작업을 조용히 버립니다.
    - DiscardOldestPolicy: 큐의 맨 앞에 있는(가장 오래된) 작업을 삭제하고 새 작업을 다시 시도합니다.

## 예제 - RejectedExecutionHandler Policy 변경하여 ThreadPoolExecutor 생성
```java
import java.util.concurrent.*;

public class DiscardPolicyExample {
    public static void main(String[] args) {
        // 1. 설정: Core 2, Max 2, Queue 5 (최대 7개 작업 수용 가능)
        int corePoolSize = 2;
        int maxPoolSize = 2;
        int queueCapacity = 5;

        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                corePoolSize,
                maxPoolSize,
                10, TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(queueCapacity),
                // 2. DiscardPolicy 설정: 넘치는 작업은 그냥 무시함
                new ThreadPoolExecutor.DiscardPolicy()
        );

        // 3. 작업 10개 제출 (7개는 처리/대기, 3개는 버려짐)
        for (int i = 1; i <= 10; i++) {
            final int taskId = i;
            System.out.println("작업 제출 시도: " + taskId);
            
            executor.execute(() -> {
                try {
                    System.out.println("  [실행] 작업 " + taskId + " 시작 (" + Thread.currentThread().getName() + ")");
                    Thread.sleep(1000); // 작업을 오래 걸리게 함
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        System.out.println("--- 모든 작업 제출 완료 (예외 발생 안 함) ---");
        executor.shutdown();
    }
}
```

## 예제 - 커스텀 RejectedExecutionHandler 사용

```java
import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

public class CustomRejectedHandlerExample {

    public static void main(String[] args) {
        int corePoolSize = 2;
        int maxPoolSize = 4;
        long keepAliveTime = 10;
        int queueCapacity = 5;

        // 1. LinkedBlockingQueue 크기 제한 설정 (중요)
        BlockingQueue<Runnable> workQueue = new LinkedBlockingQueue<>(queueCapacity);

        // 2. RejectedExecutionHandler 구현 (대기 로직)
        RejectedExecutionHandler handler = (r, executor) -> {
            try {
                System.out.println("큐가 꽉 찼습니다! 작업 대기 중... : " + r.toString());
                // put() 메서드를 사용하여 큐에 공간이 생길 때까지 블로킹(대기)
                executor.getQueue().put(r);
                // offer(e, time, unit) 형식을 사용하여 큐에 자리가 날 때까지 지정된 시간 동안만 기다리게 할 수 있습니다.
                // executor.getQueue().offer(r, 10, TimeUnit.SECOND);
                System.out.println("큐에 공간이 생겨 작업을 다시 넣었습니다.");
            } catch (InterruptedException e) {
                // 인터럽트 발생 시 작업 처리 실패 처리
                Thread.currentThread().interrupt();
                throw new RejectedExecutionException("작업 처리 중 인터럽트 발생", e);
            }
        };

        // 3. ThreadPoolExecutor 생성
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
                corePoolSize,
                maxPoolSize,
                keepAliveTime,
                TimeUnit.SECONDS,
                workQueue,
                Executors.defaultThreadFactory(),
                handler // 커스텀 핸들러 등록
        );

        // 4. 테스트 작업 제출
        for (int i = 0; i < 15; i++) {
            final int taskId = i;
            executor.execute(() -> {
                try {
                    System.out.println("작업 실행 중: " + taskId + " | Thread: " + Thread.currentThread().getName());
                    Thread.sleep(1000); // 작업 시간 1초
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }

        executor.shutdown();
    }
}
```