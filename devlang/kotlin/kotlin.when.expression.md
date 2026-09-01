# Kotlin `when`: statement vs expression

`when`은 Kotlin에서 문(statement)으로도, 식(expression)으로도 쓸 수 있다. 겉모습은 비슷해 보여도 컴파일러가 이 둘을 다르게 취급하며, 특히 **exhaustiveness(누락 없음) 검사 여부**가 갈린다.

## 1. 판단 기준: "결과값을 실제로 소비하는가"

문/식을 가르는 기준은 `=` 뒤에 오느냐가 아니라, **`when`의 결과값이 어딘가에서 실제로 사용되는가**다. `= when(...)`은 그 소비 방법 중 가장 흔한 형태일 뿐이고, 아래 형태들도 전부 동일하게 expression으로 취급된다.

### expression으로 취급되는 경우 (결과값을 소비함)

```kotlin
// 1) 함수의 식 본문 (= 뒤에 옴)
fun f(x: Filter): FilterNode = when (x) { ... }

// 2) 변수에 대입
val node = when (x) { ... }

// 3) 블록 본문 함수에서 return 뒤에 직접 사용
fun f(x: Filter): FilterNode {
    return when (x) { ... }   // 이것도 expression 취급 → exhaustive 검사 받음
}

// 4) 다른 함수의 인자로 전달
someFunction(when (x) { ... })
```

### statement로 취급되는 경우 (결과값을 버림)

```kotlin
fun f(x: Filter): FilterNode {
    val builder = FilterNode.newBuilder()
    when (x) {                  // 결과값이 아무데도 쓰이지 않음 → statement
        is AndFilter -> builder.setAnd(...)
        is SearchStringFilter -> builder.setSearchString(...)
    }
    return builder.build()      // 실제 반환은 이 줄이 담당
}
```

> **주의**: Kotlin은 함수 본문이 `{ }` 블록이면, 마지막 줄이라도 `return`을 명시하지 않으면 자동으로 반환값이 되지 않는다(Rust와 다른 점). 그래서 `when`이 블록의 마지막에 있어도 `return`이나 대입 없이 그냥 놓여 있으면 statement다.

## 2. 왜 이 구분이 중요한가: exhaustiveness

Kotlin은 **`when`이 expression으로 쓰일 때만** sealed class/interface나 enum에 대한 exhaustiveness를 강제한다. statement로 쓰이면 분기를 빠뜨려도 컴파일이 그냥 통과한다.

```kotlin
sealed interface Filter
data class AndFilter(val targets: List<Filter>) : Filter
data class SearchStringFilter(val value: String) : Filter
data class StatusFilter(val value: List<String>) : Filter
```

**statement로 쓴 경우 — 분기 누락이 조용히 통과됨:**

```kotlin
fun log(filter: Filter) {
    when (filter) {                    // statement
        is AndFilter -> println("and")
        is SearchStringFilter -> println("search")
        // StatusFilter 분기 누락!
    }
    // 컴파일 에러 없음. StatusFilter가 들어오면 그냥 아무 것도 안 하고 지나감.
}
```

**expression으로 쓴 경우 — 분기 누락이 컴파일 에러로 잡힘:**

```kotlin
fun label(filter: Filter): String = when (filter) {   // expression
    is AndFilter -> "and"
    is SearchStringFilter -> "search"
    // StatusFilter 분기 누락!
}
// 컴파일 에러: "'when' expression must be exhaustive, add necessary 'is StatusFilter' branch"
```

`else` 분기를 추가하면 어떤 sealed 타입에 대해서도 컴파일은 통과하지만, 그러면 **새 하위 타입이 추가됐을 때 컴파일러가 알려주는 안전망이 사라진다** — `else`가 새 타입도 조용히 흡수해버리기 때문이다. 그래서 exhaustiveness의 이점을 제대로 누리려면 `else` 없이 모든 하위 타입을 명시적으로 나열해야 한다.

## 3. 실전 예시: DTO → gRPC 매핑

`sealed interface Filter`를 proto 메시지로 변환하는 매퍼 함수를 짤 때, `when`을 expression으로 쓰면 "새 필터 타입을 추가했는데 매핑 코드를 깜빡했다"는 실수를 컴파일 타임에 잡아낼 수 있다.

```kotlin
private fun toFilterProto(filter: Filter): FilterNode = when (filter) {
    is AndFilter -> FilterNode.newBuilder()
        .setAnd(
            AndFilterProto.newBuilder()
                .addAllTargets(filter.targets.map { toFilterProto(it) })
        )
        .build()

    is SearchStringFilter -> FilterNode.newBuilder()
        .setSearchString(
            SearchStringFilterProto.newBuilder().setValue(filter.value)
        )
        .build()

    is StatusFilter -> FilterNode.newBuilder()
        .setStatus(
            StatusFilterProto.newBuilder().addAllValues(filter.value.map { it.name })
        )
        .build()
}
// sealed interface + when-식이라 else 없이도 exhaustive
// → 새 서브타입 추가 시 컴파일 에러로 강제됨
```

전체 맥락(REST DTO → gRPC 매핑 설계)은 [`jackson.rest.to.grpc.md`](../java/jackson/jackson.rest.to.grpc.md) 참고.

## 4. 요약

| | `when` statement | `when` expression |
|---|---|---|
| 판단 기준 | 결과값을 안 씀 | 결과값을 대입/반환/전달 등으로 씀 |
| `else` 없이 분기 누락 시 | 컴파일 정상 통과 (조용히 무시) | 컴파일 에러 (sealed class/enum인 경우) |
| 용도 | 부수효과(로깅, 상태 변경 등) | 값 생성, 특히 sealed 타입의 안전한 분기 처리 |
| 관련 Java 비교 | [`java.switch.expression.md`](../java/java.switch.expression.md) | |
