# Java Scoped Value

- [Java Scoped Value](#java-scoped-value)
  - [개요](#개요)
  - [핵심 개념](#핵심-개념)
    - [ThreadLocal과의 비교](#threadlocal과의-비교)
  - [사용 방법](#사용-방법)
    - [기본 API](#기본-api)
    - [일반 메서드에서 값 전달](#일반-메서드에서-값-전달)
    - [Rebinding (중첩 바인딩)](#rebinding-중첩-바인딩)
    - [가상 스레드(virtual thread)와 함께 사용](#가상-스레드virtual-thread와-함께-사용)
  - [주의사항 / 트러블슈팅](#주의사항--트러블슈팅)
  - [참고 자료](#참고-자료)

## 개요

`ScopedValue`는 메서드 파라미터를 계속 전달하지 않고도, 현재 실행 범위(scope)에서만 읽을 수 있는 읽기 전용 key-value 컨텍스트를 제공하는 API다.

- `scope`란: `ScopedValue` 값이 유효한 동적 실행 범위다.
- `실행 범위`란: `ScopedValue.where(key, value).run(runnable)` / `call(callable)`로 바인딩을 적용했을 때, runnable/callable의 시작부터 종료까지 이어지는 실행 흐름 구간(및 그 호출 흐름)이다.
- 언제 쓰는지: 요청 ID, 사용자 컨텍스트, 트레이싱 정보처럼 호출 체인 전반에 전달해야 하는 값을 안전하게 공유할 때 사용한다.
- ThreadLocal 관계: `ScopedValue`는 `ThreadLocal`의 한계를 줄이기 위해 도입된 **대체/보완용 별도 컨텍스트 모델**이다.
- 한계/주의:
  - 전역 mutable 상태 저장소가 아니다.
  - 값은 `where(...).run(...)`/`call(...)` 범위 밖에서 접근할 수 없다.
  - JDK 25 부터 정식 API에 포함되었다.

`ScopedValue`는 `ThreadLocalMap`에 값을 `set/remove`하는 방식이 아니라 scope 기반 바인딩으로 관리된다.
그래서 `ThreadLocal`처럼 임의 시점 변경/정리 모델이 아니라, `where(...).run/call` 범위 진입/종료에 따라 바인딩이 자동 적용·해제된다.

## 핵심 개념

- `ThreadLocal`과의 차이:
  - `ThreadLocal`은 `remove()` 호출 누락 시 메모리 누수 위험이 있다.
  - `ScopedValue`는 **명시적 범위 바인딩**으로 생명주기가 코드 블록에 고정되어 코드 블록 실행 완료시 자동 해제 되므로 별도의 메모리 누수 위험이 없다.
- 읽기 전용:
  - 한 번 바인딩된 값은 같은 scope 안에서 읽기만 가능하다.
  - 값 변경이 필요하면 새 scope로 **재바인딩(rebinding)** 해야 한다.
- 구조화 동시성과 궁합:
  - `StructuredTaskScope` + 가상 스레드 환경에서 상위 컨텍스트를 하위 작업에 전달하기 쉽다.

### ThreadLocal과의 비교

| 구분 | ThreadLocal | ScopedValue |
| --- | --- | --- |
| 주요 목적 | 스레드별 개별 상태(Mutable State) 저장 | 범위가 정해진 불변 컨텍스트(Context) 전달 |
| 수명 관리 | 명시적으로 remove()를 안 하면 계속 남음 | 스코프(범위) 종료 시 자동 해제·복원 |
| 값 변경 | set()으로 언제든지 값 수정 가능 | 수정 불가 (하위 범위에서 재바인딩만 가능) |
| 자식 스레드 | 상속형(Inheritable) 사용 시 복사 비용 발생 | 구조화된 범위 내에서 안전하게 전달 |

## 사용 방법

- 기본 흐름: `where()`로 key/value를 바인딩한 뒤, `run()` 또는 `call()`로 해당 바인딩이 적용된 코드 블럭을 수행한다.

### 기본 API

```java
import java.lang.ScopedValue;

public class Main {
    private static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();
}
```

- 주요 메서드:
  - `ScopedValue.where(key, value).run(runnable)`
  - `ScopedValue.where(key, value).call(callable)`
  - `ScopedValue.runWhere(key, value, runnable)`
  - `ScopedValue.callWhere(key, value, callable)`
  - `key.isBound()`, `key.get()`, `key.orElse(defaultValue)`

### 일반 메서드에서 값 전달

```java
import java.lang.ScopedValue;

public class Main {
    private static final ScopedValue<String> REQUEST_ID = ScopedValue.newInstance();

    public static void main(String[] args) {
        ScopedValue.where(REQUEST_ID, "req-1001").run(Main::service);
    }

    private static void service() {
        repository();
    }

    private static void repository() {
        // 파라미터 없이도 현재 scope의 값을 읽을 수 있음
        System.out.println("requestId=" + REQUEST_ID.get());
    }
}
```

### Rebinding (중첩 바인딩)

```java
import java.lang.ScopedValue;

public class Main {
    private static final ScopedValue<String> USER = ScopedValue.newInstance();

    public static void main(String[] args) {
        ScopedValue.where(USER, "user-A").run(() -> {
            System.out.println("outer=" + USER.get()); // user-A

            ScopedValue.where(USER, "user-B").run(() -> {
                System.out.println("inner=" + USER.get()); // user-B
            });

            System.out.println("outer-again=" + USER.get()); // user-A
        });
    }
}
```

- 내부 scope가 끝나면 바깥 scope 값이 다시 유효해진다.

### 가상 스레드(virtual thread)와 함께 사용

```java
import java.lang.ScopedValue;
import java.util.concurrent.StructuredTaskScope;

public class Main {
    private static final ScopedValue<String> TRACE_ID = ScopedValue.newInstance();

    public static void main(String[] args) throws Exception {
        ScopedValue.where(TRACE_ID, "trace-42").run(() -> {
            try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
                var t1 = scope.fork(() -> callA());
                var t2 = scope.fork(() -> callB());
                scope.join();
                scope.throwIfFailed();
                System.out.println(t1.resultNow() + ", " + t2.resultNow());
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        });
    }

    private static String callA() {
        return "A:" + TRACE_ID.orElse("none");
    }

    private static String callB() {
        return "B:" + TRACE_ID.orElse("none");
    }
}
```

- 부모 scope에서 바인딩한 컨텍스트 값을 하위 동시 작업에서 일관되게 사용할 수 있다.

## 주의사항 / 트러블슈팅

- `IllegalStateException: not bound` 발생:
  - 원인: 바인딩 범위 밖에서 `get()` 호출.
  - 대응: `isBound()` 또는 `orElse(...)`로 방어하고, 실제 실행 경로가 `where(...).run/call` 내부인지 확인.
- `ThreadLocal` 대체 시 주의:
  - mutable 객체를 값으로 넣으면 참조 자체는 공유된다. 필요하면 불변 객체를 사용한다.
  - 기존 `ThreadLocal.remove()` 패턴은 `ScopedValue`로 바꾸면 대부분 제거 가능하지만, 비동기 경계에서 실행 범위가 끊기지 않는지 점검해야 한다.
- 성능:
  - 값 조회 자체보다 컨텍스트 설계(범위의 크기, 바인딩 중첩 깊이, 불필요한 재바인딩 여부)가 더 큰 영향을 준다.

## 참고 자료

- https://dev-post.com/java-scoped-value/
- https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/ScopedValue.html
- https://openjdk.org/jeps/429
- https://openjdk.org/jeps/446