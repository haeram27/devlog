# REST → gRPC 폴리모픽 필터 구조 설계

이 문서는 다음의 환경에서 특정 목적의 다형성 Json을 전달하가 위한 Dto 정의, GRPC 정의를 설명한다.

`FE > REST API > API Gateway(BE) > Grpc > MSA Service(BE)`

FE 용 REST API 구현시, 화면의 시나리오가 Table에 대해서
Search, Filter(컴럼 값, SQL에서 IN 해당), Pagenation, Sorting 조건을
다형성 Json을 이용해 수신하는 Dto 구조와
이를 다시 Grpc로 전달하는 grpc protobuf정의를 설명한다.

## 1. 문제 정의

주어진 JSON은 **재귀적인 discriminated union(판별 합타입)** 구조다.

```json
{
    "filter": {
        "type": "AND",
        "targets": [
            { "type": "SEARCH_STRING", "value": "foo" },
            { "type": "STATUS", "value": ["PENDING", "CREATING", "CREATED", "FAILED", "DELETED"] },
            { "type": "REPORT_CATEGORY", "value": ["BASIC", "UNIFIED", "QUERY"] },
            { "type": "SCHEDULE_FREQUENCY", "value": ["IMMEDIATE", "SPECIFIC_TIME", "EVERY_DAY", "EVERY_WEEK", "EVERY_MONTH_DAY", "EVERY_YEAR"] }
        ]
    },
    "pagination": { "page_size": 20, "page_number": 1 },
    "sort": { "sort_key": "last_run_at", "sort_order": "DESC" }
}
```

- `filter`는 `type` 필드로 실제 하위 타입이 결정되는 다형성 노드다.
- `AND` 타입은 자기 자신과 같은 타입(`Filter`)의 리스트를 자식으로 가진다 → **재귀 구조**.
- 나머지 리프 타입(`SEARCH_STRING`, `STATUS`, `REPORT_CATEGORY`, `SCHEDULE_FREQUENCY`)은 각자 다른 모양의 `value`를 가진다.

이런 구조를 REST 계층(Java DTO)과 gRPC 계층(proto message) 양쪽에서 "제일 자연스러운 방식으로" 표현하는 것이 핵심 설계 문제다. 아래는 특정 리포지토리의 기존 구현과 무관하게, 일반적으로 권장되는 설계다.

---

## 2. Java REST DTO 설계

### 핵심 원칙: 판별 합타입은 `sealed interface` + `record`로 표현한다

Java 17+에서는 이런 "닫힌 다형성 집합"을 표현하는 가장 안전한 방법이 `sealed interface`다. 컴파일러가 허용된 하위 타입을 강제하고, 이후 `switch` 패턴 매칭에서 **exhaustiveness(누락 없음)**를 컴파일 타임에 검증해준다 — Kotlin의 `sealed class` + `when`과 동일한 이점이다.

```java
// ── 판별 타입 마커 ──────────────────────────────────────────
public enum FilterType {
    AND, SEARCH_STRING, STATUS, REPORT_CATEGORY, SCHEDULE_FREQUENCY
}

// ── 폴리모픽 루트 ───────────────────────────────────────────
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.EXISTING_PROPERTY,
    property = "type"
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = AndFilter.class, name = "AND"),
    @JsonSubTypes.Type(value = SearchStringFilter.class, name = "SEARCH_STRING"),
    @JsonSubTypes.Type(value = StatusFilter.class, name = "STATUS"),
    @JsonSubTypes.Type(value = ReportCategoryFilter.class, name = "REPORT_CATEGORY"),
    @JsonSubTypes.Type(value = ScheduleFrequencyFilter.class, name = "SCHEDULE_FREQUENCY"),
})
public sealed interface Filter
    permits AndFilter, SearchStringFilter, StatusFilter, ReportCategoryFilter, ScheduleFrequencyFilter {
    FilterType type();
}

// ── 리프/재귀 노드 (record = 불변 값 객체) ──────────────────
public record AndFilter(
    @NotEmpty List<@Valid Filter> targets,
    FilterType type
) implements Filter {
    public AndFilter { type = FilterType.AND; }         // 판별자 고정
}

public record SearchStringFilter(
    @NotBlank String value,
    FilterType type
) implements Filter {
    public SearchStringFilter { type = FilterType.SEARCH_STRING; }
}

public record StatusFilter(
    @NotEmpty List<ReportStatus> value,          // 문자열이 아니라 enum 리스트로!
    FilterType type
) implements Filter {
    public StatusFilter { type = FilterType.STATUS; }
}

public record ReportCategoryFilter(
    @NotEmpty List<ReportCategory> value,
    FilterType type
) implements Filter {
    public ReportCategoryFilter { type = FilterType.REPORT_CATEGORY; }
}

public record ScheduleFrequencyFilter(
    @NotEmpty List<ScheduleFrequency> value,
    FilterType type
) implements Filter {
    public ScheduleFrequencyFilter { type = FilterType.SCHEDULE_FREQUENCY; }
}
```

```java
// ── 나머지 요청 본문 (평범한 flat DTO) ──────────────────────
public record Pagination(
    @NotNull @Positive Integer pageSize,
    @NotNull @PositiveOrZero Integer pageNumber
) {}

public record Sort(
    @NotBlank String sortKey,
    @NotBlank String sortOrder     // 또는 enum SortOrder { ASC, DESC }
) {}

public record ReportPlanListRequest(
    @NotNull @Valid Filter filter,
    @NotNull @Valid Pagination pagination,
    @Valid Sort sort               // null 허용 → "정렬 안 함/기본 정렬"
) {}
```

`snake_case` ↔ `camelCase` 변환은 필드마다 `@JsonProperty`를 반복하지 말고, `ObjectMapper` 전역 설정으로 한 번에 처리한다:

```java
objectMapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE);
```

### 왜 `value`를 `String`이 아니라 enum으로 받는가

`STATUS`/`REPORT_CATEGORY`/`SCHEDULE_FREQUENCY`의 `value`를 `List<String>`이 아니라 `List<ReportStatus>` 같은 실제 enum으로 받으면:

- 잘못된 값(`"FOO"`)이 오면 **Jackson 역직렬화 단계에서 자동으로 400**이 나고, 컨트롤러/서비스 로직에서 유효성 검사를 따로 안 해도 된다.
- 이후 gRPC 매핑 코드에서 `switch`로 안전하게 변환할 수 있다(문자열 오타로 인한 런타임 버그를 컴파일 타임 문제로 옮김).

### 방어적 설계: 재귀 깊이/크기 제한

`AND`가 자기 자신을 포함할 수 있는 재귀 구조를 외부 입력으로 그대로 받으면, 악의적이거나 실수로 매우 깊게 중첩된 JSON이 들어와 **StackOverflow(재귀 매핑 시)나 과도한 처리 비용**을 유발할 수 있다. 일반적으로:

- Jackson의 `StreamReadConstraints`(또는 Spring의 `spring.mvc.jackson` 관련 설정)로 JSON 중첩 깊이 자체를 제한하거나,
- 역직렬화 후 커스텀 `@AssertTrue` validator로 트리 깊이(`maxDepth`)를 검증한다.

---

## 3. gRPC 메시지 설계 (proto3)

### 핵심 원칙: 판별 합타입은 `oneof`로 표현한다

proto3의 `oneof`는 "이 메시지 중 정확히 하나만 채워진다"는 것을 프로토콜 레벨에서 표현하는 기능이며, Java의 `sealed interface`/Kotlin의 `sealed class`와 개념적으로 동일하다. `AndFilter`는 자기 자신 타입(`FilterNode`)의 반복 필드를 가지므로 재귀도 그대로 표현된다.

```proto
syntax = "proto3";

message ReportPlanListRequest {
  FilterNode filter = 1;
  Pagination pagination = 2;
  Sort sort = 3;               // proto3 message 타입은 기본적으로 nullable(=optional message)
}

message FilterNode {
  oneof condition {
    AndFilter and = 1;
    SearchStringFilter search_string = 2;
    StatusFilter status = 3;
    ReportCategoryFilter report_category = 4;
    ScheduleFrequencyFilter schedule_frequency = 5;
  }
}

message AndFilter {
  repeated FilterNode targets = 1;
}

message SearchStringFilter {
  string value = 1;
}

message StatusFilter {
  repeated ReportStatus values = 1;        // enum 또는 repeated string 가능
}

message ReportCategoryFilter {
  repeated ReportCategory values = 1;      // enum 또는 repeated string 가능
}

message ScheduleFrequencyFilter {
  repeated ScheduleFrequency values = 1;   // enum 또는 repeated string 가능
}

enum ReportStatus {
  REPORT_STATUS_UNSPECIFIED = 0;   // proto3 enum은 0번 값이 필수 (unset 시 기본값)
  PENDING = 1;
  CREATING = 2;
  CREATED = 3;
  FAILED = 4;
  DELETED = 5;
}

enum ReportCategory {
  REPORT_CATEGORY_UNSPECIFIED = 0;
  BASIC = 1;
  UNIFIED = 2;
  QUERY = 3;
}

enum ScheduleFrequency {
  SCHEDULE_FREQUENCY_UNSPECIFIED = 0;
  IMMEDIATE = 1;
  SPECIFIC_TIME = 2;
  EVERY_DAY = 3;
  EVERY_WEEK = 4;
  EVERY_MONTH_DAY = 5;
  EVERY_YEAR = 6;
}

message Pagination {
  int32 page_size = 1;
  int32 page_number = 2;
}

message Sort {
  string sort_key = 1;
  string sort_order = 2;
}
```

`sort_key`/`sort_order`는 필요하면 `enum SortOrder { ASC = 1; DESC = 2; }`로 강타입화할 수도 있다.

### 트레이드오프: proto `enum` vs `repeated string`

이건 정답이 하나로 정해진 문제가 아니라 **의도적으로 선택해야 하는 설계 트레이드오프**다.

| | proto `enum` 사용 | `repeated string` 사용 |
|---|---|---|
| 타입 안전성 | 컴파일 타임에 보장 | 런타임에만 검증 가능 |
| 스키마 자기 문서화 | `.proto`만 보면 허용값을 알 수 있음 | 별도 문서/코드 확인 필요 |
| 값 추가 시 배포 순서 | REST 서버와 gRPC 서버가 **같은 버전의 proto**를 써야 함(락스텝 배포) | REST/gRPC 서버가 **독립적으로 배포 가능**(새 상태값을 REST 쪽에 먼저 추가해도 gRPC 서버가 몰라도 그냥 문자열로 통과시킬 수 있음) |
| 하위 호환성 | `UNSPECIFIED = 0` 관례로 어느 정도 대응 가능하지만 근본적으로 값 추가 시 재컴파일 필요 | 매우 유연 |

**일반적 권장**: 값의 집합이 두 서비스가 항상 함께 배포되는 내부 시스템이면 `enum`(타입 안전성 우선). 서로 다른 팀/배포 주기를 가진 마이크로서비스 경계라면 `string`이 실무적으로 더 많이 쓰인다 — 이 저장소의 기존 구현이 `repeated string`을 택한 것도 그런 이유일 가능성이 높다.

### 참고: proto3에서 "필수(non-null) 필드"는 어떻게 표현하나

proto3에는 `required` 키워드가 아예 없다(proto2에는 있었지만, 하위 호환성 문제 때문에 proto3에서 의도적으로 제거됨). 그래서 "이 필드는 반드시 값이 있어야 한다"를 proto 스키마 자체로 강제할 방법은 없다.

- **스칼라 필드**(`string`, `int32`, `bool` 등): `optional` 키워드 없이 선언하면 "unset"과 "타입의 zero-value(`""`, `0`, `false`)로 명시적으로 설정"을 구분할 수 없고 `has*()`도 생기지 않는다. `optional` 키워드를 붙여야 presence tracking(`has*()`)이 생긴다.
- **메시지 타입 필드**(nested message): `optional` 키워드 없이도 원래부터 있음/없음이 구분된다(`has*()`가 항상 생성됨). 위 예시의 `Sort sort = 3;`가 이 경우다.

즉 proto3에서 "optional"은 필드 종류에 따라 의미가 다르고, 어느 쪽이든 "필수" 강제는 없다. 값이 반드시 있어야 하는 필드는 애플리케이션 레벨에서 검증해야 한다:

1. **문서/주석 관례**: `// required` 주석으로 계약만 명시(강제력 없음).
2. **서버 핸들러에서 수동 검증**: `if (!request.hasXxx() || request.getXxx().isEmpty()) throw ...`처럼 직접 체크.
3. **`protoc-gen-validate`(PGV) / buf validate 플러그인**: `string name = 1 [(validate.rules).string.min_len = 1];`처럼 선언적으로 제약을 걸고, 생성된 검증 코드가 런타임에 강제하게 만드는 방식 — 실무에서 가장 널리 쓰인다.

REST 쪽의 Bean Validation(`@NotNull`)이 하던 역할을, gRPC 쪽에서는 프로토콜이 아니라 별도 검증 계층이 담당해야 한다는 뜻이다.

---

## 4. DTO → gRPC 매핑

`sealed interface` + `switch` 패턴 매칭(Java 21+)을 쓰면, 새 필터 타입이 추가됐을 때 매핑 함수를 안 고치면 **컴파일 에러**가 나서 누락을 원천 차단한다.

toFilterProto (kotlin)
```java
private fun toFilterProto(filter: Filter): FilterNode {
    val builder = FilterNode.newBuilder()
    when (filter) {
        is AndFilter -> builder.setAnd(
            AndFilter.newBuilder()
                .addAllTargets(filter.targets.map { toFilterProto(it) })
                .build()
        )

        is SearchStringFilter -> builder.setSearchString(
            SearchStringFilter.newBuilder()
                .setValue(filter.value)
                .build()
        )

        is StatusFilter -> builder.setStatus(
            StatusFilter.newBuilder()
                .addAllValues(filter.value.map { it.name })
                .build()
        )

        is ReportCategoryFilter -> builder.setReportCategory(
            ReportCategoryFilter.newBuilder()
                .addAllValues(filter.value.map { it.name })
                .build()
        )

        is ScheduleFrequencyFilter -> builder.setScheduleFrequency(
            ScheduleFrequencyFilter.newBuilder()
                .addAllValues(filter.value.map { it.name })
                .build()
        )
    }
    return builder.build()
}
```

toFilterProto (java)
```java
public final class FilterMapper {

    public static FilterNode toFilterProto(Filter filter) {
        return switch (filter) {
            case AndFilter f -> FilterNode.newBuilder()
                .setAnd(AndFilterProto.newBuilder()
                    .addAllTargets(f.targets().stream().map(FilterMapper::toProto).toList()))
                .build();

            case SearchStringFilter f -> FilterNode.newBuilder()
                .setSearchString(SearchStringFilterProto.newBuilder().setValue(f.value()))
                .build();

            case StatusFilter f -> FilterNode.newBuilder()
                .setStatus(StatusFilterProto.newBuilder()
                    .addAllValues(f.value().stream().map(FilterMapper::toProto).toList()))
                .build();

            case ReportCategoryFilter f -> FilterNode.newBuilder()
                .setReportCategory(ReportCategoryFilterProto.newBuilder()
                    .addAllValues(f.value().stream().map(FilterMapper::toProto).toList()))
                .build();

            case ScheduleFrequencyFilter f -> FilterNode.newBuilder()
                .setScheduleFrequency(ScheduleFrequencyFilterProto.newBuilder()
                    .addAllValues(f.value().stream().map(FilterMapper::toProto).toList()))
                .build();
        };
        // sealed interface라 default 분기 없이도 exhaustive — 새 서브타입 추가 시 컴파일 에러로 강제됨
    }

    private static ReportStatusProto toProto(ReportStatus s) { /* enum → enum 매핑 */ }
}
```

`AndFilter`의 재귀는 `targets`를 순회하며 `toFilterProto`를 재귀 호출하는 것으로 자연스럽게 처리된다 — 트리 구조가 양쪽(DTO/proto)에서 동형(isomorphic)이기 때문에 매핑 코드도 트리를 그대로 따라간다.

---

## 5. 설계 요약

| 계층 | 다형성 표현 | 재귀 표현 | 열거값 표현 |
|---|---|---|---|
| REST JSON | `type` 판별 필드 | `targets: [Filter]` | 문자열 |
| Java DTO | `sealed interface` + `record`, Jackson `@JsonTypeInfo`/`@JsonSubTypes` | `record AndFilter(List<Filter> targets)` | Java `enum` (Jackson이 자동 매핑) |
| gRPC proto | `oneof` | `repeated FilterNode` | proto `enum` (또는 `string`, 트레이드오프 참고) |

핵심은 **세 계층 모두에서 "닫힌 다형성 트리"라는 동일한 개념을 그 계층에 가장 자연스러운 언어 기능(discriminated JSON / sealed interface / oneof)으로 표현**하는 것이다. 그렇게 하면 매핑 코드는 트리 구조를 그대로 따라가는 재귀 함수 하나로 끝나고, 컴파일러가 exhaustiveness를 보장해줘서 새 필터 타입 추가 시 놓치는 지점이 생기지 않는다.
