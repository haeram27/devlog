# kotlin coroutine terms

## 코루틴이란

- 어플리케이션 레벨에서 제공하는 실행흐름
- 코드의 특정 영역에서 Thread를 attach/detach 가능하도록 하여 코드의 실행 흐름을 제어, 
- 코루틴이 suspend(thread detach)될 때 코드 위치가 따로 저장되며 추후 해당 지점에서 resume 됨(thread attach) 하는 방식
- 코루틴을 사용할수 있는 영역을 coroutineScope라고 하며, 이 영역은 runBlocking, launch, async 키워드를 이용하여 생성 가능
- Thread는 내부 쓰레드 풀을 이용(Dispatcher.MAIN/IO/DEFAULT)
- 코틀린에서 동시성 제어를 위한 Collection은 별도로 없으며, 자바의 concurrent collection이나 atomic 타입을 이용해야 함

## 1. 코루틴 빌더 (Coroutine Builders)

코루틴을 생성하고 시작하는 함수들입니다.

- launch: 결과값 없이 실행만 하는 'Fire-and-forget' 빌더. Job을 반환합니다. 현재 스레드 Blocking 안함.
- async: 결과값이 필요한 비동기 작업 빌더. `Deferred<T>`를 반환하며 `Deferred.await()`로 결과를 받습니다. 현재 스레드 Blocking 안함.
- runBlocking: 현재 스레드를 차단(Blocking)하며 코루틴이 끝날 때까지 대기합니다. (주로 테스트나 메인 함수에서만 사용)
- produce: 데이터를 생산하고 ReceiveChannel을 반환하는 빌더입니다.

## 2. 제어 및 대기 (Control & Await)

실행 중인 코루틴의 상태를 관리하거나 결과를 기다립니다.
다음의 키워드 들은 coroutine 영역(lauch, async, runBlocking에 의해 생성된 Coroutine Scope)에서만 사용할 수 있다.
이 키워드들의 사용된 곳에서 위로 거슬러 올라가면 반드시 코루틴 빌더들을 만나게 된다.

- suspend: 함수가 실행중 suspend 가능한 CoroutineScope 임을 명시, suspend란 실행중인 Thread를 현재 코루틴에서 detach하여 현재 코루틴은 잠시 실행이 멈추고, Thread는 다른 코루틴을 실행하는 것
- await: async의 결과를 비차단(Non-blocking) 방식으로 기다립니다. `Deferred`, `CompletableDeferred`의 멤버 함수
- join: launch로 시작된 작업이 완료될 때까지 기다립니다.
- cancel: 실행 중인 코루틴을 중단시킵니다.
- delay: 스레드를 차단하지 않고 정해진 시간 동안 코루틴을 잠재웁니다.

## 3. 핵심 구성 요소 (Core Elements)

코루틴이 어디서, 어떻게 실행될지 결정하는 요소들입니다.

- CoroutineScope: 코루틴의 '범위'. 범위가 종료되면 그 안의 코루틴도 모두 종료됩니다 (예: GlobalScope, lifecycleScope).
- CoroutineContext: 코루틴의 설정값들의 집합 (이름, 디스패처, Job 등).
- Dispatcher: 코루틴을 어떤 스레드 풀에서 실행할지 결정합니다 (Main, IO, Default).
- Job: 코루틴의 생명주기를 담은 핸들러입니다. 부모-자식 관계를 형성합니다.

## 4. 구조화된 동시성 (Structured Concurrency)

코루틴 간의 계층 구조와 스코프 제어를 돕습니다.

- coroutineScope: 자식들이 모두 성공해야 완료되는 범위 생성. 하나라도 실패하면 모두 취소됩니다.
- supervisorScope: 자식의 실패가 부모나 형제에게 전파되지 않는 독립적인 범위 생성.
- withContext: 코루틴의 컨텍스트(주로 디스패처)를 즉시 전환하여 실행합니다.

## 5. 예외 처리 (Exception Handling)

코루틴 내부의 에러를 관리합니다.

- CoroutineExceptionHandler: launch 블록에서 발생하는 예외를 전역적으로 처리하는 핸들러.
- NonCancellable: 코루틴이 취소되는 중에도 반드시 실행되어야 하는 정리 작업(finally 등)에 사용되는 컨텍스트.

## 6. 데이터 스트림 (Asynchronous Streams)

연속적인 데이터 흐름을 처리합니다.

- Flow: 코루틴의 리액티브 스트림. emit으로 데이터를 보내고 collect로 받습니다.
- Channel: 코루틴 간의 통신을 위한 파이프라인 (Queue와 유사).

## Dispatcher

Kotlin 코루틴 라이브러리는 개발자가 별도로 스레드를 생성하거나 관리하는 번거로움을 덜어주기 위해 사전 정의된 공유 스레드 풀(Shared Thread Pool)
각 디스패처의 내부적인 특징과 용도는 다음과 같습니다.

### 1. Dispatchers.Default (CPU 집약적 작업용)

- 스레드 풀: JVM의 공용 ForkJoinPool을 기본으로 사용합니다.
- 스레드 개수: 기본적으로 해당 기기의 CPU 코어 수만큼 생성됩니다 (최소 2개).
- 용도: 복잡한 계산, JSON 파싱, 리스트 정렬, 알고리즘 연산 등 CPU를 많이 쓰는 작업에 최적화되어 있습니다.

### 2. Dispatchers.IO (입출력 작업용)

- 스레드 풀: Default 디스패처와 스레드 자원을 공유하지만, 동시 생성 가능한 스레드 한도가 훨씬 높습니다.
- 스레드 개수: 기본적으로 64개 또는 코어 수 중 더 큰 값으로 설정됩니다. 필요에 따라 동적으로 스레드를 늘립니다.
- 용도: 네트워크 API 호출, 파일 읽기/쓰기, 데이터베이스(DB) 쿼리 등 기다리는 시간(Wait time)이 많은 작업에 적합합니다.
- 참고: 스레드가 대기(Blocking) 상태에 빠져도 다른 스레드를 더 만들어 작업을 이어가도록 설계되어 있습니다.

- 참고: jvm 21 이상의 환경이라면 IO 작업에 Dispatcher.IO를 사용하는 것 보다 Java의 virtual thread pool을 사용하는 것이 좋습니다.
  - 다음의 코드로 virtual thread를 사용하는 coroutine dispatcher 생성 가능
  - `Executors.newVirtualThreadPerTaskExecutor().asCoroutineDispatcher()`

### 3. Dispatchers.Main (UI 작업용)

- 스레드 풀: 별도의 풀이 아니라, 플랫폼(Android, Swing 등)의 Main(UI) Thread 하나를 가리킵니다.
- 특징: 단일 스레드이며, 화면을 그리거나 사용자 입력에 즉각 반응해야 하는 작업에 쓰입니다.
- 주의: 서버 사이드(Ktor, Spring) 환경에서는 별도의 설정 없이는 존재하지 않거나 사용되지 않는 경우가 많습니다.

### 4. Dispatchers.Unconfined (특수 목적)

- 특징: 특정 스레드 풀에 고정되지 않습니다. 첫 suspend 지점 전까지는 현재 호출한 스레드에서 실행되다가, resume 될 때는 중단시킨 스레드에서 그대로 실행됩니다.
- 용도: 특정 상황의 테스트 코드 외에는 일반적인 개발 환경에서 권장되지 않습니다.

### Dispatcher.IO에 Java의 virtual thread 사용

Dispatchers.IO는 Java의 Virtual Thread를 사용하는 것이 좋음

- 제한 없는 확장성: 기존 Dispatchers.IO는 기본 64개(최대 설정값 존재)의 OS 스레드를 사용하지만, 이 방식은 수천~수만 개의 요청이 들어와도 각 요청마다 가벼운 Virtual Thread를 할당하므로 스레드 부족 현상이 거의 발생하지 않습니다.
- 블로킹 안전성: 내부에서 Thread.sleep()이나 JDBC 같은 블로킹 API를 호출하더라도 OS 스레드를 점유하지 않고 Virtual Thread만 일시 정지되므로 서버 전체 성능 저하를 막을 수 있습니다.

Java의 Executors를 사용하여 Virtual Thread 전용 Executor를 만든 뒤, 이를 Kotlin 코루틴의 asCoroutineDispatcher() 확장 함수를 통해 디스패처로 변환

#### 1. 기본 설정 코드

```kotlin
import kotlinx.coroutines.*
import java.util.concurrent.Executors

// 1. Virtual Thread 기반의 Executor 생성
// 2. .asCoroutineDispatcher()를 사용하여 Executor를 코루틴 디스패처로 변환
val VirtualThreadDispatcher = Executors.newVirtualThreadPerTaskExecutor().asCoroutineDispatcher()

fun main() = runBlocking {
    launch(VirtualThreadDispatcher) {
        println("실행 중인 스레드: ${Thread.currentThread()}")
        // 결과 예: VirtualThread[#30,forkjoinpool-1-worker-1]/runnable@...

        delay(1000)
        println("중단 후 다시 실행되는 스레드: ${Thread.currentThread()}")
    }
}
```

#### 2. Spring Boot에서 전역 설정 (추천)

서버 전체에서 Dispatchers.IO 대신 Virtual Thread를 기본으로 사용하고 싶다면, 별도의 오브젝트나 설정 클래스에 정의해두고 사용

```kotlin
object AppDispatchers {
    // 모든 IO 작업을 이 디스패처에서 실행하도록 관리
    val IO = Executors.newVirtualThreadPerTaskExecutor().asCoroutineDispatcher()
}

// 사용 예시
suspend fun saveUser() = withContext(AppDispatchers.IO) {
    // JDBC나 File IO 등 블로킹 작업 수행 시 Virtual Thread가 효율적으로 처리
    userRepository.save(user)
}
```
