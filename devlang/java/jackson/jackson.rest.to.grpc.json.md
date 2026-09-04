# Grpc Proto oneof와 json mapping

기준 proto:

```proto
message ReportPlanFilterV1 {
  oneof condition {
    ReportPlanFilterAndV1 and = 1;
    ReportPlanFilterSearchStringV1 search_string = 2;
    ReportPlanFilterStatusV1 status = 3;
    ReportPlanFilterReportCategoryV1 report_category = 4;
    ReportPlanFilterScheduleFrequencyV1 schedule_frequency = 5;
  }
}
```

핵심: `oneof`라서 `type/value`가 아니라 **선택된 필드명이 래퍼 키**가 된다.

---

## 0) oneof와 JSON key 관계 설명

`oneof condition`의 멤버 이름이 protobuf JSON에서 그대로 key가 된다.

- proto 멤버: `and`, `search_string`, `status`, `report_category`, `schedule_frequency`
- JSON key: 위 이름 중 **정확히 하나**가 `filter` 객체 안에 와야 함

예:

```json
{ "filter": { "search_string": { "value": "foo" } } }
```

이 구조는 아래 의미와 같다.

- `condition_case = SEARCH_STRING`
- `search_string.value = "foo"`

반대로 아래는 `oneof` 규칙에 맞지 않는다.

```json
{ "filter": { "type": "SEARCH_STRING", "value": "foo" } }
```

이 형태는 `condition` 멤버(`search_string` 등)를 직접 설정하지 않기 때문에,
현재 서버에서는 `CONDITION_NOT_SET`로 해석되어 `filter condition must be set` 예외가 발생한다.

---

## 1) `ReportPlanFilterV1` 단독 JSON (각 oneof 대안)

### 1. AND
```json
{
  "and": {
    "targets": [
      { "search_string": { "value": "foo" } },
      { "status": { "values": ["PENDING"] } }
    ]
  }
}
```

### 2. SEARCH_STRING
```json
{
  "search_string": {
    "value": "foo"
  }
}
```

### 3. STATUS
```json
{
  "status": {
    "values": ["PENDING", "CREATING", "CREATED", "FAILED", "DELETED"]
  }
}
```

### 4. REPORT_CATEGORY
```json
{
  "report_category": {
    "values": ["BASIC", "UNIFIED", "QUERY"]
  }
}
```

### 5. SCHEDULE_FREQUENCY
```json
{
  "schedule_frequency": {
    "values": [
      "IMMEDIATE",
      "SPECIFIC_TIME",
      "EVERY_MINUTE",
      "EVERY_HOUR",
      "EVERY_DAY",
      "EVERY_WEEK",
      "EVERY_MONTH_DAY",
      "EVERY_MONTH_WEEK",
      "EVERY_YEAR",
      "PERIODIC",
      "PERIODIC_DAY"
    ]
  }
}
```

---

## 2) `ReportPlanListRequestV1` 전체 요청 예제

### A. 단일 검색어 필터
```json
{
  "filter": {
    "search_string": { "value": "foo" }
  },
  "pagination": {
    "page_size": 20,
    "page_number": 1
  },
  "sort": {
    "sort_key": "last_run_at",
    "sort_order": "DESC"
  }
}
```

### B. 복합 AND 필터
```json
{
  "filter": {
    "and": {
      "targets": [
        { "search_string": { "value": "foo" } },
        {
          "status": {
            "values": ["PENDING", "CREATING", "CREATED", "FAILED", "DELETED"]
          }
        },
        {
          "report_category": {
            "values": ["BASIC", "UNIFIED", "QUERY"]
          }
        },
        {
          "schedule_frequency": {
            "values": ["IMMEDIATE", "SPECIFIC_TIME", "EVERY_DAY", "EVERY_WEEK", "EVERY_MONTH_DAY", "EVERY_YEAR"]
          }
        }
      ]
    }
  },
  "pagination": {
    "page_size": 20,
    "page_number": 1
  },
  "sort": {
    "sort_key": "last_run_at",
    "sort_order": "DESC"
  }
}
```

### C. 필터 미지정(필드 생략)
```json
{
  "pagination": {
    "page_size": 20,
    "page_number": 1
  },
  "sort": {
    "sort_key": "last_run_at",
    "sort_order": "DESC"
  }
}
```

---

## 3) 현재 서버 로직 기준 주의사항

1. `filter`를 보냈는데 내부 condition이 비어 있으면(`{ "filter": {} }`) 서버에서  
   `filter condition must be set` 예외가 난다.

2. `page_number`는 현재 **1 이상**만 허용된다.

3. `search_string.value`가 빈 문자열이면 파싱은 되지만, 실제 조회 조건은 사실상 no-op이 된다.

4. `status/report_category/schedule_frequency`의 `values`에 알 수 없는 문자열이 오면 서버에서 예외가 난다.

---

## 4) camelCase 키 형태도 가능한가?

protobuf JSON 파서(`JsonFormat.Parser`)는 일반적으로 proto 원본 필드명(snake_case)과 JSON 이름(lowerCamelCase)을 모두 허용한다.  
즉 아래도 수신 가능하다.

```json
{
  "filter": {
    "searchString": { "value": "foo" }
  },
  "pagination": {
    "pageSize": 20,
    "pageNumber": 1
  },
  "sort": {
    "sortKey": "last_run_at",
    "sortOrder": "DESC"
  }
}
```

다만 현재 BFF DTO/Jackson 레이어를 함께 쓸 때는 팀에서 정한 snake_case 계약으로 통일하는 편이 안전하다.

---

## 5) 왜 “모든 형태”를 유한 개로 나열하기 어려운가

`and.targets` 안에 다시 `ReportPlanFilterV1`가 재귀적으로 들어가므로 조합은 무한대다.  
따라서 실제 “모든 형태”는:

- 원자 조건 4종(`search_string`, `status`, `report_category`, `schedule_frequency`)
- 그룹 조건 1종(`and`)
- 그리고 이들을 재귀 조합한 모든 트리

로 정의된다. 위 예제는 이 규칙을 전부 커버하는 최소 완전 예시다.

---

## 6) oneof 규칙에 맞는 Java/Jackson 매핑 클래스 예시

`type` 필드 기반이 아니라 wrapper key(`and`, `search_string`...)를 판별자로 써야 하므로
`@JsonTypeInfo(include = JsonTypeInfo.As.WRAPPER_OBJECT)`가 핵심이다.

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;
import com.fasterxml.jackson.databind.PropertyNamingStrategies;
import com.fasterxml.jackson.databind.annotation.JsonNaming;
import java.util.List;

@JsonIgnoreProperties(ignoreUnknown = true)
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.WRAPPER_OBJECT
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = ReportPlanFilterDto.And.class, name = "and"),
    @JsonSubTypes.Type(value = ReportPlanFilterDto.SearchString.class, name = "search_string"),
    @JsonSubTypes.Type(value = ReportPlanFilterDto.Status.class, name = "status"),
    @JsonSubTypes.Type(value = ReportPlanFilterDto.ReportCategory.class, name = "report_category"),
    @JsonSubTypes.Type(value = ReportPlanFilterDto.ScheduleFrequency.class, name = "schedule_frequency")
})
public sealed interface ReportPlanFilterDto
    permits ReportPlanFilterDto.And,
            ReportPlanFilterDto.SearchString,
            ReportPlanFilterDto.Status,
            ReportPlanFilterDto.ReportCategory,
            ReportPlanFilterDto.ScheduleFrequency {

    @JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
    record And(List<ReportPlanFilterDto> targets) implements ReportPlanFilterDto {}

    @JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
    record SearchString(String value) implements ReportPlanFilterDto {}

    @JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
    record Status(List<String> values) implements ReportPlanFilterDto {}

    @JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
    record ReportCategory(List<String> values) implements ReportPlanFilterDto {}

    @JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
    record ScheduleFrequency(List<String> values) implements ReportPlanFilterDto {}
}
```

요청 루트 DTO 예시:

```java
@JsonIgnoreProperties(ignoreUnknown = true)
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public record ReportPlanListRequestDto(
    ReportPlanFilterDto filter,
    ReportPlanPaginationDto pagination,
    ReportPlanSortDto sort
) {}
```

정상 매핑 예:

```json
{
  "filter": {
    "and": {
      "targets": [
        { "search_string": { "value": "foo" } },
        { "status": { "values": ["PENDING"] } }
      ]
    }
  }
}
```
