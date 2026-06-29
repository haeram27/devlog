# CompletableDeferred

- CompletableDeferred는 코루틴 버전의 Future 또는 Promise라고 이다.
- 특정 시점에 외부에서 성공 결과나 실패(예외)를 수동으로 주입할 수 있는 일종의 '결과 대기 박스'입니다.

- CompletableDeferred 멤버 메소드
  - complete(value): 결과를 박스에 넣고 대기 중인 await()들을 깨웁니다. 한 번 완료(complete)되면 상태가 고정됩니다. 여러 번 호출해도 첫 번째 결과만 유효합니다.
  - completeWithException(ex): 에러를 던져서 대기 중인 await()들에서 예외가 발생하게 합니다.

## 콜백 기반 API를 코루틴으로 변환할 때

가장 많이 쓰이는 패턴입니다. 기존의 리스너(Listener)나 콜백 구조를 suspend 함수로 래핑할 때 유용합니다.

```kotlin
suspend fun fetchDataFromOldApi(): String {
    val deferred = CompletableDeferred<String>()

    // 기존의 콜백 기반 서비스 호출
    legacyService.request(object : Callback {
        override fun onSuccess(result: String) {
            // 외부에서 결과를 수동으로 주입 (await를 깨움)
            deferred.complete(result)
        }

        override fun onError(e: Exception) {
            // 외부에서 예외를 주입
            deferred.completeWithException(e)
        }
    })

    return deferred.await() // 결과가 올 때까지 이 코루틴만 '중단'됨
}
```

## 특정 이벤트 신호 대기 (CountDownLatch와 가장 유사)

어떤 복잡한 초기화가 완료될 때까지 다른 코루틴들이 기다려야 하는 상황에서 유용합니다.

```kotlin
class SystemManager {
    private val isInitialized = CompletableDeferred<Unit>()

    // 초기화 작업
    fun init() {
        // ... 복잡한 로직 ...
        isInitialized.complete(Unit) // 신호 발송!
    }

    // 다른 곳에서 호출할 대기 함수
    suspend fun waitForReady() {
        isInitialized.await() // 초기화 전까지 여기서 멈춤
    }
}
```

## 생산자-소비자 간의 단발성 결과 전달

한 코루틴이 데이터를 만들고, 다른 코루틴이 그 결과 딱 하나를 간절히 기다릴 때 사용합니다.

```kotlin
fun main() = runBlocking {
    val deferred = CompletableDeferred<Int>()

    launch {
        delay(1000)
        println("1. 계산 완료!")
        deferred.complete(42) // 결과 전달
    }

    launch {
        println("2. 결과 기다리는 중...")
        val result = deferred.await() // 1초 뒤 여기서 값을 받음
        println("3. 받은 결과: $result")
    }

    unit // runBlocking 종료 방지
}
```

CompletableDeferred는 결과값을 딱 한 번 주고받을 때 사용합니다. 만약 데이터가 여러 번 연속해서 흘러나오는 구조라면 Channel이나 SharedFlow를 사용하는 것이 더 적절합니다.
지금 고민 중인 상황이 단발성 응답인가요, 아니면 지속적인 이벤트인가요? 그에 맞춰 더 적합한 도구를 골라드릴 수 있습니다.

## 다수의 코루틴 실행 후 결과 취합

- java의 `CountDownLatch.await()` 사용과 같음

```kotlin
import kotlinx.coroutines.*

fun main() = runBlocking {
    val count = 10
    // 1. 10개의 결과 대기 박스(Deferred) 생성
    val deferreds = List(count) { CompletableDeferred<String>() }

    // 2. 10개의 코루틴 동시 실행
    deferreds.forEachIndexed { index, deferred ->
        launch(Dispatchers.Default) {
            try {
                // 실제 비동기 작업 시뮬레이션 (예: API 호출)
                doWorkWithCallback(index) { result ->
                    deferred.complete(result) // 완료 신호 주입
                }
            } catch (e: Exception) {
                deferred.completeWithException(e)
            }
        }
    }

    println("10개의 작업 완료를 최대 10초간 대기합니다...")

    try {
        // 3. withTimeout을 사용하여 10초 제한 설정
        withTimeout(10_000L) {
            // 4. 모든 Deferred가 완료될 때까지 비차단(Non-blocking) 대기
            val results = deferreds.awaitAll()
            println("모든 작업 완료 성공: $results")
        }
    } catch (e: TimeoutCancellationException) {
        println("시간 초과! 10초 내에 모든 작업이 완료되지 않았습니다.")
        // 필요 시 여기서 진행 중인 작업들을 취소하는 로직 추가
    }
}

// 콜백 기반의 가상 함수
fun doWorkWithCallback(id: Int, callback: (String) -> Unit) {
    // 실제로는 네트워크 전송 등이 일어남
    Thread.sleep((500..12000).random().toLong()) // 랜덤한 작업 시간
    callback("Work-$id Result")
}
```
