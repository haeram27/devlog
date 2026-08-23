# Java Structured Concurrency (`StructuredTaskScope`)

- https://openjdk.org/jeps/505 (JDK 25, 4차 Preview)
- https://openjdk.org/jeps/499 (JDK 24, 3차 Preview)
- https://openjdk.org/jeps/453 (JDK 21, 1차 Preview)
- https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/concurrent/StructuredTaskScope.html

## ⚠️ Preview API 주의

- `StructuredTaskScope`는 아직 **Preview API** 이다 (JDK 19에서 Incubator로 시작 → JDK 21부터 Preview → JDK 25 기준 4차 Preview 진행 중, 정식(finalize) 채택 전).
- 사용하려면 컴파일/실행 시 `--enable-preview` 옵션이 반드시 필요하다.

```bash
javac --release 25 --enable-preview Main.java
java --enable-preview Main
```

- JDK 버전에 따라 API 형태가 계속 바뀌었다. 특히 **JDK 21~24**와 **JDK 25**는 API 모양이 다르므로 사용중인 JDK 버전을 반드시 확인해야 한다.
  - JDK 21~24: `new StructuredTaskScope<>()` 생성자 + `ShutdownOnFailure` / `ShutdownOnSuccess` 서브클래스 사용
  - JDK 25~: `StructuredTaskScope.open()` 정적 팩토리 메소드 + `Joiner` 인터페이스 사용 (아래 본문은 JDK 25 기준으로 설명)
- 이 문서는 **JDK 25 기준(JEP 505)** API를 기준으로 설명하며, 하위 버전과의 차이는 마지막 "버전별 API 변화" 섹션에 정리한다.

## 왜 필요한가? (초급)

### 기존 `ExecutorService` 방식의 문제점

여러 개의 하위 작업(subtask)을 동시에 실행하고 결과를 합쳐야 하는 상황을 생각해보자.

```java
Response handle() throws ExecutionException, InterruptedException {
    Future<String> user  = executor.submit(() -> findUser());
    Future<Integer> order = executor.submit(() -> fetchOrder());

    String theUser = user.get();   // findUser 결과 대기
    int theOrder = order.get();    // fetchOrder 결과 대기

    return new Response(theUser, theOrder);
}
```

이 코드는 아래와 같은 문제를 갖는다.

1. **Thread Leak(스레드 누수)**: `findUser()`가 예외를 던지면 `handle()`은 실패하지만, `fetchOrder()`는 아무도 취소하지 않으므로 백그라운드에서 계속 실행된다.
2. **취소 전파 안됨**: `handle()`을 실행하는 스레드가 인터럽트 되어도 하위 작업(`findUser`, `fetchOrder`)에는 전파되지 않는다.
3. **불필요한 대기**: `fetchOrder()`가 먼저 실패해도, `findUser()`가 끝날 때까지 `user.get()`에서 계속 블로킹된다. 실패를 더 빨리 알 수 있음에도 놓친다.
4. **가독성/관찰성 저하**: 스레드 덤프를 봐도 `findUser`, `fetchOrder`가 `handle()`의 하위 작업이라는 관계가 드러나지 않는다.

근본 원인은 `ExecutorService`/`Future`가 **부모-자식 관계를 전혀 강제하지 않는 "구조화되지 않은(unstructured)" 동시성**이기 때문이다.

### 단일 스레드 코드는 원래 "구조화"되어 있다

```java
Response handle() throws IOException {
    String theUser = findUser();   // 순차 실행
    int theOrder = fetchOrder();
    return new Response(theUser, theOrder);
}
```

이 코드는 호출 스택(call stack) 구조 덕분에 `findUser()`가 실패하면 자동으로 `handle()`도 실패하고, `fetchOrder()`는 아예 시작되지 않는다. **Structured Concurrency**는 이런 "블록 구조 = 생명주기"라는 원칙을 멀티스레드 코드에도 그대로 적용하려는 시도다.

## 기본 사용법 (초급)

핵심 클래스는 `java.util.concurrent.StructuredTaskScope` 이다. 사용 흐름은 다음과 같다.

1. `StructuredTaskScope.open(...)`으로 스코프를 연다 (연 스레드 = **owner**).
2. `scope.fork(...)`로 하위 작업(subtask)을 등록한다. 각 하위 작업은 기본적으로 **가상 스레드(Virtual Thread)** 로 실행된다.
3. `scope.join()`으로 모든 하위 작업이 끝날 때까지 기다린다.
4. 결과를 처리한다.
5. `try-with-resources`로 스코프를 닫는다 (`close()`).

```java
Response handle() throws InterruptedException {
    try (var scope = StructuredTaskScope.open()) {

        Subtask<String>  user  = scope.fork(() -> findUser());
        Subtask<Integer> order = scope.fork(() -> fetchOrder());

        scope.join();   // 두 하위 작업을 모두 기다리고, 실패하면 예외를 던짐

        // 여기 도달했다면 둘 다 성공한 것
        return new Response(user.get(), order.get());
    }
}
```

- `StructuredTaskScope.open()` (파라미터 없는 버전)은 **"하나라도 실패하면 나머지를 취소하고 실패로 처리"** 하는 기본 정책을 사용한다.
- `Subtask<T>.get()`은 `join()` 이후에만 호출해야 한다 (그 전에 호출하면 예외 발생).

### `open()`이 제공하는 3가지 자동 보장

| 특징 | 설명 |
|---|---|
| Fail-Fast(단락 처리) | 하위 작업 중 하나라도 실패하면, 나머지 하위 작업들은 자동으로 인터럽트(취소)된다 |
| 취소 전파 | owner 스레드가 `join()` 도중 인터럽트되면, 스코프가 닫힐 때 모든 하위 작업이 취소된다 |
| 명확한 생명주기 | 모든 하위 작업의 스레드는 `try` 블록을 벗어나기 전에 반드시 종료된다 (스레드 누수 없음) |

## Joiner: 완료 정책 선택하기 (중급)

`open()`에 파라미터로 `Joiner`를 넘기면 다양한 "완료 정책(completion policy)"을 선택할 수 있다. `Joiner`는 하위 작업의 완료를 어떻게 처리하고 `join()`의 반환값을 무엇으로 만들지 결정한다.

### 1. 기본 정책: 실패 시 즉시 취소 (앞선 예제와 동일)

```java
try (var scope = StructuredTaskScope.open()) {  // 내부적으로 Joiner.<T>awaitAllSuccessfulOrThrow() 와 유사
    ...
}
```

### 2. `Joiner.allSuccessfulOrThrow()`: 모두 성공해야 하고, 스트림으로 결과 받기

```java
<T> List<T> runConcurrently(Collection<Callable<T>> tasks) throws InterruptedException {
    try (var scope = StructuredTaskScope.open(Joiner.<T>allSuccessfulOrThrow())) {
        tasks.forEach(scope::fork);
        return scope.join().map(Subtask::get).toList();  // join()이 Stream<Subtask<T>>를 반환
    }
}
```

- 하나라도 실패하면 `join()`이 `StructuredTaskScope.FailedException`을 던진다 (원인은 실패한 하위 작업의 예외).

### 3. `Joiner.anySuccessfulResultOrThrow()`: 가장 먼저 성공한 결과만 필요 (Race 패턴)

```java
<T> T race(Collection<Callable<T>> tasks) throws InterruptedException {
    try (var scope = StructuredTaskScope.open(Joiner.<T>anySuccessfulResultOrThrow())) {
        tasks.forEach(scope::fork);
        return scope.join();  // 가장 먼저 성공한 결과 하나를 반환
    }
}
```

- 하나라도 성공하는 즉시 나머지는 취소하고 그 결과를 반환한다. (예: 동일 데이터를 제공하는 이중화된 서버 중 가장 빨리 응답하는 것을 사용)
- 모두 실패하면 `FailedException`을 던진다.

### 4. 그 외 내장 Joiner

| Joiner | 동작 |
|---|---|
| `awaitAll()` | 성공/실패 여부와 무관하게 **모든** 하위 작업이 끝날 때까지 대기 (취소 없음) |
| `awaitAllSuccessfulOrThrow()` | 모두 성공할 때까지 대기, 실패 시 나머지 취소 후 예외 |
| `allSuccessfulOrThrow()` | 위와 동일한 정책이지만 결과를 `Stream<Subtask<T>>`로 반환 |
| `anySuccessfulResultOrThrow()` | 하나만 성공하면 즉시 그 결과 반환 (Race) |
| `allUntil(Predicate<Subtask<T>> isDone)` | 모두 성공하거나, Predicate가 `true`를 반환하는 하위 작업이 나오면 취소하고 스트림 반환 |

⚠️ **주의**: `Joiner` 인스턴스는 **스코프 하나당 한 번만** 사용해야 한다. 여러 스코프에서 재사용하거나, 스코프가 닫힌 뒤 재사용하면 안 된다.

## 커스텀 Joiner 만들기 (고급)

`Joiner<T, R>` 인터페이스를 직접 구현하면 원하는 완료 정책을 만들 수 있다.

```java
public interface Joiner<T, R> {
    default boolean onFork(Subtask<? extends T> subtask);     // fork 시점에 호출
    default boolean onComplete(Subtask<? extends T> subtask);  // 하위 작업 완료 시점에 호출
    R result() throws Throwable;                                // join()의 최종 결과 생성
}
```

- `onFork`/`onComplete`가 `true`를 반환하면 **스코프를 즉시 취소**한다 (나머지 하위 작업 인터럽트).
- `onComplete`는 여러 하위 작업 스레드에서 **동시에 호출될 수 있으므로 반드시 thread-safe** 해야 한다.

예제: 실패한 것은 무시하고 성공한 결과만 모으는 Joiner

```java
class CollectingJoiner<T> implements Joiner<T, Stream<T>> {
    private final Queue<T> results = new ConcurrentLinkedQueue<>();

    @Override
    public boolean onComplete(Subtask<? extends T> subtask) {
        if (subtask.state() == Subtask.State.SUCCESS) {
            results.add(subtask.get());
        }
        return false; // 스코프를 취소하지 않고 계속 진행
    }

    @Override
    public Stream<T> result() {
        return results.stream();
    }
}

<T> List<T> allSuccessful(List<Callable<T>> tasks) throws InterruptedException {
    try (var scope = StructuredTaskScope.open(new CollectingJoiner<T>())) {
        tasks.forEach(scope::fork);
        return scope.join().toList();
    }
}
```

## `Subtask`의 상태 (`Subtask.State`)

`fork()`가 반환하는 `Subtask<T>` 객체는 하위 작업의 상태를 나타낸다.

| 상태 | 의미 |
|---|---|
| `UNAVAILABLE` | 아직 완료되지 않음 (실행 중이거나 대기 중) |
| `SUCCESS` | 정상적으로 완료됨 → `get()`으로 결과 조회 가능 |
| `FAILED` | 예외를 던지고 종료됨 → `exception()`으로 원인 조회 가능 |

```java
Subtask<String> s = scope.fork(() -> doWork());
scope.join();

switch (s.state()) {
    case SUCCESS -> System.out.println(s.get());
    case FAILED  -> System.out.println(s.exception());
    case UNAVAILABLE -> System.out.println("join() 전에는 도달하지 않음");
}
```

## 타임아웃과 스레드 팩토리 설정 (고급)

`open(Joiner, Function<Configuration, Configuration>)` 형태의 3번째 팩토리 메소드로 이름, 타임아웃, 스레드 팩토리를 지정할 수 있다.

```java
<T> List<T> runConcurrently(Collection<Callable<T>> tasks,
                             ThreadFactory factory,
                             Duration timeout) throws InterruptedException {

    try (var scope = StructuredTaskScope.open(
            Joiner.<T>allSuccessfulOrThrow(),
            cf -> cf.withThreadFactory(factory)
                    .withTimeout(timeout))) {

        tasks.forEach(scope::fork);
        return scope.join().map(Subtask::get).toList();
    }
}
```

- `withTimeout(Duration)`: 지정 시간이 지나면 스코프를 취소하고, `join()`이 `StructuredTaskScope.TimeoutException`을 던진다.
- `withThreadFactory(ThreadFactory)`: 하위 작업을 실행할 스레드를 커스터마이징 (예: 스레드 이름 접두어 지정).
- `withName(String)`: 관찰/모니터링(스레드 덤프 등)을 위한 스코프 이름 지정.

## 취소(Cancellation) 동작 원리 (고급)

1. owner 스레드가 `join()` 호출 전/도중에 인터럽트될 수 있다 (예: owner 자신이 상위 스코프의 하위 작업으로서 취소당한 경우).
2. 이 경우 `join()`은 예외를 던지고, `try-with-resources`가 스코프를 취소한다.
3. 스코프가 취소되면 아직 끝나지 않은 모든 하위 작업이 인터럽트되고, `close()`는 그 작업들이 실제로 종료될 때까지 **대기**한다.
4. 따라서 **하위 작업은 인터럽트에 신속히 반응하도록 작성**해야 한다. 인터럽트에 응답하지 않는(예: 인터럽트 불가능한 블로킹 호출을 사용하는) 하위 작업은 스코프의 `close()`를 무한정 지연시킬 수 있다.

```java
// 잘못된 예: 인터럽트에 응답하지 않아 스코프 종료를 지연시킴
scope.fork(() -> {
    someNonInterruptibleBlockingCall(); // 취소되어도 즉시 끝나지 않음
    return result;
});
```

## Structured 사용 강제 (Structural Enforcement)

런타임이 다음과 같은 규칙을 강제하며, 위반 시 `StructureViolationException`을 던진다.

- **owner 스레드만** `fork()`/`join()`을 호출할 수 있다. 다른 스레드에서 호출하면 예외.
- 스코프는 열었던 순서의 **역순으로 닫혀야** 한다 (중첩된 스코프의 올바른 nesting 유지).
- `try-with-resources` 없이 사용하거나, `close()` 호출 없이 메소드를 빠져나가면 문제가 발생할 수 있다. → **항상 `try-with-resources`로 사용**할 것.
- `StructuredTaskScope`는 `ExecutorService`/`Executor` 인터페이스를 구현하지 않는다. 기존 코드의 "구조화되지 않은" 사용 패턴(예: 다른 스레드에 넘겨 재사용)과 섞이는 것을 막기 위한 의도적 설계다.

## 예외 처리 패턴 (중급~고급)

`join()`이 실패를 나타내면 `StructuredTaskScope.FailedException`을 던진다. 원인은 `getCause()`로 조회한다.

```java
try (var scope = StructuredTaskScope.open()) {
    scope.fork(() -> findUser());
    scope.fork(() -> fetchOrder());
    scope.join();
} catch (StructuredTaskScope.FailedException e) {
    Throwable cause = e.getCause();
    switch (cause) {
        case IOException ioe -> handleIoError(ioe);
        default -> handleUnknownError(cause);
    }
}
```

- 특정 예외에 대해 기본값으로 대체하고 싶다면, **하위 작업 내부에서 예외를 잡아 기본값을 반환**하는 편이 owner에서 예외를 처리하는 것보다 더 적절한 경우가 많다.

## 중첩(Nested) 스코프

하위 작업 내부에서 또 다른 `StructuredTaskScope`를 열어 자신만의 하위 작업을 fork할 수 있다. 이렇게 하면 스코프의 계층 구조(tree)가 만들어지고, 이 구조는 코드의 블록 구조(중첩)와 그대로 일치한다. 상위 스코프가 취소되면 하위 스코프 트리 전체가 취소된다.

## `ScopedValue`와의 연동

스코프 안에서 fork된 하위 작업은 owner의 `ScopedValue` 바인딩을 그대로 상속받는다. 즉, owner가 특정 `ScopedValue`에서 읽은 값을, 모든 하위 작업도 동일하게 읽게 된다. (`ThreadLocal`과 달리 불변이며 명시적으로 범위가 한정된 값 전달 방식)

## 관찰성(Observability) — 스레드 덤프

가상 스레드용 JSON 스레드 덤프 포맷이 `StructuredTaskScope` 계층까지 함께 보여주도록 확장되었다.

```bash
jcmd <pid> Thread.dump_to_file -format=json <file>
```

- 각 스코프의 JSON 객체는 그 안에서 fork된 스레드들의 스택 트레이스 배열과, 자신의 부모 스코프에 대한 참조를 포함한다.
- owner 스레드는 보통 `join()`에서 블로킹되어 있으므로, 덤프만 봐도 "이 작업이 지금 어떤 하위 작업들을 기다리고 있는지" 파악할 수 있다.

## 버전별 API 변화 요약

| JDK | JEP | 상태 | 주요 API |
|---|---|---|---|
| 19 | 428 | Incubator | `jdk.incubator.concurrent.StructuredTaskScope` |
| 20 | 437 | Incubator (재도입) | 동일 |
| 21 | 453 | Preview (1차) | `new StructuredTaskScope<>()` 생성자, `fork()`가 `Future` 반환, `ShutdownOnFailure`/`ShutdownOnSuccess` |
| 22 | 462 | Preview (2차) | `fork()`가 `Subtask` 반환으로 변경 |
| 23 | 480 | Preview (3차) | 세부 조정 |
| 24 | 499 | Preview (4차→표기상 3차 재표기) | `ShutdownOnFailure`/`ShutdownOnSuccess` 유지 |
| 25 | 505 | Preview (최신) | 생성자 → `open()` 정적 팩토리, `Joiner` 인터페이스 도입 (본 문서 기준) |

- JDK 21~24에서는 아래처럼 `ShutdownOnFailure`/`ShutdownOnSuccess`를 사용했다 (참고용, JDK 25에서는 deprecated 방향).

```java
// JDK 21~24 스타일 (참고)
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> user  = scope.fork(() -> findUser());
    Future<Integer> order = scope.fork(() -> fetchOrder());

    scope.join();
    scope.throwIfFailed();  // 실패 시 예외를 던짐

    return new Response(user.resultNow(), order.resultNow());
}
```

- **`Future` 대신 `Subtask`를 쓰는 이유**: `Future.get()`은 블로킹 메소드라 구조화된 흐름과 맞지 않는다. `Subtask.get()`은 `join()` 이후에만 호출하는 것을 전제로 하며, 실제로는 이전 `Future.resultNow()`(논블로킹, 이미 완료된 결과만 즉시 반환)와 동일하게 동작한다.

## 정리: 언제 사용해야 하는가

- 여러 개의 독립적인 I/O 작업(예: 여러 API 호출, DB 조회)을 **동시에** 실행하고, 결과를 **한 곳에서 취합**해야 할 때.
- 하나라도 실패하면 나머지를 즉시 취소하고 싶을 때 (Fail-Fast).
- 여러 개의 중복(redundant) 소스 중 가장 빨리 응답하는 것만 필요할 때 (Race).
- `ExecutorService` + `Future` 조합으로 직접 취소/타임아웃/예외 전파를 관리하던 코드를 더 안전하고 명확하게 바꾸고 싶을 때.
- **주의**: 아직 Preview 단계이므로 프로덕션에 바로 적용하기보다는, 신규 프로젝트에서 실험적으로 도입하거나 향후 정식화에 대비해 학습하는 용도로 우선 활용하는 것을 권장한다.
