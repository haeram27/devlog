# Kotlin 코루틴 사용 패턴

## 1. 기본 실행 패턴 (Launch vs Async)

코루틴을 시작하는 가장 기본적인 두 가지 방법입니다. Kotlin 공식 문서에서는 작업의 결과값 필요 여부에 따라 선택할 것을 권장합니다.

- launch (Fire-and-forget): 결과값을 반환하지 않는 작업에 사용합니다. Job 객체를 반환하며 join()으로 완료를 대기할 수 있습니다.

```kotlin
val job = CoroutineScope(Dispatchers.Default).launch {
    // 백그라운드 작업
    delay(1000L)
    println("Task completed")
}
job.join() // 작업이 끝날 때까지 대기
```

- async (Deferred result): 결과값이 필요한 작업에 사용합니다. Deferred를 반환하며 await()를 통해 결과값을 받습니다.

```kotlin
val deferred = CoroutineScope(Dispatchers.Default).async {
    delay(1000L)
    "Result Data"
}
val result = deferred.await() // 결과를 받을 때까지 중단(suspend) 후 반환
```

```kotlin
suspend fun fetchAllData() = coroutineScope {
    // 1. 여러 비동기 작업 생성 (리스트 형태)
    val deferreds: List<Deferred<String>> = listOf(
        async { fetchFromServiceA() },
        async { fetchFromServiceB() },
        async { fetchFromServiceC() }
    )

    // 2. 모든 작업이 완료될 때까지 '중단(Suspend)' 후 결과 리스트 반환
    val results: List<String> = deferreds.awaitAll()

    println("모든 결과 수집 완료: $results")
}
```

- suspend: suspend가 붙은 함수는 실행 중에 스레드를 점유하지 않고 잠시 멈췄다가(Suspend=현재 코루틴으로 부터 Thread가 detach), 나중에 결과가 준비되면 멈췄던 지점부터 다시 실행(Resume=현재 코루틴에 Thread가 attach)될 수 있는 특별한 함수입니다.

## 2. 구조화된 동시성 (Structured Concurrency)

부모 코루틴이 자식 코루틴의 생명주기를 관리하는 패턴입니다. [Android Developers](https://developer.android.com/kotlin/coroutines/coroutines-adv?hl=ko) 가이드에 따르면, 이는 메모리 누수를 방지하고 예외 처리를 용이하게 합니다.

- coroutineScope: 자식 코루틴 중 하나라도 실패하면 전체가 취소됩니다.
- supervisorScope: 자식의 실패가 다른 자식이나 부모에게 영향을 주지 않습니다.

```kotlin
suspend fun fetchData() = supervisorScope {
    launch { /* 작업 1 (실패해도 2에 영향 없음) */ }
    launch { /* 작업 2 */ }
}
```

## 3. 디스패처 전환 (WithContext)

작업의 성격에 따라 실행 스레드 풀을 변경하는 패턴입니다. 안드로이드 UI 작업은 반드시 메인 스레드에서 수행해야 하므로 매우 중요합니다.

- Dispatchers.Main: UI 업데이트 및 가벼운 작업.
- Dispatchers.IO: 네트워크 요청, 디스크 읽기/쓰기 등 입출력 작업.
- Dispatchers.Default: CPU 집약적인 계산 작업.

```kotlin
withContext(Dispatchers.IO) {
    val data = apiService.getData() // 네트워크 작업
    withContext(Dispatchers.Main) {
        textView.text = data // UI 업데이트
    }
}
```

## 4. 연속 및 병렬 처리 패턴

비동기 작업들을 어떻게 조합하느냐에 따른 패턴입니다.

- 순차 실행 (Sequential): 앞선 작업의 결과가 다음 작업에 필요할 때 사용합니다.
- 병렬 실행 (Parallel): 여러 독립적인 작업을 동시에 수행하여 전체 시간을 단축합니다.

```kotlin
// 병렬 실행 예시
val res1 = async { task1() }
val res2 = async { task2() }
val combined = res1.await() + res2.await()
```

## 5. 예외 처리 패턴

코루틴 내부에서 발생하는 에러를 관리하는 방법입니다.

- try-catch: 일반적인 예외 처리 방식입니다.
- CoroutineExceptionHandler: launch로 시작된 코루틴의 미처리 예외를 전역적으로 캡처합니다.

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    println("Caught $exception")
}
scope.launch(handler) { throw Exception() }
```