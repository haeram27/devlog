# Jackson에서 json 다형성(Polymorphic) 직렬화/역직렬화

다형성(polymorphic) 타입이란 하나의 공통 타입으로 취급되지만 실제로는 여러 구체 타입(하위 타입)을 나뉘는 타입이다. 이런 타입을 JSON으로 표현할 때는 모든 하위 타입에 공통인 고정부(타입 식별자 등)와, 하위 타입마다 다른 변동부로 구조가 나뉘는 경우가 많다. `@JsonTypeInfo`와 `@JsonSubTypes`는 Jackson에서 이런 다형성 JSON을 올바른 하위 타입으로 매핑하기 위한 어노테이션이다.

## 핵심 요약

- `@JsonTypeInfo`와 `@JsonSubTypes`는 역할이 나뉩니다.
  - `@JsonTypeInfo`: 타입 식별자 지정, 부모(추상) 타입에 선언해 "타입 식별자를 어떤 방식(`use`)으로, 어디에(`include`) 기록할지" 규칙을 정의합니다.
  - `@JsonSubTypes`: 실제 하위 클래스 지정, 그 규칙에서 사용할 **논리적 이름(name) ↔ 실제 하위 클래스(value)** 매핑 테이블을 정의합니다.
- 동작 흐름:
  - **역직렬화 시**: JSON에서 `@JsonTypeInfo(property = "type")`로 지정된 필드 값(예: `"DOG"`)을 읽음 → `@JsonSubTypes` 매핑 테이블에서 일치하는 클래스(`Dog::class`)를 찾음 → 해당 클래스의 인스턴스로 변환.
  - **직렬화 시**: 반대로 실제 런타임 클래스(`Dog`)를 보고 매핑 테이블에서 대응하는 `name`(`"DOG"`)을 찾아 JSON 필드 값으로 기록.
- `@JsonTypeInfo`만으로는 "타입 정보를 어떻게 표현할지"만 정해질 뿐 실제 값-클래스 대응은 알 수 없고, `@JsonSubTypes`가 매핑 테이블 역할을 해야 실제 클래스 변환이 가능합니다. (`Id.NAME` 방식일 때 필수 조합이며, `Id.CLASS`/`Id.MINIMAL_CLASS`는 클래스명 자체를 식별자로 쓰므로 `@JsonSubTypes` 없이도 동작 가능합니다.)

## 어노테이션 설명

**`@JsonTypeInfo`**

- 다형성 타입 처리를 위해 직렬화/역직렬화 시 JSON에 "타입 정보"를 함께 기록하거나 읽어오도록 지시하는 어노테이션입니다.
- 부모(추상) 클래스/인터페이스에 선언하며, 하위 타입 중 어떤 것으로 변환해야 하는지 판별하는 기준을 정의합니다.
- 주요 파라미터:
  - `use`: 타입을 식별하는 방식을 지정합니다. (`JsonTypeInfo.Id`)
    - `Id.NAME`: 별도로 지정한 논리적 이름(문자열)으로 타입을 식별합니다. (`@JsonSubTypes`와 함께 사용하는 것이 일반적)
    - `Id.CLASS`: 완전한 클래스명(FQCN)을 그대로 타입 식별자로 사용합니다.
    - `Id.MINIMAL_CLASS`: 공통 패키지 경로를 생략한 축약된 클래스명을 사용합니다.
    - `Id.DEDUCTION`: 타입 식별자 없이 JSON 필드 구성만으로 타입을 추론합니다.
  - `include`: 타입 정보를 JSON의 어느 위치에 포함할지 지정합니다. (`JsonTypeInfo.As`)
    - `As.PROPERTY`: 타입 정보를 별도의 프로퍼티(필드)로 추가합니다.
    - `As.EXISTING_PROPERTY`: 이미 클래스에 존재하는 프로퍼티를 타입 식별자로 재사용합니다. (필드가 중복 추가되지 않음)
    - `As.WRAPPER_OBJECT`: `{ "타입명": { ...실제 데이터... } }` 형태로 객체를 한 번 더 감쌉니다.
    - `As.WRAPPER_ARRAY`: `[ "타입명", { ...실제 데이터... } ]` 형태의 2개짜리 배열로 감쌉니다.
  - `property`: 타입 식별자를 저장할 JSON 필드명을 지정합니다. (예: `"type"`)
  - `visible`: `true`로 설정하면 타입 식별용 프로퍼티 값을 역직렬화된 객체의 필드에도 그대로 매핑합니다. (기본값 `false`)
  - `defaultImpl`: JSON에 타입 정보가 없거나 일치하는 타입을 찾지 못했을 때 사용할 기본 구현 클래스를 지정합니다.

**`@JsonSubTypes(...)`**

- `@JsonTypeInfo(use = Id.NAME, ...)`와 함께 사용되며, 논리적 타입 이름(name)과 실제 하위 클래스(Java/Kotlin class) 간의 매핑 테이블을 정의하는 어노테이션입니다.
- 부모 클래스에 선언하며, 내부에 여러 개의 `@JsonSubTypes.Type` 항목을 배열로 나열합니다.
- `@JsonSubTypes.Type`의 주요 파라미터:
  - `value`: 매핑 대상이 되는 하위 클래스(`::class` 또는 `.class`)를 지정합니다.
  - `name`: 해당 하위 클래스에 대응하는 논리적 이름(문자열)을 지정합니다. `@JsonTypeInfo`의 `property`에 지정된 필드 값과 매칭됩니다.
  - `names`: 하나의 클래스에 여러 개의 별칭(이름)을 매핑하고 싶을 때 사용하는 문자열 배열입니다. (Jackson 2.12+)
- 동작 방식:
  - **직렬화 시**: 객체의 실제 런타임 클래스를 보고, 매핑 테이블에서 일치하는 `name`을 찾아 JSON의 타입 필드 값으로 씁니다.
  - **역직렬화 시**: JSON의 타입 필드 값을 읽고, 매핑 테이블에서 일치하는 `value`(클래스)를 찾아 해당 타입의 인스턴스로 생성합니다.

## 다형성(Polymorphic) JSON 직렬화/역직렬화 예제 - 스케줄 정보

### 구현 내용

다음 테이블은 스케줄 타입 별로 변동 parameter를 구조를 정의한다.

각 스케줄 타입은 서로 다른 파라미터를 갖기 때문에 스케줄을 담을 class는 scheduler_type 별로 다른 형상을 가져야만한다.

그러므로 스케줄 정보를 담는 json은 type에 따라 변동되 형상을 갖게 된다. 이 변동 부분을 코드 레벨에서 Java의 다형성을 이용해 핸들링 하는 것을 목표로 등장한 기술이 Jackson의 `@JsonTypeInfo`와 `@JsonSubTypes`이다.

schedule_type 별 파라미터

| scheduler_type   | 파라미터 필드 | 파라미터 객체 필드 | 설명 |
|---|---|---|---|
| IMMEDIATE        | -                       | -                      | 즉시 1회 실행. 추가 파라미터 없음 |
| SPECIFIC_TIME    | specific_time_params    | date_time: string (ISO-8601 OffsetDateTime)   | 지정 일시에 1회 실행 |
| EVERY_MINUTE     | every_minute_params     | second: integer (0~59) | 매분 지정 초에 실행 |
| EVERY_HOUR       | every_hour_params       | minute: integer (0~59) | 매시간 지정 분:초에 실행 |
|                  |                         | second: integer (0~59) |    |
| EVERY_DAY        | every_day_params        | hour: integer (0~23)   | 매일 지정 시:분:초에 실행 |
|                  |                         | minute: integer (0~59) |    |
|                  |                         | second: integer (0~59) |    |
| EVERY_WEEK       | every_week_params       | days_of_week: list (요일 목록, MONDAY~SUNDAY) | 매주 지정 요일 시:분:초에 실행 |
|                  |                         | hour: integer (0~23)   |    |
|                  |                         | minute: integer (0~59) |    |
|                  |                         | second: integer (0~59) |    |
| EVERY_MONTH_DAY  | every_month_day_params  | days_of_month: list (1~31 일 목록) | 매월 지정 일 시:분:초에 실행 |
|                  |                         | hour: integer (0~23)   |    |
|                  |                         | minute: integer (0~59) |    |
|                  |                         | second: integer (0~59) |    |
| EVERY_MONTH_WEEK | every_month_week_params | day_of_week: string (MONDAY~SUNDAY) | 매월 N째 주 지정 요일 시:분:초에 실행 |
|                  |                         | week_order: integer (1~5, 몇째 주) |    |
|                  |                         | hour: integer (0~23)   |    |
|                  |                         | minute: integer (0~59) |    |
|                  |                         | second: integer (0~59) |    |
| EVERY_YEAR       | every_year_params       | months: list (1~12 월 목록) | 매년 지정 월/일 시:분:초에 실행       |
|                  |                         | day_of_month: integer (1~31) |    |
|                  |                         | hour: integer (0~23)   |    |
|                  |                         | minute: integer (0~59) |    |
|                  |                         | second: integer (0~59) |    |
| PERIODIC         | periodic_params         | interval_seconds: long (반복 간격 초) | 지정 간격(초)마다 반복 실행 |
| PERIODIC_DAY     | periodic_day_params     | period_days: integer (반복 일 수)     | N일마다 지정 시각에 실행 |
|                  |                         | time_of_day: string (HH:mm:ss, LocalTime)     |    |

### java 예제

sealed interface + record 조합 사용

#### 1. Base/Sub클래스 정의

```java
package com.example.dto.scheduler;

import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;
import java.time.DayOfWeek;
import java.util.List;

/**
 * Jackson의 Json 다형성 맵핑을 위한 Schedule Dto 구현
 *
 * <p>판별자({@code type})가 params와 같은 객체 안에 함께 존재하므로
 * {@code EXISTING_PROPERTY}를 사용한다 — 판별자가 형제 필드로 분리되어 있는 경우(예:
 * {@code AgentMasterSearchRequestDtoV1}의 {@code EXTERNAL_PROPERTY})와는 다른 케이스다.
 * record 기반 생성자 바인딩에서 판별자 프로퍼티 값이 그대로 채워지도록 {@code visible = true}가
 * 필요하다.
 *
 * <p>모든 직접 서브타입과 각 서브타입의 params 클래스({@code SpecificTimeParamsDtoV1} 등)가
 * 이 파일 안에 nested record로 선언되어 있으므로 컴파일러가 {@code permits} 목록을 자동으로
 * 추론한다 — 명시적으로 적지 않아도 된다. 다른 패키지에서 params 클래스를 참조할 때는
 * {@code ReportScheduleDtoV1.SpecificTimeParamsDtoV1}처럼 정규화하거나 nested type을
 * import해서 사용한다.
 */
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.EXISTING_PROPERTY,
    property = "type",
    visible = true
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = ReportScheduleDtoV1.Immediate.class, name = "IMMEDIATE"),
    @JsonSubTypes.Type(value = ReportScheduleDtoV1.SpecificTime.class, name = "SPECIFIC_TIME"),
    @JsonSubTypes.Type(value = ReportScheduleDtoV1.EveryMinute.class, name = "EVERY_MINUTE"),
    @JsonSubTypes.Type(value = ReportScheduleDtoV1.EveryHour.class, name = "EVERY_HOUR"),
    @JsonSubTypes.Type(value = ReportScheduleDtoV1.EveryDay.class, name = "EVERY_DAY"),
    @JsonSubTypes.Type(value = ReportScheduleDtoV1.EveryWeek.class, name = "EVERY_WEEK"),
    @JsonSubTypes.Type(value = ReportScheduleDtoV1.EveryMonthDay.class, name = "EVERY_MONTH_DAY"),
    @JsonSubTypes.Type(value = ReportScheduleDtoV1.EveryMonthWeek.class, name = "EVERY_MONTH_WEEK"),
    @JsonSubTypes.Type(value = ReportScheduleDtoV1.EveryYear.class, name = "EVERY_YEAR"),
    @JsonSubTypes.Type(value = ReportScheduleDtoV1.Periodic.class, name = "PERIODIC"),
    @JsonSubTypes.Type(value = ReportScheduleDtoV1.PeriodicDay.class, name = "PERIODIC_DAY"),
})
public sealed interface ReportScheduleDtoV1 {

    // 구현 record가 반드시 이 시그니처(이름+반환타입)를 만족하는 accessor를 갖도록 강제하는 추상 메서드
    ReportScheduleTypeDtoV1 type();

    String timezone();

    /**
     * 스케줄 실행 방식 판별자.
     *
     * <p>{@code report_plan.proto}의 {@code ScheduleTypeV1}과 동일한 상수 구성을 갖는다
     * ({@code SCHEDULE_TYPE_UNSPECIFIED}는 JSON 계약에는 없으므로 제외).
     */
    enum ReportScheduleTypeDtoV1 {
        IMMEDIATE,
        SPECIFIC_TIME,
        EVERY_MINUTE,
        EVERY_HOUR,
        EVERY_DAY,
        EVERY_WEEK,
        EVERY_MONTH_DAY,
        EVERY_MONTH_WEEK,
        EVERY_YEAR,
        PERIODIC,
        PERIODIC_DAY
    }

    /**
     * 즉시 1회 실행. 추가 파라미터 없음.
     *
     * <pre>{@code
     * {
     *   "type": "IMMEDIATE",
     *   "timezone": "UTC"
     * }
     * }</pre>
     */
    record Immediate(
        @JsonProperty("type") ReportScheduleTypeDtoV1 type,
        @JsonProperty("timezone") String timezone
    ) implements ReportScheduleDtoV1 {

    }

    /**
     * 지정 일시에 1회 실행.
     *
     * <pre>{@code
     * {
     *   "type": "SPECIFIC_TIME",
     *   "timezone": "UTC",
     *   "params": {
     *     "date_time": "2026-08-12T00:00:00+09:00"
     *   }
     * }
     * }</pre>
     */
    record SpecificTime(
        @JsonProperty("type") ReportScheduleTypeDtoV1 type,
        @JsonProperty("timezone") String timezone,
        @JsonProperty("params") SpecificTimeParamsDtoV1 params
    ) implements ReportScheduleDtoV1 {

    }

    /** 지정 일시에 1회 실행. {@code report_plan.proto}의 {@code SpecificTimeParamsV1}. */
    record SpecificTimeParamsDtoV1(
        @JsonProperty("date_time") String dateTime
    ) {

    }

    /**
     * 매분 지정 초에 실행.
     *
     * <pre>{@code
     * {
     *   "type": "EVERY_MINUTE",
     *   "timezone": "UTC",
     *   "params": {
     *     "second": 30
     *   }
     * }
     * }</pre>
     */
    record EveryMinute(
        @JsonProperty("type") ReportScheduleTypeDtoV1 type,
        @JsonProperty("timezone") String timezone,
        @JsonProperty("params") EveryMinuteParamsDtoV1 params
    ) implements ReportScheduleDtoV1 {

    }

    /** 매분 지정 초에 실행. {@code report_plan.proto}의 {@code EveryMinuteParamsV1}. */
    record EveryMinuteParamsDtoV1(
        @JsonProperty("second") Integer second
    ) {

    }

    /**
     * 매시간 지정 분:초에 실행.
     *
     * <pre>{@code
     * {
     *   "type": "EVERY_HOUR",
     *   "timezone": "UTC",
     *   "params": {
     *     "minute": 15,
     *     "second": 30
     *   }
     * }
     * }</pre>
     */
    record EveryHour(
        @JsonProperty("type") ReportScheduleTypeDtoV1 type,
        @JsonProperty("timezone") String timezone,
        @JsonProperty("params") EveryHourParamsDtoV1 params
    ) implements ReportScheduleDtoV1 {

    }

    /** 매시간 지정 분:초에 실행. {@code report_plan.proto}의 {@code EveryHourParamsV1}. */
    record EveryHourParamsDtoV1(
        @JsonProperty("minute") Integer minute,
        @JsonProperty("second") Integer second
    ) {

    }

    /**
     * 매일 지정 시:분:초에 실행.
     *
     * <pre>{@code
     * {
     *   "type": "EVERY_DAY",
     *   "timezone": "UTC",
     *   "params": {
     *     "hour": 9,
     *     "minute": 15,
     *     "second": 30
     *   }
     * }
     * }</pre>
     */
    record EveryDay(
        @JsonProperty("type") ReportScheduleTypeDtoV1 type,
        @JsonProperty("timezone") String timezone,
        @JsonProperty("params") EveryDayParamsDtoV1 params
    ) implements ReportScheduleDtoV1 {

    }

    /** 매일 지정 시:분:초에 실행. {@code report_plan.proto}의 {@code EveryDayParamsV1}. */
    record EveryDayParamsDtoV1(
        @JsonProperty("hour") Integer hour,
        @JsonProperty("minute") Integer minute,
        @JsonProperty("second") Integer second
    ) {

    }

    /**
     * 매주 지정 요일 시:분:초에 실행.
     *
     * <pre>{@code
     * {
     *   "type": "EVERY_WEEK",
     *   "timezone": "UTC",
     *   "params": {
     *     "days_of_week": ["MONDAY", "FRIDAY"],
     *     "hour": 9,
     *     "minute": 15,
     *     "second": 30
     *   }
     * }
     * }</pre>
     */
    record EveryWeek(
        @JsonProperty("type") ReportScheduleTypeDtoV1 type,
        @JsonProperty("timezone") String timezone,
        @JsonProperty("params") EveryWeekParamsDtoV1 params
    ) implements ReportScheduleDtoV1 {

    }

    /** 매주 지정 요일 시:분:초에 실행. {@code report_plan.proto}의 {@code EveryWeekParamsV1}. */
    record EveryWeekParamsDtoV1(
        @JsonProperty("days_of_week") List<DayOfWeek> daysOfWeek,
        @JsonProperty("hour") Integer hour,
        @JsonProperty("minute") Integer minute,
        @JsonProperty("second") Integer second
    ) {

    }

    /**
     * 매월 지정 일 시:분:초에 실행.
     *
     * <pre>{@code
     * {
     *   "type": "EVERY_MONTH_DAY",
     *   "timezone": "UTC",
     *   "params": {
     *     "days_of_month": [1, 15],
     *     "hour": 9,
     *     "minute": 15,
     *     "second": 30
     *   }
     * }
     * }</pre>
     */
    record EveryMonthDay(
        @JsonProperty("type") ReportScheduleTypeDtoV1 type,
        @JsonProperty("timezone") String timezone,
        @JsonProperty("params") EveryMonthDayParamsDtoV1 params
    ) implements ReportScheduleDtoV1 {

    }

    /** 매월 지정 일 시:분:초에 실행. {@code report_plan.proto}의 {@code EveryMonthDayParamsV1}. */
    record EveryMonthDayParamsDtoV1(
        @JsonProperty("days_of_month") List<Integer> daysOfMonth,
        @JsonProperty("hour") Integer hour,
        @JsonProperty("minute") Integer minute,
        @JsonProperty("second") Integer second
    ) {

    }

    /**
     * 매월 N째 주 지정 요일 시:분:초에 실행.
     *
     * <pre>{@code
     * {
     *   "type": "EVERY_MONTH_WEEK",
     *   "timezone": "UTC",
     *   "params": {
     *     "day_of_week": "MONDAY",
     *     "week_order": 2,
     *     "hour": 9,
     *     "minute": 15,
     *     "second": 30
     *   }
     * }
     * }</pre>
     */
    record EveryMonthWeek(
        @JsonProperty("type") ReportScheduleTypeDtoV1 type,
        @JsonProperty("timezone") String timezone,
        @JsonProperty("params") EveryMonthWeekParamsDtoV1 params
    ) implements ReportScheduleDtoV1 {

    }

    /**
     * 매월 N째 주 지정 요일 시:분:초에 실행. {@code report_plan.proto}의
     * {@code EveryMonthWeekParamsV1}.
     */
    record EveryMonthWeekParamsDtoV1(
        @JsonProperty("day_of_week") DayOfWeek dayOfWeek,
        @JsonProperty("week_order") Integer weekOrder,
        @JsonProperty("hour") Integer hour,
        @JsonProperty("minute") Integer minute,
        @JsonProperty("second") Integer second
    ) {

    }

    /**
     * 매년 지정 월/일 시:분:초에 실행.
     *
     * <pre>{@code
     * {
     *   "type": "EVERY_YEAR",
     *   "timezone": "UTC",
     *   "params": {
     *     "months": [1, 6],
     *     "day_of_month": 15,
     *     "hour": 9,
     *     "minute": 15,
     *     "second": 30
     *   }
     * }
     * }</pre>
     */
    record EveryYear(
        @JsonProperty("type") ReportScheduleTypeDtoV1 type,
        @JsonProperty("timezone") String timezone,
        @JsonProperty("params") EveryYearParamsDtoV1 params
    ) implements ReportScheduleDtoV1 {

    }

    /** 매년 지정 월/일 시:분:초에 실행. {@code report_plan.proto}의 {@code EveryYearParamsV1}. */
    record EveryYearParamsDtoV1(
        @JsonProperty("months") List<Integer> months,
        @JsonProperty("day_of_month") Integer dayOfMonth,
        @JsonProperty("hour") Integer hour,
        @JsonProperty("minute") Integer minute,
        @JsonProperty("second") Integer second
    ) {

    }

    /**
     * 지정 간격(초)마다 반복 실행.
     *
     * <pre>{@code
     * {
     *   "type": "PERIODIC",
     *   "timezone": "UTC",
     *   "params": {
     *     "interval_seconds": 3600
     *   }
     * }
     * }</pre>
     */
    record Periodic(
        @JsonProperty("type") ReportScheduleTypeDtoV1 type,
        @JsonProperty("timezone") String timezone,
        @JsonProperty("params") PeriodicParamsDtoV1 params
    ) implements ReportScheduleDtoV1 {

    }

    /** 지정 간격(초)마다 반복 실행. {@code report_plan.proto}의 {@code PeriodicParamsV1}. */
    record PeriodicParamsDtoV1(
        @JsonProperty("interval_seconds") Long intervalSeconds
    ) {

    }

    /**
     * N일마다 지정 시각에 실행.
     *
     * <pre>{@code
     * {
     *   "type": "PERIODIC_DAY",
     *   "timezone": "UTC",
     *   "params": {
     *     "period_days": 2,
     *     "time_of_day": "09:00:00"
     *   }
     * }
     * }</pre>
     */
    record PeriodicDay(
        @JsonProperty("type") ReportScheduleTypeDtoV1 type,
        @JsonProperty("timezone") String timezone,
        @JsonProperty("params") PeriodicDayParamsDtoV1 params
    ) implements ReportScheduleDtoV1 {

    }

    /**
     * N일마다 지정 시각에 실행. {@code report_plan.proto}의 {@code PeriodicDayParamsV1}.
     *
     * <p>{@code time_of_day}는 {@code HH:mm:ss} 형식 문자열이다.
     */
    record PeriodicDayParamsDtoV1(
        @JsonProperty("period_days") Integer periodDays,
        @JsonProperty("time_of_day") String timeOfDay
    ) {

    }
}
```

#### 2. UnitTest

```java
package com.example.dto.scheduler;

import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.params.provider.Arguments.arguments;

import com.example.dto.scheduler.ReportScheduleDtoV1.EveryDayParamsDtoV1;
import com.example.dto.scheduler.ReportScheduleDtoV1.EveryHourParamsDtoV1;
import com.example.dto.scheduler.ReportScheduleDtoV1.EveryMinuteParamsDtoV1;
import com.example.dto.scheduler.ReportScheduleDtoV1.EveryMonthDayParamsDtoV1;
import com.example.dto.scheduler.ReportScheduleDtoV1.EveryMonthWeekParamsDtoV1;
import com.example.dto.scheduler.ReportScheduleDtoV1.EveryWeekParamsDtoV1;
import com.example.dto.scheduler.ReportScheduleDtoV1.EveryYearParamsDtoV1;
import com.example.dto.scheduler.ReportScheduleDtoV1.Immediate;
import com.example.dto.scheduler.ReportScheduleDtoV1.PeriodicDayParamsDtoV1;
import com.example.dto.scheduler.ReportScheduleDtoV1.PeriodicParamsDtoV1;
import com.example.dto.scheduler.ReportScheduleDtoV1.ReportScheduleTypeDtoV1;
import com.example.dto.scheduler.ReportScheduleDtoV1.SpecificTimeParamsDtoV1;
import java.time.DayOfWeek;
import java.util.List;
import java.util.stream.Stream;

import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.Arguments;
import org.junit.jupiter.params.provider.MethodSource;

import tools.jackson.databind.json.JsonMapper;

/**
 * {@link ReportScheduleDtoV1}의 {@code type} 별 다형성 JSON 바인딩
 * (직렬화/역직렬화/round-trip) 검증.
 */
class ReportScheduleDtoV1JsonTest {

    private static final JsonMapper JSON_MAPPER = JsonMapper.builder().build();

    private static Stream<Arguments> scheduleDtos() {
        return Stream.of(
            new Arguments2(
                new Immediate(ReportScheduleTypeDtoV1.IMMEDIATE, "UTC"),
                "{\"type\":\"IMMEDIATE\",\"timezone\":\"UTC\"}"
            ),
            new Arguments2(
                new ReportScheduleDtoV1.SpecificTime(
                    ReportScheduleTypeDtoV1.SPECIFIC_TIME, "UTC",
                    new SpecificTimeParamsDtoV1("2026-08-12T00:00:00+09:00")
                ),
                "{\"type\":\"SPECIFIC_TIME\",\"timezone\":\"UTC\","
                    + "\"params\":{\"date_time\":\"2026-08-12T00:00:00+09:00\"}}"
            ),
            new Arguments2(
                new ReportScheduleDtoV1.EveryMinute(
                    ReportScheduleTypeDtoV1.EVERY_MINUTE, "UTC", new EveryMinuteParamsDtoV1(30)
                ),
                "{\"type\":\"EVERY_MINUTE\",\"timezone\":\"UTC\","
                    + "\"params\":{\"second\":30}}"
            ),
            new Arguments2(
                new ReportScheduleDtoV1.EveryHour(
                    ReportScheduleTypeDtoV1.EVERY_HOUR, "UTC", new EveryHourParamsDtoV1(15, 30)
                ),
                "{\"type\":\"EVERY_HOUR\",\"timezone\":\"UTC\","
                    + "\"params\":{\"minute\":15,\"second\":30}}"
            ),
            new Arguments2(
                new ReportScheduleDtoV1.EveryDay(
                    ReportScheduleTypeDtoV1.EVERY_DAY, "UTC", new EveryDayParamsDtoV1(9, 15, 30)
                ),
                "{\"type\":\"EVERY_DAY\",\"timezone\":\"UTC\","
                    + "\"params\":{\"hour\":9,\"minute\":15,\"second\":30}}"
            ),
            new Arguments2(
                new ReportScheduleDtoV1.EveryWeek(
                    ReportScheduleTypeDtoV1.EVERY_WEEK, "UTC",
                    new EveryWeekParamsDtoV1(List.of(DayOfWeek.MONDAY, DayOfWeek.FRIDAY), 9, 15, 30)
                ),
                "{\"type\":\"EVERY_WEEK\",\"timezone\":\"UTC\","
                    + "\"params\":{\"days_of_week\":[\"MONDAY\",\"FRIDAY\"],"
                    + "\"hour\":9,\"minute\":15,\"second\":30}}"
            ),
            new Arguments2(
                new ReportScheduleDtoV1.EveryMonthDay(
                    ReportScheduleTypeDtoV1.EVERY_MONTH_DAY, "UTC",
                    new EveryMonthDayParamsDtoV1(List.of(1, 15), 9, 15, 30)
                ),
                "{\"type\":\"EVERY_MONTH_DAY\",\"timezone\":\"UTC\","
                    + "\"params\":{\"days_of_month\":[1,15],"
                    + "\"hour\":9,\"minute\":15,\"second\":30}}"
            ),
            new Arguments2(
                new ReportScheduleDtoV1.EveryMonthWeek(
                    ReportScheduleTypeDtoV1.EVERY_MONTH_WEEK, "UTC",
                    new EveryMonthWeekParamsDtoV1(DayOfWeek.MONDAY, 2, 9, 15, 30)
                ),
                "{\"type\":\"EVERY_MONTH_WEEK\",\"timezone\":\"UTC\","
                    + "\"params\":{\"day_of_week\":\"MONDAY\",\"week_order\":2,"
                    + "\"hour\":9,\"minute\":15,\"second\":30}}"
            ),
            new Arguments2(
                new ReportScheduleDtoV1.EveryYear(
                    ReportScheduleTypeDtoV1.EVERY_YEAR, "UTC",
                    new EveryYearParamsDtoV1(List.of(1, 6), 15, 9, 15, 30)
                ),
                "{\"type\":\"EVERY_YEAR\",\"timezone\":\"UTC\","
                    + "\"params\":{\"months\":[1,6],\"day_of_month\":15,"
                    + "\"hour\":9,\"minute\":15,\"second\":30}}"
            ),
            new Arguments2(
                new ReportScheduleDtoV1.Periodic(
                    ReportScheduleTypeDtoV1.PERIODIC, "UTC", new PeriodicParamsDtoV1(3600L)
                ),
                "{\"type\":\"PERIODIC\",\"timezone\":\"UTC\","
                    + "\"params\":{\"interval_seconds\":3600}}"
            ),
            new Arguments2(
                new ReportScheduleDtoV1.PeriodicDay(
                    ReportScheduleTypeDtoV1.PERIODIC_DAY, "UTC",
                    new PeriodicDayParamsDtoV1(2, "09:00:00")
                ),
                "{\"type\":\"PERIODIC_DAY\",\"timezone\":\"UTC\","
                    + "\"params\":{\"period_days\":2,\"time_of_day\":\"09:00:00\"}}"
            )
        ).map(a -> arguments(a.dto, a.expectedJson));
    }

    @ParameterizedTest
    @MethodSource("scheduleDtos")
    void serializesToExpectedSnakeCaseJson(ReportScheduleDtoV1 dto, String expectedJson) {
        String actualJson = JSON_MAPPER.writeValueAsString(dto);

        assertThat(actualJson).isEqualTo(expectedJson);
    }

    @ParameterizedTest
    @MethodSource("scheduleDtos")
    void deserializesToConcreteSubtypeByScheduleType(ReportScheduleDtoV1 dto, String expectedJson) {
        ReportScheduleDtoV1 deserialized = JSON_MAPPER.readValue(expectedJson, ReportScheduleDtoV1.class);

        assertThat(deserialized)
            .isInstanceOf(dto.getClass())
            .isEqualTo(dto);
    }

    @ParameterizedTest
    @MethodSource("scheduleDtos")
    void roundTripsThroughJson(ReportScheduleDtoV1 dto, String expectedJson) {
        String json = JSON_MAPPER.writeValueAsString(dto);
        ReportScheduleDtoV1 roundTripped = JSON_MAPPER.readValue(json, ReportScheduleDtoV1.class);

        assertThat(roundTripped).isEqualTo(dto);
    }

    private record Arguments2(ReportScheduleDtoV1 dto, String expectedJson) {

    }
}
```

#### 3. `instanceof` 없이 다형적으로 읽기: 패턴 매칭 `switch`

`sealed interface` + `record` 조합이 주는 진짜 이점은 인터페이스에 공통 accessor를 몇 개 더 올리는 게 아니라, **`instanceof`/캐스팅 없이 각 서브타입의 값을 안전하게 읽을 수 있는 패턴 매칭 `switch`(Java 21+)**다.

- `sealed`로 서브타입 목록이 컴파일 타임에 확정되므로, `switch`에서 모든 서브타입을 다루면 `default` 절 없이도 컴파일된다 — **exhaustiveness check**. 나중에 서브타입을 추가했는데 어떤 `switch`에서 그 case를 빠뜨리면 그 자리에서 컴파일 에러가 난다.
- 각 `case` 안에서 패턴 변수(`st`, `em` 등)는 이미 해당 서브타입으로 타입이 확정되어 있으므로, `params()`가 그 서브타입 고유의 구체 타입을 그대로 반환한다. 별도의 `instanceof`/캐스팅이 필요 없다.

```java
String describeParams(ReportScheduleDtoV1 dto) {
    return switch (dto) {
        case ReportScheduleDtoV1.Immediate i -> "no params";
        case ReportScheduleDtoV1.SpecificTime st ->
            "date_time=" + st.params().dateTime();
        case ReportScheduleDtoV1.EveryMinute em ->
            "second=" + em.params().second();
        case ReportScheduleDtoV1.EveryHour eh ->
            "minute=" + eh.params().minute() + ", second=" + eh.params().second();
        case ReportScheduleDtoV1.EveryDay ed ->
            "hour=" + ed.params().hour();
        case ReportScheduleDtoV1.EveryWeek ew ->
            "daysOfWeek=" + ew.params().daysOfWeek();
        case ReportScheduleDtoV1.EveryMonthDay emd ->
            "daysOfMonth=" + emd.params().daysOfMonth();
        case ReportScheduleDtoV1.EveryMonthWeek emw ->
            "dayOfWeek=" + emw.params().dayOfWeek() + ", weekOrder=" + emw.params().weekOrder();
        case ReportScheduleDtoV1.EveryYear ey ->
            "months=" + ey.params().months();
        case ReportScheduleDtoV1.Periodic p ->
            "intervalSeconds=" + p.params().intervalSeconds();
        case ReportScheduleDtoV1.PeriodicDay pd ->
            "periodDays=" + pd.params().periodDays();
        // sealed + 모든 case를 다뤘으므로 default 없이 컴파일된다.
    };
}
```

`record`의 컴포넌트까지 한 번에 분해하고 싶다면 record 패턴으로 더 줄일 수도 있다:

```java
case ReportScheduleDtoV1.EveryHour(var scheduleType, var timezone, var params) ->
    "minute=" + params.minute() + ", second=" + params.second();
```

이 방식이 `instanceof`로 타입을 하나씩 확인하는 것보다 나은 이유:

- **안전성**: 캐스팅이 아예 없으므로 `ClassCastException` 여지가 없다.
- **완전성 검사**: 새 서브타입을 추가했는데 처리 로직에서 빠뜨리면 컴파일 시점에 걸러진다(`instanceof` 체인은 이런 보장이 없다).
- **가독성**: `if (dto instanceof SpecificTime st) { ... } else if (...) { ... }`로 체인을 쌓는 것보다 `switch` 한 곳에 모든 분기가 모여 있어 한눈에 들어온다.


### kotlin

#### 1. Base/Sub클래스 및 params 클래스 정의

Java의 `sealed interface`는 컴파일러가 `permits` 목록을 자동 추론하려면 하위 타입들이 같은 파일(또는 명시적 `permits`)에 있어야 하지만, Kotlin의 `sealed class`는 그런 제약이 없다(Kotlin 1.5+부터는 같은 모듈·패키지면 충분). 즉 변동부 `params` 클래스를 별도 파일로 분리할지, Base/Sub 클래스와 한 파일에 둘지는 순전히 가독성·조직화 관점의 선택이다. 아래는 각 Sub 클래스 바로 아래에 그 클래스가 쓰는 `params` 클래스를 함께 선언해 하나의 파일로 합친 예다.

```kotlin
package com.example.schedule

import com.fasterxml.jackson.annotation.JsonSubTypes
import com.fasterxml.jackson.annotation.JsonTypeInfo
import tools.jackson.databind.PropertyNamingStrategies
import tools.jackson.databind.annotation.JsonNaming

enum class ReportScheduleType {
    IMMEDIATE,
    SPECIFIC_TIME,
    EVERY_MINUTE,
    EVERY_HOUR,
    EVERY_DAY,
    EVERY_WEEK,
    EVERY_MONTH_DAY,
    EVERY_MONTH_WEEK,
    EVERY_YEAR,
    PERIODIC,
    PERIODIC_DAY,
}
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.EXISTING_PROPERTY, property = "type")
@JsonSubTypes(
    JsonSubTypes.Type(value = ReportScheduleImmediateDto::class, name = "IMMEDIATE"),
    JsonSubTypes.Type(value = ReportScheduleSpecificTimeDto::class, name = "SPECIFIC_TIME"),
    JsonSubTypes.Type(value = ReportScheduleEveryMinuteDto::class, name = "EVERY_MINUTE"),
    JsonSubTypes.Type(value = ReportScheduleEveryHourDto::class, name = "EVERY_HOUR"),
    JsonSubTypes.Type(value = ReportScheduleEveryDayDto::class, name = "EVERY_DAY"),
    JsonSubTypes.Type(value = ReportScheduleEveryWeekDto::class, name = "EVERY_WEEK"),
    JsonSubTypes.Type(value = ReportScheduleEveryMonthDayDto::class, name = "EVERY_MONTH_DAY"),
    JsonSubTypes.Type(value = ReportScheduleEveryMonthWeekDto::class, name = "EVERY_MONTH_WEEK"),
    JsonSubTypes.Type(value = ReportScheduleEveryYearDto::class, name = "EVERY_YEAR"),
    JsonSubTypes.Type(value = ReportSchedulePeriodicDto::class, name = "PERIODIC"),
    JsonSubTypes.Type(value = ReportSchedulePeriodicDayDto::class, name = "PERIODIC_DAY"),
)
sealed class ReportScheduleDto(open val type: ReportScheduleType)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class ReportScheduleImmediateDto(
    val timezone: String = "",
    override val type: ReportScheduleType = ReportScheduleType.IMMEDIATE,
) : ReportScheduleDto(type)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class ReportScheduleSpecificTimeDto(
    val timezone: String = "",
    val params: SpecificTimeParamsDto,
    override val type: ReportScheduleType = ReportScheduleType.SPECIFIC_TIME,
) : ReportScheduleDto(type)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class SpecificTimeParamsDto(
    val dateTime: Long = 0,
)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class ReportScheduleEveryMinuteDto(
    val timezone: String = "",
    val params: EveryMinuteParamsDto,
    override val type: ReportScheduleType = ReportScheduleType.EVERY_MINUTE,
) : ReportScheduleDto(type)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class EveryMinuteParamsDto(
    val second: Int = 0,
)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class ReportScheduleEveryHourDto(
    val timezone: String = "",
    val params: EveryHourParamsDto,
    override val type: ReportScheduleType = ReportScheduleType.EVERY_HOUR,
) : ReportScheduleDto(type)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class EveryHourParamsDto(
    val minute: Int = 0,
    val second: Int = 0,
)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class ReportScheduleEveryDayDto(
    val timezone: String = "",
    val params: EveryDayParamsDto,
    override val type: ReportScheduleType = ReportScheduleType.EVERY_DAY,
) : ReportScheduleDto(type)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class EveryDayParamsDto(
    val hour: Int = 0,
    val minute: Int = 0,
    val second: Int = 0,
)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class ReportScheduleEveryWeekDto(
    val timezone: String = "",
    val params: EveryWeekParamsDto,
    override val type: ReportScheduleType = ReportScheduleType.EVERY_WEEK,
) : ReportScheduleDto(type)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class EveryWeekParamsDto(
    /** ISO 요일. 1=MONDAY ~ 7=SUNDAY */
    val daysOfWeek: List<Int> = emptyList(),
    val hour: Int = 0,
    val minute: Int = 0,
    val second: Int = 0,
)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class ReportScheduleEveryMonthDayDto(
    val timezone: String = "",
    val params: EveryMonthDayParamsDto,
    override val type: ReportScheduleType = ReportScheduleType.EVERY_MONTH_DAY,
) : ReportScheduleDto(type)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class EveryMonthDayParamsDto(
    val daysOfMonth: List<Int> = emptyList(),
    val hour: Int = 0,
    val minute: Int = 0,
    val second: Int = 0,
)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class ReportScheduleEveryMonthWeekDto(
    val timezone: String = "",
    val params: EveryMonthWeekParamsDto,
    override val type: ReportScheduleType = ReportScheduleType.EVERY_MONTH_WEEK,
) : ReportScheduleDto(type)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class EveryMonthWeekParamsDto(
    /** ISO 요일. 1=MONDAY ~ 7=SUNDAY */
    val dayOfWeek: Int = 0,
    val weekOrder: Int = 0,
    val hour: Int = 0,
    val minute: Int = 0,
    val second: Int = 0,
)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class ReportScheduleEveryYearDto(
    val timezone: String = "",
    val params: EveryYearParamsDto,
    override val type: ReportScheduleType = ReportScheduleType.EVERY_YEAR,
) : ReportScheduleDto(type)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class EveryYearParamsDto(
    val months: List<Int> = emptyList(),
    val dayOfMonth: Int = 0,
    val hour: Int = 0,
    val minute: Int = 0,
    val second: Int = 0,
)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class ReportSchedulePeriodicDto(
    val timezone: String = "",
    val params: PeriodicParamsDto,
    override val type: ReportScheduleType = ReportScheduleType.PERIODIC,
) : ReportScheduleDto(type)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class PeriodicParamsDto(
    val intervalSeconds: Long = 0,
)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class ReportSchedulePeriodicDayDto(
    val timezone: String = "",
    val params: PeriodicDayParamsDto,
    override val type: ReportScheduleType = ReportScheduleType.PERIODIC_DAY,
) : ReportScheduleDto(type)

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy::class)
data class PeriodicDayParamsDto(
    val periodDays: Int = 0,
    val timeOfDay: String = "",
)
```

Base 클래스 정의문의 `open val type`에서 `open`은 이 프로퍼티가 하위 클래스에서 재정의(override) 될 수 있음을 의미한다.

```kotlin
sealed class ReportScheduleDto(open val type: ReportScheduleType)
```

Kotlin에서 클래스와 마찬가지로 프로퍼티도 기본적으로 `final`이라 하위 클래스에서 오버라이드할 수 없다. `type`은 `open`으로 선언되어 있기 때문에, 각 하위 클래스(`ReportScheduleImmediateDto` 등)에서

```kotlin
override val type: ReportScheduleType = ReportScheduleType.IMMEDIATE,
```

처럼 `override`로 재정의해서 자신만의 기본값을 가질 수 있다. 
만약 `open`이 없다면 이 `override` 선언들이 모두 컴파일 에러가 난다.


Base 클래스에 JsonTypeInfo 항목(예제의 `type`)이외에 다른 프로퍼티도 추가 가능하다.

`sealed class` 생성자에 프로퍼티를 추가하면 되고, 각 하위 클래스는 자신의 생성자에서 그 값을 받아 `super(...)` 호출로 넘겨주면 된다.

예를 들어 지금 모든 하위 클래스에 중복돼 있는 `timezone`을 공통 변수로 옮긴다면:

```kotlin
sealed class ReportScheduleDto(
    open val type: ReportScheduleType,
    open val timezone: String = "",
)
```

```kotlin
data class ReportScheduleImmediateDto(
    override val timezone: String = "",
    override val type: ReportScheduleType = ReportScheduleType.IMMEDIATE,
) : ReportScheduleDto(type, timezone)
```

하지만 중복 정의를 피하는 것이 목적이라면 이 방식을 추천 하지 않는다. sub class인 `data class`마다 base class의 프로퍼티를 다시 선언해야 하는 건 상위 클래스에 두든 안 두든 똑같으니, "중복을 줄이는" 목적이라면 상위 클래스에 올릴 이유가 없다.

상위 클래스에 `timezone`을 선언하는 게 의미 있는 유일한 경우는, `ReportScheduleDto` 타입(추상 타입)으로만 들고 있는 코드에서 `when`으로 서브타입을 분기하지 않고 바로 `.timezone`에 접근하고 싶을 때다 (`type`이 그런 이유로 상위에 있는 것과 동일). 그런 다형적 접근이 필요 없다면, 지금처럼 각 하위 클래스에 독립적인 `val timezone`으로 두는 게 더 단순하고 맞는 선택이다.

주의할 점:
- 상위 클래스 프로퍼티도 하위에서 override하려면 `open`으로 선언해야 한다(지금 `type`처럼).
- `data class`는 주 생성자에 있는 프로퍼티만 `equals`/`hashCode`/`copy`에 포함시킨다. 상위 클래스 생성자 파라미터를 `override val`로 하위 `data class` 주 생성자에도 선언해야 그 값이 `copy()`/`equals`에 반영된다. 즉 상위 클래스에만 `val`로 두고 하위에서 override 안 하면 `data class`의 자동 생성 메서드에서 빠진다.
- Jackson 직렬화 순서는 하위 클래스 생성자 프로퍼티 선언 순서를 따르므로, JSON 필드 순서를 맞추려면 각 서브타입에서 선언 순서를 신경 써야 한다.

#### Java `sealed interface`와의 비교: base 타입엔 뭘 올려야 하나

뒤에 나올 Java 예제의 `ReportScheduleDtoV1`은 `scheduleType()`뿐 아니라 `timezone()`까지 인터페이스에 abstract 메서드로 선언한다. Kotlin 예제가 `type`만 상위에 두는 것과 비교하면 원칙은 같다.

- Jackson의 다형성 바인딩은 base 타입(`sealed interface`/`sealed class`)에 어떤 멤버가 선언되어 있는지와 무관하게, concrete record/data class에 붙은 애너테이션만으로 동작한다. 따라서 `timezone()`을 인터페이스에 올리는 것은 JSON 바인딩의 필수 조건이 아니라, 업캐스팅된 참조에서 `switch`/`when`으로 분기하지 않고 바로 접근하기 위한 순수 API 설계 선택이다.
- Kotlin에서 상위 `sealed class`에 프로퍼티를 올리면 하위 `data class`는 주 생성자에 `override val`을 다시 선언하고 `super(...)`로 넘겨야 한다(`data class`의 `copy`/`equals`가 주 생성자 프로퍼티만 보기 때문) — 중복 제거 효과가 없으므로 실익이 있을 때만 올리는 게 맞다.
- Java `record` + `sealed interface`는 인터페이스에 필드를 둘 수 없고 abstract 메서드만 선언 가능하지만, 각 record는 어차피 JSON 계약상 자신의 컴포넌트로 `timezone`을 가지고 있어야 한다. 그래서 인터페이스에 `String timezone();` 시그니처만 추가하면 record가 자동 생성한 accessor가 그대로 계약을 만족시킨다 — 추가 보일러플레이트가 사실상 0이라 편의상 올린 것이다.

그렇다면 같은 논리로 `params`도 인터페이스에 올리면 되지 않을까 싶지만, `params`는 `scheduleType`/`timezone`과 성격이 다르다.

- `params`는 서브타입마다 실제 타입이 다르다(`SpecificTimeParamsDtoV1`, `EveryMinuteParamsDtoV1`, ...). 인터페이스에 올리려면 공통 타입(마커 인터페이스 등)이 필요한데, 결국 호출부는 그 마커 타입을 다시 `instanceof`/패턴 매칭으로 내려찍어야 실제 필드(`hour`, `minute` 등)에 접근할 수 있다 — `scheduleType`으로 분기하는 것과 다를 바 없어서 다형적 접근의 이점이 사라진다.
- `Immediate`는 애초에 `params`가 없다. 인터페이스에 `params()`를 강제하면 `Immediate`도 널이나 빈 마커 타입을 반환하도록 만들어야 해서 불필요한 null 처리/타입이 늘어난다.
- 즉 `scheduleType`/`timezone`은 "모든 서브타입에 항상 존재하고 타입도 동일"하기 때문에 공짜로 올릴 수 있지만, `params`는 "존재 여부도 다르고 타입도 다르다"는 점에서 근본적으로 다르다. 그래서 인터페이스로 올리지 않고 각 record가 자신만의 구체 타입 `params`를 갖도록 둔 것이다.

## Java의 `sealed` + `record` 패턴 조합

### 목적/용도

`sealed`(닫힌 타입 계층)와 `record`(불변 데이터 캐리어)를 함께 쓰면 "이 타입은 정해진 몇 가지 변형(variant) 중 하나이고, 각 변형은 고정된 데이터만 담는다"는 걸 코드로 표현할 수 있다 — 함수형 언어의 대수적 데이터 타입(algebraic data type)과 같은 목적이다. `ReportScheduleDtoV1`처럼 "스케줄 종류는 11가지뿐이고, 각 종류마다 필요한 데이터가 다르다"는 도메인을 표현하는 데 적합하다.

- `sealed`가 없으면: 누구나 새 구현체를 추가할 수 있으므로, 그걸 다루는 코드(`switch`, `if-else` 체인)가 모든 경우를 다뤘는지 컴파일러가 보장해 줄 수 없다.
- `record`가 없으면: 불변 데이터 캐리어를 만들려고 생성자/`equals`/`hashCode`/`toString`/accessor를 매번 손으로 작성해야 한다.
- 둘을 합치면: "닫힌 변형 목록"(`sealed`) × "각 변형의 불변 데이터"(`record`)가 만나 완전성 검사(exhaustiveness check)가 되는 패턴 매칭 `switch`까지 이어진다.

### Java 버전 제약 — 언제부터 정식으로 동작하나

이 조합의 각 요소는 도입/정식화 시점이 다르다.

| 기능 | Preview 도입 | 정식(표준) |
|---|---|---|
| `record` | Java 14 | **Java 16** |
| `sealed` 클래스/인터페이스 | Java 15 | **Java 17** |
| `instanceof` 패턴 매칭 | Java 14 | **Java 16** |
| `switch` 패턴 매칭 + `sealed` exhaustiveness + record 패턴(`case Foo(var a, var b) ->`) | Java 17~20 | **Java 21** |

- `record`와 `sealed` 자체는 Java 17이면 전부 정식 문법으로 쓸 수 있다.
- 하지만 이 문서에서 소개한 **"`instanceof` 없이 `switch`로 완전성 검사까지 받으며 읽기"**는 `switch` 패턴 매칭과 record 패턴이 정식화된 **Java 21(LTS)부터**만 별도 플래그 없이 동작한다. Java 17~20에서 같은 문법을 쓰려면 `--enable-preview`가 필요하고, 버전마다 preview API 형태가 조금씩 달라 실무에선 권장하지 않는다.
- 즉 "sealed + record 선언"은 17부터, "그걸 활용하는 exhaustive switch/record 패턴 읽기"는 21부터라고 구분해서 기억하면 된다.

### 제약 사항

- **닫힌 계층 범위**: `sealed` 타입의 허용된 하위 타입(`permits`, 혹은 같은 파일에서 자동 추론)은 같은 모듈 안에 있어야 한다. 외부 모듈에서 새 서브타입을 추가해 확장하는 구조(플러그인 형태)에는 맞지 않는다.
- **record는 상속 불가**: `record`는 암묵적으로 `final`이고 다른 클래스를 `extends`할 수 없다(오직 인터페이스만 `implements`). 그래서 공통 상태를 상위 클래스에 두는 방식 자체가 불가능하고, 앞서 설명한 것처럼 공통 accessor는 인터페이스의 추상 메서드로만 강제할 수 있다.
- **완전성 검사는 `sealed`에 의존**: 다루는 타입이 `sealed`가 아니라 그냥 `interface`/추상 클래스라면, `switch`에 `default` 없이는 컴파일이 안 되고 컴파일러가 "새 서브타입을 빠뜨렸는지"도 검증해 주지 못한다.
- **낮은 Java 버전과의 호환성**: 사내 서비스가 Java 17 LTS에 머물러 있다면 `sealed`/`record` 선언 자체는 가능하지만, 이 문서의 `switch` 패턴 매칭/record 패턴 예제는 Java 21 이상으로 올려야 프리뷰 플래그 없이 쓸 수 있다.