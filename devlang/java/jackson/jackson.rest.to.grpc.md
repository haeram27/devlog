# REST → gRPC 폴리모픽 필터 구조 설계

이 문서는 다음의 환경에서 특정 목적의 다형성 Json을 전달하가 위한 Dto 정의, GRPC 정의를 설명한다.

`FE > REST API > API Gateway(BE) > Grpc > MSA Service(BE)`

FE 용 REST API 구현시, 목록(리스트)뷰 화면에서
Search(단어 기반 filter), Filter(컴럼 값, SQL에서 IN 해당), Pagenation, Sorting 조건을
다형성 Json을 이용해 수신하는 Dto 구조와
이를 다시 Grpc로 전달하는 grpc protobuf정의를 설명한다.

리스트 뷰 화면에서 Search, Filter, Pagination, Sorting을 구현하려면 4개의 Java 객체 구현이 필요하다.

## 1. 문제 정의

주어진 JSON은 **재귀적인 discriminated union(판별 합타입)** 구조다.

```json
{
    "filter": {
        "type": "AND",
        "targets": [
            {
              "type": "SEARCH_STRING",
              "value": "foo"
            },
            {
              "type": "STATUS",
              "value": ["PENDING", "CREATING", "CREATED", "FAILED", "DELETED"]
            },
            {
              "type": "CATEGORY",
              "value": ["BASIC", "UNIFIED", "QUERY"]
            },
            { 
              "type": "FREQUENCY",
              "value": ["IMMEDIATE", "SPECIFIC_TIME", "EVERY_DAY", "EVERY_WEEK", "EVERY_MONTH_DAY", "EVERY_YEAR"]
            }
        ]
    },
    "pagination": { "page_size": 20, "page_number": 1 },
    "sort": { "sort_key": "last_run_at", "sort_order": "DESC" }
}
```

- `filter`는 `type` 필드로 실제 하위 타입이 결정되는 다형성 노드다.
- `AND` 타입은 자기 자신과 같은 타입(`Filter`)의 리스트를 자식으로 가진다 → **재귀 구조**.
- 나머지 리프 타입(`SEARCH_STRING`, `STATUS`, `CATEGORY`, `FREQUENCY`)은 각자 다른 모양의 `value`를 가진다.

이런 구조를 REST 계층(Java DTO)과 gRPC 계층(proto message) 양쪽에서 "제일 자연스러운 방식으로" 표현하는 것이 핵심 설계 문제다. 아래는 특정 리포지토리의 기존 구현과 무관하게, 일반적으로 권장되는 설계다.

---

## 2. Java REST DTO 설계

### 핵심 원칙: 판별 합타입은 `sealed interface` + `record`로 표현한다

Java 17+에서는 이런 "닫힌 다형성 집합"을 표현하는 가장 안전한 방법이 `sealed interface`다. 컴파일러가 허용된 하위 타입을 강제하고, 이후 `switch` 패턴 매칭에서 **exhaustiveness(누락 없음)**를 컴파일 타임에 검증해준다 — Kotlin의 `sealed class` + `when`과 동일한 이점이다.

### MyFilter.java

```java
package com.example.sample.dto;

import java.util.List;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

import jakarta.validation.Valid;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotEmpty;
import tools.jackson.databind.PropertyNamingStrategies;
import tools.jackson.databind.annotation.JsonNaming;

@JsonIgnoreProperties(ignoreUnknown = true)
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.EXISTING_PROPERTY,
    property = "type"
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = MyFilter.AndFilter.class, name = "AND"),
    @JsonSubTypes.Type(value = MyFilter.SearchStringFilter.class, name = "SEARCH_STRING"),
    @JsonSubTypes.Type(value = MyFilter.StatusFilter.class, name = "STATUS"),
    @JsonSubTypes.Type(value = MyFilter.CategoryFilter.class, name = "CATEGORY"),
    @JsonSubTypes.Type(value = MyFilter.FrequencyFilter.class, name = "FREQUENCY"),
})
public sealed interface MyFilter
    permits MyFilter.AndFilter,
    MyFilter.SearchStringFilter,
    MyFilter.StatusFilter,
    MyFilter.CategoryFilter,
    MyFilter.FrequencyFilter {
    FilterType type();

    // ── 판별 타입 마커 ──────────────────────────────────────────
    enum FilterType {
        AND, SEARCH_STRING, STATUS, CATEGORY, FREQUENCY
    }

    enum Status {
        PENDING, CREATING, CREATED, FAILED, DELETED
    }

    enum Category {
        BASIC, UNIFIED, QUERY
    }

    enum Frequency {
        CUSTOM_DAILY, CUSTOM_WEEKLY, CUSTOM_MONTHLY, DAILY, WEEKLY, MONTHLY
    }
    
    // ── 리프/재귀 노드 (record = 불변 값 객체) ──────────────────
    @JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
    public record AndFilter(
        // @Valid: targets 리스트 각 요소(MyFilter)까지 재귀적으로 Bean Validation 수행
        // @NotEmpty: null/빈 리스트를 허용하지 않음(최소 1개 요소 필요)
        @NotEmpty List<@Valid MyFilter> targets,
        FilterType type
    ) implements MyFilter {
        public AndFilter { type = FilterType.AND; } // 판별자 고정
    }

    @JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
    public record SearchStringFilter(
        // @NotBlank: null/빈 문자열/공백-only 문자열을 허용하지 않음
        @NotBlank String value,
        FilterType type
    ) implements MyFilter {
        public SearchStringFilter { type = FilterType.SEARCH_STRING; }
    }

    @JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
    public record StatusFilter(
        // @NotEmpty: 최소 1개 상태값이 있어야 함
        @NotEmpty List<Status> value,
        FilterType type
    ) implements MyFilter {
        public StatusFilter { type = FilterType.STATUS; }
    }

    @JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
    public record CategoryFilter(
        // @NotEmpty: 최소 1개 카테고리 값이 있어야 함
        @NotEmpty List<Category> value,
        FilterType type
    ) implements MyFilter {
        public CategoryFilter { type = FilterType.CATEGORY; }
    }

    @JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
    public record FrequencyFilter(
        // @NotEmpty: 최소 1개 주기 값이 있어야 함
        @NotEmpty List<String> value,
        FilterType type
    ) implements MyFilter {
        public FrequencyFilter { type = FilterType.FREQUENCY; }
    }
}

```

#### Pagination.java

```java
package com.example.sample.dto;

import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Positive;
import jakarta.validation.constraints.PositiveOrZero;
import tools.jackson.databind.PropertyNamingStrategies;
import tools.jackson.databind.annotation.JsonNaming;

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public record Pagination(
    @NotNull @Positive Integer pageSize,
    // valid minimum pageNumber is 1
    @NotNull @PositiveOrZero Integer pageNumber
) {}
```

#### Sort.java

```java
package com.example.sample.dto;

import jakarta.validation.constraints.NotBlank;
import tools.jackson.databind.PropertyNamingStrategies;
import tools.jackson.databind.annotation.JsonNaming;

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public record Sort(
    @NotBlank String sortKey,
    @NotBlank String sortOrder     // 또는 enum SortOrder { ASC, DESC }
) {}
```

#### ListRequest.java

```java
package com.example.sample.dto;

import jakarta.validation.Valid;
import jakarta.validation.constraints.NotNull;
import tools.jackson.databind.PropertyNamingStrategies;
import tools.jackson.databind.annotation.JsonNaming;

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public record ListRequest(
    @NotNull @Valid MyFilter filter,
    @NotNull @Valid Pagination pagination,
    @Valid Sort sort               // null 허용 → "정렬 안 함/기본 정렬"
) {}
```

`snake_case` ↔ `camelCase` 변환은 필드마다 `@JsonProperty`를 반복하지 말고, 네이밍 전략으로 한 번에 처리한다. 방법은 두 가지이며 **둘 중 하나만 적용하면 된다**.

**(A) 클래스 단위 — 위 예제들이 쓰는 방식**

DTO에 `@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)`를 붙이면 그 클래스에 한해 전략이 적용되며, 전역 설정은 필요 없다. 애플리케이션의 다른 JSON(외부 API 응답 등)은 기본 `camelCase`를 유지하고 싶을 때 적합하다.

**(B) 전역 단위 — 애너테이션 없이 처리**

모든 JSON을 `snake_case`로 통일한다면 DTO의 `@JsonNaming`을 모두 지우고 전역 설정만 둔다.

Spring Boot에서는 프로퍼티로 설정하는 것이 가장 간단하다:

```yaml
spring:
  jackson:
    property-naming-strategy: SNAKE_CASE
```

`ObjectMapper`를 직접 만든다면 Jackson 3(`tools.jackson`)에서는 `ObjectMapper`가 불변이므로 빌더로 지정한다:

```java
ObjectMapper mapper = JsonMapper.builder()
    .propertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE)
    .build();
```

Jackson 2(`com.fasterxml.jackson`)라면 `objectMapper.setPropertyNamingStrategy(PropertyNamingStrategies.SNAKE_CASE)`를 쓴다.

> 우선순위: 클래스의 `@JsonNaming`이 전역 설정보다 우선한다. 둘 다 `SNAKE_CASE`라면 결과는 같지만 중복이므로 한쪽으로 정리하는 편이 좋다.

### 왜 `value`를 `String`이 아니라 enum으로 받는가

`STATUS`/`CATEGORY`/`FREQUENCY`의 `value`를 `List<String>`이 아니라 `List<Status>` 같은 실제 enum으로 받으면:

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
    CategoryFilter category = 4;
    ScheduleFrequencyFilter frequency = 5;
  }
}

message AndFilter {
  repeated FilterNode targets = 1;
}

message SearchStringFilter {
  string value = 1;
}

message StatusFilter {
  repeated Status values = 1;        // enum 또는 repeated string 가능
}

message CategoryFilter {
  repeated Category values = 1;      // enum 또는 repeated string 가능
}

message ScheduleFrequencyFilter {
  repeated Frequency values = 1;   // enum 또는 repeated string 가능
}

enum Status {
  STATUS_UNSPECIFIED = 0;   // proto3 enum은 0번 값이 필수 (unset 시 기본값)
  PENDING = 1;
  CREATING = 2;
  CREATED = 3;
  FAILED = 4;
  DELETED = 5;
}

enum Category {
  CATEGORY_UNSPECIFIED = 0;
  BASIC = 1;
  UNIFIED = 2;
  QUERY = 3;
}

enum Frequency {
  FREQUENCY_UNSPECIFIED = 0;
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

> `when`을 **식(expression, 값 표현, 반환 값이 있는 when)**으로 써야 한다 — 그래야 Kotlin 컴파일러가 sealed interface에 대해 exhaustiveness(누락 없음)를 강제해준다.

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
            SearchStringFilterProto.newBuilder()
                .setValue(filter.value)
        )
        .build()

    is StatusFilter -> FilterNode.newBuilder()
        .setStatus(
            StatusFilterProto.newBuilder()
                .addAllValues(filter.value.map { it.name })
        )
        .build()

    is CategoryFilter -> FilterNode.newBuilder()
        .setReportCategory(
            CategoryFilterProto.newBuilder()
                .addAllValues(filter.value.map { it.name })
        )
        .build()

    is ScheduleFrequencyFilter -> FilterNode.newBuilder()
        .setScheduleFrequency(
            FrequencyFilterProto.newBuilder()
                .addAllValues(filter.value.map { it.name })
        )
        .build()
}
// sealed interface + when-식이라 else 분기 없이도 exhaustive — 새 서브타입 추가 시 컴파일 에러로 강제됨
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

            case CategoryFilter f -> FilterNode.newBuilder()
                .setReportCategory(CategoryFilterProto.newBuilder()
                    .addAllValues(f.value().stream().map(FilterMapper::toProto).toList()))
                .build();

            case ScheduleFrequencyFilter f -> FilterNode.newBuilder()
                .setScheduleFrequency(FrequencyFilterProto.newBuilder()
                    .addAllValues(f.value().stream().map(FilterMapper::toProto).toList()))
                .build();
        };
        // sealed interface라 default 분기 없이도 exhaustive — 새 서브타입 추가 시 컴파일 에러로 강제됨
    }

    private static ReportStatusProto toProto(Status s) { /* enum → enum 매핑 */ }
}
```

`AndFilter`의 재귀는 `targets`를 순회하며 `toFilterProto`를 재귀 호출하는 것으로 자연스럽게 처리된다 — 트리 구조가 양쪽(DTO/proto)에서 동형(isomorphic)이기 때문에 매핑 코드도 트리를 그대로 따라간다.

---

## 4. JPA query

### Specification

- sort specification
```java
package com.example.sample.repository.specification;

import org.springframework.data.jpa.domain.Specification;

import com.example.sample.repository.entity.MyListEntity;

import jakarta.persistence.criteria.CriteriaBuilder;
import jakarta.persistence.criteria.Expression;
import jakarta.persistence.criteria.Root;

/**
 * {@code MyListListRequestV1.sort}(sort_key/sort_order)를 {@code tb_report_plan} 조회 정렬로
 * 변환한다.
 *
 * <p>정렬 대상 6종({@code name}/{@code description}/{@code viewer_admin_ids}(첫 항목)/
 * {@code admin_id}/{@code details}(JSON 0번째 요소의 {@code base_report_id})/
 * {@code last_run_at}) 중 일부는 단순 컬럼이 아니라 배열/JSON 경로 추출식이라
 * {@code Sort.by("property")}로 표현할 수 없다. 그래서 {@link Specification}에서
 * {@code CriteriaQuery.orderBy(...)}를 직접 설정하고 조건 자체는 {@code cb.conjunction()}(no-op)을
 * 반환해, 필터 {@link Specification}과 {@code .and()}로 결합해 하나의 쿼리로 실행한다.
 *
 * <p>{@code sort_key}/{@code sort_order} 화이트리스트 검증은 {@link #orderBy}에서 즉시(eager) 수행한다 —
 * {@link Specification}의 {@code toPredicate}(DB 접근 시점)까지 미루지 않아, 잘못된 요청을 쿼리 실행 전에
 * {@link IllegalArgumentException}으로 빠르게 실패시킨다.
 */
public final class MyListSortSpecifications {

    private MyListSortSpecifications() {
    }

    /** 허용된 {@code sort_key} 화이트리스트. 그 외 값은 {@link #orderBy}에서 즉시 거부된다. */
    private enum SortKey {
        NAME("name"),
        DESCRIPTION("description"),
        VIEWER_ADMIN_IDS("viewer_admin_ids"),
        BASIC_REPORT_DETAILS("details"),
        ADMIN_ID("admin_id"),
        ADMIN_IP("admin_ip"),
        LAST_RUN_AT("last_run_at"),
        MODIFIED_AT("modified_at"),
        CREATED_AT("created_at");

        private final String wireValue;

        SortKey(String wireValue) {
            this.wireValue = wireValue;
        }

        static SortKey fromWireValue(String value) {
            for (SortKey key : values()) {
                if (key.wireValue.equals(value)) {
                    return key;
                }
            }
            throw new IllegalArgumentException("Unknown sort_key value: " + value);
        }
    }

    public static Specification<MyListEntity> orderBy(String sortKey, String sortOrder) {
        if (sortKey == null || sortKey.isBlank()) {
            return Specification.unrestricted();
        }
        SortKey key = SortKey.fromWireValue(sortKey);
        boolean ascending = toAscending(sortOrder);
        return (root, query, cb) -> {
            Expression<?> sortExpression = toSortExpression(key, root, cb);
            query.orderBy(ascending ? cb.asc(sortExpression) : cb.desc(sortExpression));
            return cb.conjunction();
        };
    }

    private static Expression<?> toSortExpression(SortKey key, Root<MyListEntity> root, CriteriaBuilder cb) {
        return switch (key) {
            case NAME -> root.get("name");
            case DESCRIPTION -> root.get("description");
            case ADMIN_ID -> root.get("adminId");
            case ADMIN_IP -> root.get("adminIp");
            case LAST_RUN_AT -> root.get("lastRunAt");
            case MODIFIED_AT -> root.get("modifiedAt");
            case CREATED_AT -> root.get("createdAt");
            // Postgres text[]는 1-based 인덱스. Hibernate 6의 array_get(array, index) 함수를 사용한다.
            case VIEWER_ADMIN_IDS -> cb.function(
                "array_get", String.class, root.get("viewerAdminIds"), cb.literal(1));
            // details(jsonb 배열) 0번째 요소의 base_report_id.
            // 숫자 인덱스를 경로 요소로 넘기면 jsonb 배열 인덱싱으로 동작한다.
            case BASIC_REPORT_DETAILS -> cb.function(
                "jsonb_extract_path_text", String.class, root.get("basicReportDetails"),
                cb.literal("0"), cb.literal("base_report_id"));
        };
    }

    private static boolean toAscending(String sortOrder) {
        if (sortOrder == null || sortOrder.isBlank()) {
            return true;
        }
        if ("ASC".equalsIgnoreCase(sortOrder)) {
            return true;
        }
        if ("DESC".equalsIgnoreCase(sortOrder)) {
            return false;
        }
        throw new IllegalArgumentException("Unknown sort_order value: " + sortOrder);
    }
}
```

- 

```java
package com.example.sample.repository.specification;

import java.util.List;

import org.springframework.data.jpa.domain.Specification;

import com.example.sample.dto.filter.MyListFilterDto;
import com.example.sample.dto.scheduler.MyListScheduleDtoV1.MyListScheduleTypeDtoV1;
import com.example.sample.repository.entity.MyListEntity;
import com.example.sample.repository.enums.MyListCategory;
import com.example.sample.repository.enums.MyListStatus;

import jakarta.persistence.criteria.Expression;

/**
 * {@link MyListFilterDto}(재귀 {@code oneof} 필터 트리)를 {@code tb_my_list} 조회용
 * {@link Specification}으로 변환한다.
 *
 * <p>각 조건의 {@code values}/{@code targets}가 비어 있으면 "조건 없음"으로 간주해 전체를
 * 통과시킨다({@code Specification.where(null)}). enum으로 변환할 수 없는 값(잘못된
 * status/MyList_category/schedule_frequency 문자열)은 {@link IllegalArgumentException}으로
 * 즉시 실패시킨다(기존 {@code toMyListCategory} 등과 동일한 컨벤션).
 */
public final class MyListSpecifications {

    private MyListSpecifications() {
    }

    public static Specification<MyListEntity> fromFilter(MyListFilterDto filter) {
        if (filter == null) {
            return Specification.unrestricted();
        }
        return switch (filter) {
            case MyListFilterDto.AndDto and -> fromAnd(and);
            case MyListFilterDto.SearchStringDto searchString -> fromSearchString(searchString);
            case MyListFilterDto.StatusDto status -> fromStatus(status);
            case MyListFilterDto.MyListCategoryDto MyListCategory -> fromMyListCategory(MyListCategory);
            case MyListFilterDto.ScheduleFrequencyDto scheduleFrequency -> fromScheduleFrequency(scheduleFrequency);
        };
    }

    private static Specification<MyListEntity> fromAnd(MyListFilterDto.AndDto and) {
        List<MyListFilterDto> targets = and.targets();
        if (targets == null || targets.isEmpty()) {
            return Specification.unrestricted();
        }
        return Specification.allOf(targets.stream().map(MyListSpecifications::fromFilter).toList());
    }

    /** {@code name}/{@code description}/{@code admin_id} 중 하나라도 대소문자 구분 없이 포함하면 매칭한다. */
    private static Specification<MyListEntity> fromSearchString(MyListFilterDto.SearchStringDto searchString) {
        String value = searchString.value();
        if (value == null || value.isBlank()) {
            return Specification.unrestricted();
        }
        String pattern = "%" + value.toLowerCase() + "%";
        return (root, query, cb) -> cb.or(
            cb.like(cb.lower(root.get("name")), pattern),
            cb.like(cb.lower(root.get("description")), pattern),
            cb.like(cb.lower(root.get("adminId")), pattern),
            cb.like(cb.lower(root.get("adminIp")), pattern));
    }

    private static Specification<MyListEntity> fromStatus(MyListFilterDto.StatusDto status) {
        List<String> values = status.values();
        if (values == null || values.isEmpty()) {
            return Specification.unrestricted();
        }
        List<MyListStatus> statuses = values.stream().map(MyListSpecifications::toMyListStatus).toList();
        return (root, query, cb) -> root.get("status").in(statuses);
    }

    private static Specification<MyListEntity> fromMyListCategory(MyListFilterDto.MyListCategoryDto MyListCategory) {
        List<String> values = MyListCategory.values();
        if (values == null || values.isEmpty()) {
            return Specification.unrestricted();
        }
        List<MyListCategory> categories = values.stream().map(MyListSpecifications::toMyListCategory).toList();
        return (root, query, cb) -> root.get("MyListCategory").in(categories);
    }

    /**
     * {@code schedule}(jsonb) 컬럼에 {@code {"type": "EVERY_WEEK", ...}} 형태로 저장된 스케줄 종류를
     * {@code jsonb_extract_path_text}로 추출해 비교한다.
     */
    private static Specification<MyListEntity> fromScheduleFrequency(
        MyListFilterDto.ScheduleFrequencyDto scheduleFrequency) {
        List<String> values = scheduleFrequency.values();
        if (values == null || values.isEmpty()) {
            return Specification.unrestricted();
        }
        List<String> scheduleTypes = values.stream()
            .map(MyListSpecifications::toScheduleTypeName)
            .toList();
        return (root, query, cb) -> {
            Expression<String> scheduleType = cb.function(
                "jsonb_extract_path_text", String.class, root.get("schedule"), cb.literal("type"));
            return scheduleType.in(scheduleTypes);
        };
    }

    private static MyListStatus toMyListStatus(String value) {
        try {
            return MyListStatus.valueOf(value);
        } catch (IllegalArgumentException e) {
            throw new IllegalArgumentException("Unknown status value: " + value, e);
        }
    }

    private static MyListCategory toMyListCategory(String value) {
        try {
            return MyListCategory.valueOf(value);
        } catch (IllegalArgumentException e) {
            throw new IllegalArgumentException("Unknown MyList_category value: " + value, e);
        }
    }

    private static String toScheduleTypeName(String value) {
        try {
            return MyListScheduleTypeDtoV1.valueOf(value).name();
        } catch (IllegalArgumentException e) {
            throw new IllegalArgumentException("Unknown schedule_frequency value: " + value, e);
        }
    }
}
```

```java
@Service
@Slf4j
@RequiredArgsConstructor
public class MyListService {

    private final MyListRepository myListRepository;

    /**
     * {@code getMyListV1()} 요청으로 {@code tb_report_plan}에서 단건 조회한 엔티티를 반환한다.
     */
    public MyListEntity getMyList(long reportPlanId) {
        return myListRepository.findById(reportPlanId)
                .orElseThrow(() -> new IllegalArgumentException(
                    "MyList not found for report_plan_id: " + reportPlanId));
    }

    /**
     * {@code getMyListListV1()} 요청의 {@code filter}/{@code sort}/{@code pagination}을 적용해
     * {@code tb_report_plan}을 조회한다. {@code pagination}이 없으면 기존과 동일하게 전체 조회한다
     * (하위 호환).
     */
    public Page<MyListEntity> getMyListList(MyListListRequestV1 request) {
        MyListFilterDto filterDto = request.hasFilter() ? toFilterDto(request.getFilter()) : null;
        Specification<MyListEntity> spec = MyListSpecifications.fromFilter(filterDto);
        if (request.hasSort()) {
            spec = spec.and(MyListSortSpecifications.orderBy(
                request.getSort().getSortKey(), request.getSort().getSortOrder()));
        }
        Pageable pageable = toPageable(request);
        Specification<MyListEntity> finalSpec = spec;
        return myListRepository.findAll(finalSpec, pageable);
    }
}
```

## 5. 설계 요약

| 계층 | 다형성 표현 | 재귀 표현 | 열거값 표현 |
|---|---|---|---|
| REST JSON | `type` 판별 필드 | `targets: [Filter]` | 문자열 |
| Java DTO | `sealed interface` + `record`, Jackson `@JsonTypeInfo`/`@JsonSubTypes` | `record AndFilter(List<Filter> targets)` | Java `enum` (Jackson이 자동 매핑) |
| gRPC proto | `oneof` | `repeated FilterNode` | proto `enum` (또는 `string`, 트레이드오프 참고) |

핵심은 **세 계층 모두에서 "닫힌 다형성 트리"라는 동일한 개념을 그 계층에 가장 자연스러운 언어 기능(discriminated JSON / sealed interface / oneof)으로 표현**하는 것이다. 그렇게 하면 매핑 코드는 트리 구조를 그대로 따라가는 재귀 함수 하나로 끝나고, 컴파일러가 exhaustiveness를 보장해줘서 새 필터 타입 추가 시 놓치는 지점이 생기지 않는다.
