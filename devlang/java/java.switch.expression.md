# Java `switch`: statement vs expression (그리고 Kotlin `when`과의 차이)

Java의 `switch`도 Kotlin의 `when`처럼 statement/expression 두 형태로 쓸 수 있다. 다만 exhaustiveness(누락 없음) 검사가 강제되는 조건이 Kotlin과 **정확히 대응하지 않는다** — 이 문서는 그 차이를 정리한다.

전체 배경은 [`kotlin.when.expression.md`](../kotlin/kotlin.when.expression.md) 참고.

## 1. 전통적 `switch` 문 (콜론 기반) — 언제나 exhaustiveness 없음

Java 초창기부터 있던 형태다. `case ... :` + `break`를 쓰고, 값을 만들지 않고 순수하게 제어 흐름만 담당한다.

```java
void log(int status) {
    switch (status) {
        case 1:
            System.out.println("pending");
            break;
        case 2:
            System.out.println("created");
            break;
        // 다른 값은 그냥 무시되고 지나감 — default 없어도 컴파일 정상
    }
}
```

enum을 switch해도 마찬가지다:

```java
enum Status { PENDING, CREATED, FAILED }

void log(Status s) {
    switch (s) {
        case PENDING: System.out.println("pending"); break;
        case CREATED: System.out.println("created"); break;
        // FAILED 분기 누락 — 컴파일 에러 없음, IDE 경고만 뜰 수 있음
    }
}
```

**이 형태는 statement이고, 값을 만들지 않으므로 exhaustiveness를 요구할 이유가 없다** — Kotlin의 `when` statement와 동일한 성격이다.

## 2. `switch` 식 (화살표, Java 14+) — 값을 만들어야 하므로 exhaustive해야 함

`->`를 쓰고 `= switch(...) { ... }` 형태로 값을 만들어낸다. **값을 반드시 생산해야 하므로, 모든 경우를 커버하지 않으면 컴파일 에러**다.

```java
String label = switch (status) {
    case PENDING -> "pending";
    case CREATED -> "created";
    // FAILED 분기 누락!
};
// 컴파일 에러: "the switch expression does not cover all possible input values"
```

`default`를 추가하거나 모든 enum 상수를 다뤄야 컴파일된다. 이건 Kotlin의 `when` expression과 정확히 같은 규칙이다 — **값을 만드는 식은 반드시 총체적(total)이어야 한다.**

```java
String label = switch (status) {
    case PENDING -> "pending";
    case CREATED -> "created";
    case FAILED -> "failed";     // 전부 커버 → OK
};
```

## 3. 패턴 매칭 `switch` (Java 21, sealed 타입) — statement여도 exhaustive해야 함 ★ Kotlin과 다른 지점

여기가 Kotlin과 **실질적으로 갈리는 부분**이다. Java 21의 패턴 매칭 switch(`case Type t ->`)를 `sealed interface`에 대해 쓰면, **switch가 statement든 expression이든 상관없이** 컴파일러가 exhaustiveness를 강제한다.

```java
public sealed interface Filter permits AndFilter, SearchStringFilter, StatusFilter {}
public record AndFilter(List<Filter> targets) implements Filter {}
public record SearchStringFilter(String value) implements Filter {}
public record StatusFilter(List<String> value) implements Filter {}
```

**switch 식(expression)** — 값을 만드니 당연히 exhaustive해야 함:

```java
String label = switch (filter) {
    case AndFilter f -> "and";
    case SearchStringFilter f -> "search";
    // StatusFilter 누락!
};
// 컴파일 에러
```

**switch 문(statement)** — 값을 안 만드는데도 exhaustive해야 함:

```java
void log(Filter filter) {
    switch (filter) {                       // statement (값 반환 없음)
        case AndFilter f -> System.out.println("and");
        case SearchStringFilter f -> System.out.println("search");
        // StatusFilter 누락!
    }
}
// 컴파일 에러:
// "the switch statement does not cover all possible input values"
```

이건 Kotlin의 `when` statement가 분기를 빠뜨려도 조용히 통과하는 것과 **정반대**다. Java의 패턴 매칭 switch는 sealed 타입에 대해 **statement/expression 구분과 무관하게** 총체성(totality) 분석을 수행한다 (JEP 441). 즉 Java에서 exhaustiveness를 가르는 기준은 "statement냐 expression이냐"가 아니라 **"패턴 매칭(타입 기반 case)을 쓰는 sealed 타입 switch냐 아니냐"**다.

## 4. 실전 예시: DTO → gRPC 매핑

값을 만들어야 하는 매퍼 함수이므로 자연스럽게 switch 식으로 작성하고, `sealed interface`이므로 `default` 없이도 exhaustive 검사를 받는다.

```java
public static FilterNode toFilterProto(Filter filter) {
    return switch (filter) {
        case AndFilter f -> FilterNode.newBuilder()
            .setAnd(AndFilterProto.newBuilder()
                .addAllTargets(f.targets().stream().map(FilterMapper::toFilterProto).toList()))
            .build();

        case SearchStringFilter f -> FilterNode.newBuilder()
            .setSearchString(SearchStringFilterProto.newBuilder().setValue(f.value()))
            .build();

        case StatusFilter f -> FilterNode.newBuilder()
            .setStatus(StatusFilterProto.newBuilder().addAllValues(f.value()))
            .build();
    };
    // sealed interface라 default 없이도 exhaustive — 새 서브타입 추가 시 컴파일 에러로 강제됨
}
```

전체 맥락(REST DTO → gRPC 매핑 설계)은 [`jackson.rest.to.grpc.md`](jackson/jackson.rest.to.grpc.md) 참고.

## 5. Kotlin `when` vs Java `switch` 비교

| | Kotlin `when` | Java `switch` (전통적, 값 라벨) | Java `switch` (패턴 매칭, sealed 타입) |
|---|---|---|---|
| exhaustiveness를 가르는 기준 | statement냐 expression이냐 | 항상 없음 | 항상 있음 (statement/expression 무관) |
| statement로 쓸 때 분기 누락 | 조용히 통과 | 조용히 통과 | **컴파일 에러** |
| expression으로 쓸 때 분기 누락 | 컴파일 에러 (sealed/enum) | 컴파일 에러 (enum, `default` 없으면) | 컴파일 에러 |
| `else`/`default` 필요 여부 | sealed/enum을 다 다루면 불필요 | 값 라벨은 웬만하면 필요 | permitted 하위 타입을 다 다루면 불필요 |

**결론**: "값을 만드는 식은 총체적이어야 한다"는 원칙은 두 언어가 동일하다. 차이는 **"타입 기반 패턴 매칭을 쓰는 statement"**를 어떻게 다루느냐인데, Kotlin은 이 경우에도 여전히 관대하지만(exhaustiveness 검사 안 함), Java 21의 패턴 매칭 switch는 statement든 expression이든 sealed 타입이면 무조건 총체성을 요구한다. 그래서 Kotlin에서 sealed 타입에 대한 컴파일 타임 안전망을 확실히 얻고 싶다면, **의식적으로 `when`을 expression 형태로 작성**해야 한다(본 시리즈의 [`kotlin.when.expression.md`](../kotlin/kotlin.when.expression.md) 참고).
