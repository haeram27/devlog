# ilterDto 다형성 JSON 매핑 구조

## 개요
`ReportViewFilterDto`는 Jackson의 `@JsonTypeInfo` + `@JsonSubTypes`를 이용해 JSON의 `type` 필드 값에 따라
서로 다른 서브타입으로 (역)직렬화되는 **sealed class 기반 다형성 DTO**입니다.
조회 필터를 트리 구조로 표현하기 위한 것으로, 최상위는 일반적으로 `And`이며 그 안에 리프 조건들이나
또 다른 `And`를 중첩할 수 있습니다.

## 설계 방식
- `@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.EXISTING_PROPERTY, property = "type")`
  - `use = Id.NAME`: 타입 구분 값으로 클래스 이름 대신 `@JsonSubTypes.Type(name = ...)`에 지정한 논리 이름(`AND`, `TASK_STATUS`, `TASK_TYPE`)을 사용
  - `include = As.EXISTING_PROPERTY`: 구분자 값(`type`)이 JSON 안에 **이미 존재하는 실제 프로퍼티**로 포함됨 (즉, 각 서브타입 클래스가 `type` 필드를 직접 선언하고 있어야 함)
- `@JsonSubTypes`로 `type` 값과 실제 Kotlin 클래스(`And`, `TaskStatus`, `TaskType`)를 매핑
- 모든 클래스는 `@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)`로 snake_case JSON 프로퍼티 사용
- `@Schema(discriminatorProperty = "type", subTypes = [...])`로 Swagger/OpenAPI 문서에도 동일한 다형성 정보 반영
- `ReportFilterType` enum(`AND`, `TASK_STATUS`, `TASK_TYPE`)과 `Names` object 상수가 discriminator 값의 단일 소스(SSOT) 역할

## 타입별 매핑

| type 값 | Kotlin 클래스 | 필드 | 설명 |
|---|---|---|---|
| `AND` | `And` | `targets: List<ReportViewFilterDto>` | 여러 조건을 AND로 결합. 자기 자신(`ReportViewFilterDto`)을 재귀 참조하여 트리 구조 형성 |
| `TASK_STATUS` | `TaskStatus` | `taskStatus: List<String>` (`task_status`) | 포함할 수집 상태 목록. 비우면(`[]`) 조건 없음 |
| `TASK_TYPE` | `TaskType` | `taskType: List<String>` (`task_type`) | 포함할 수집 종류 목록. 비우면(`[]`) 조건 없음 |

## JSON 예시

### 단일 조건
```json
{ "type": "TASK_STATUS", "task_status": ["SUCCESS", "FAIL", "ING"] }
```

### 중첩 AND (복합 필터)
```json
{
  "type": "AND",
  "targets": [
    { "type": "TASK_STATUS", "task_status": ["SUCCESS", "FAIL"] },
    { "type": "TASK_TYPE", "task_type": ["COLLECT_AHNREPORT_FILE"] }
  ]
}
```

## 참고: `JsonTypeInfo.As` (`include`) 옵션

`@JsonTypeInfo`의 `include` 속성은 타입 구분자(discriminator)가 JSON 내 어디에, 어떤 형태로 위치할지를 결정하며 `JsonTypeInfo.As` enum 값으로 지정합니다.

| 값 | 설명 |
|---|---|
| `PROPERTY` | 구분자를 대상 객체와 **같은 레벨의 프로퍼티**로 추가 (기본값). 서브타입 클래스에 해당 필드가 없어도 Jackson이 자동으로 추가/제거 |
| `WRAPPER_OBJECT` | 객체를 `{"typeName": { ...실제 필드... }}` 형태로 한 번 감싸서 표현 |
| `WRAPPER_ARRAY` | `["typeName", { ...실제 필드... }]` 형태의 2원소 배열로 표현 |
| `EXTERNAL_PROPERTY` | 구분자를 **감싸는(부모) 객체**의 형제 프로퍼티로 둠 (필드/속성 자체에 `@JsonTypeInfo`를 붙였을 때 사용, 최상위 값에는 사용 불가) |
| `EXISTING_PROPERTY` | 구분자가 **대상 클래스가 이미 선언한 실제 프로퍼티**와 동일하다고 간주 (현재 `ReportViewFilterDto`에서 사용 중인 방식). 서브타입이 해당 필드를 직접 가지고 있어야 함 |

`ReportViewFilterDto`는 `EXISTING_PROPERTY`를 사용하므로, `And`/`TaskStatus`/`TaskType` 각각이 `type: ReportFilterType` 필드를 명시적으로 선언하고 있으며, 이 값이 곧 JSON의 `type` 필드와 직접 매핑됩니다.
