# Jackson annotation (JSON DTO/VO 구현 시 사용)

- [Jackson annotation (JSON DTO/VO 구현 시 사용)](#jackson-annotation-json-dtovo-구현-시-사용)
  - [개요](#개요)
  - [주요 어노테이션](#주요-어노테이션)
    - [`@JsonInclude(Include.NON_NULL)`](#jsonincludeincludenon_null)
    - [`@JsonIgnore`](#jsonignore)
    - [`@JsonIgnoreProperties`](#jsonignoreproperties)
    - [`@JsonIgnoreProperties(ignoreUnknown = true)`](#jsonignorepropertiesignoreunknown--true)
    - [`@JsonIgnoreType`](#jsonignoretype)
    - [`@JsonFormat`](#jsonformat)
  - [Json Mapping용 class 선언 기본형](#json-mapping용-class-선언-기본형)
  - [Spring Web에서 `@RequestBody` 자동 매핑](#spring-web에서-requestbody-자동-매핑)
  - [DTO 정의 시 자주 쓰는 어노테이션](#dto-정의-시-자주-쓰는-어노테이션)
  - [`@JsonNaming`과 `@JsonProperty`](#jsonnaming과-jsonproperty)
    - [Case Naming Convention](#case-naming-convention)
    - [`@JsonNaming(<PropertyNamingStrategies.XXX.class>)`](#jsonnamingpropertynamingstrategiesxxxclass)
    - [`@JsonProperty("<Json Field Name>")`](#jsonpropertyjson-field-name)
    - [`@JsonNaming`과 `@JsonProperty` 혼용](#jsonnaming과-jsonproperty-혼용)
  - [snake\_case(JSON) ↔ lowerCamelCase(Java) 매핑 예시](#snake_casejson--lowercamelcasejava-매핑-예시)

## 개요

- Jackson 어노테이션은 Java DTO/VO와 JSON 간 직렬화/역직렬화 규칙을 선언적으로 정의할 때 사용한다.
- 핵심 사용 목적은 다음과 같다.
  - 포함/제외 필드 제어
  - 필드명 매핑 규칙 제어
  - 날짜/시간 포맷 제어
  - 예상치 못한 필드 수신 시 예외 처리 방식 제어

## 주요 어노테이션

### `@JsonInclude(Include.NON_NULL)`

- 대상: class
- 목적: 직렬화(객체 -> JSON) 시 `null` 필드를 JSON에 포함하지 않음
- 효과: API 응답 payload를 줄이고, 클라이언트가 불필요한 `null` 필드를 처리하지 않게 함
- 주요 값:
  - `ALWAYS`: 기본값, `null`도 포함
  - `NON_NULL`: `null`이 아닌 필드만 포함
  - `NON_ABSENT`
  - `NON_EMPTY`: 비어 있지 않은 필드만 포함
  - `NON_DEFAULT`
  - `CUSTOM`
  - `USE_DEFAULTS`

예시:

```java
import com.fasterxml.jackson.annotation.JsonInclude;
import lombok.Data;

@Data
@JsonInclude(JsonInclude.Include.NON_NULL)
public class UserProfileResponse {
  private String userId;
  private String nickname;
  private String phoneNumber; // null이면 응답에서 제외
}
```

`phoneNumber == null`이면 JSON 출력에 `phoneNumber` key가 포함되지 않는다.

### `@JsonIgnore`

- 대상: property(field/getter)
- 목적: 지정한 property를 직렬화/역직렬화 대상에서 제외
- 사용 예: 비밀번호, 내부 계산용 값, 외부 노출 금지 필드

예시:

```java
import com.fasterxml.jackson.annotation.JsonIgnore;
import lombok.Data;

@Data
public class UserAccount {
  private String username;

  @JsonIgnore
  private String password;
}
```

`password`는 JSON으로 내보낼 때도 제외되고, JSON 입력을 받을 때도 매핑되지 않는다.

### `@JsonIgnoreProperties`

- 대상: class
- 목적: 지정한 JSON property를 직렬화/역직렬화 대상에서 제외
- 예: `@JsonIgnoreProperties({ "property1", "property2" })`
- 사용 예: 외부 API 응답 중 현재 서비스에서 사용하지 않는 필드 제거

예시:

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import lombok.Data;

@Data
@JsonIgnoreProperties({ "debug", "internal_note" })
public class ProductResponse {
  private String productId;
  private String productName;
}
```

입력 JSON에 `debug`, `internal_note`가 있어도 `ProductResponse`에는 매핑하지 않는다.

### `@JsonIgnoreProperties(ignoreUnknown = true)`

- 대상: class
- 목적: DTO에 없는 JSON 필드가 들어와도 예외를 던지지 않고 무시
- 사용 예: 외부 API가 필드를 추가해도 기존 클라이언트 역직렬화를 깨지 않게 유지

예시:

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import lombok.Data;

@Data
@JsonIgnoreProperties(ignoreUnknown = true)
public class WeatherResponse {
  private String city;
  private Double temperature;
}
```

JSON에 `humidity`, `air_quality` 같은 새 필드가 추가되어도 실패하지 않는다.

### `@JsonIgnoreType`

- 대상: class
- 목적: 특정 타입 자체를 (역)직렬화 대상에서 제외

예시:

```java
import com.fasterxml.jackson.annotation.JsonIgnoreType;
import lombok.Data;

@JsonIgnoreType
@Data
public class InternalAudit {
  private String traceId;
  private String createdBy;
}

@Data
public class OrderResponse {
  private String orderId;
  private InternalAudit audit; // JSON에서 전체 제외
}
```

`OrderResponse`를 직렬화해도 `audit` 전체가 출력되지 않는다.

### `@JsonFormat`

- 대상: field / getter
- 목적: 날짜/시간, 숫자, enum 등의 JSON 표현 형식을 명시적으로 고정
- 사용 상황:
  - 기본 timestamp 숫자 대신 문자열 포맷으로 내려야 할 때
  - 외부 API 스펙과 날짜/시간 형식을 정확히 맞춰야 할 때

주요 속성:

- `shape`: JSON 타입 힌트 (`STRING`, `NUMBER`, `ARRAY` 등)
- `pattern`: 날짜/시간 출력 패턴 (예: `yyyy-MM-dd HH:mm:ss`)
- `timezone`: 직렬화/역직렬화 시 적용 타임존 (예: `Asia/Seoul`)
- `locale`: 로케일 의존 문자열 제어

추가 예시) enum을 문자열로 직렬화

```java
import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;

@Data
public class JobResponse {

  @JsonFormat(shape = JsonFormat.Shape.STRING)
  private Status status;

  public enum Status {
    READY, RUNNING, DONE
  }
}
```

`status`는 숫자 인덱스가 아니라 `"READY"`, `"RUNNING"` 형태 문자열로 직렬화된다.

예시 1) `Date`를 문자열 포맷으로 고정

```java
import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;

import java.util.Date;

@Data
public class AuditResponse {

  @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Seoul")
  private Date createdAt;
}
```

출력 예시:

```json
{
  "created_at": "2026-10-16 08:30:00"
}
```

예시 2) `LocalDateTime` 패턴 지정

```java
import com.fasterxml.jackson.annotation.JsonFormat;
import lombok.Data;

import java.time.LocalDateTime;

@Data
public class EventDto {

  @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd'T'HH:mm:ss")
  private LocalDateTime startedAt;
}
```

주의:

- `LocalDateTime`은 타임존 정보가 없으므로, 타임존이 필요한 API에서는 `OffsetDateTime`/`ZonedDateTime`을 고려한다.
- `pattern`은 직렬화와 역직렬화 모두에 영향을 준다. 입력 JSON 포맷도 동일해야 한다.

## Json Mapping용 class 선언 기본형

```java
package com.example.httpclient.model;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.annotation.JsonInclude.Include;
import lombok.Data;

@Data
@JsonInclude(Include.NON_NULL) // serialize
@JsonIgnoreProperties(ignoreUnknown = true) // deserialize
public class ApiResponseHeader {
    Boolean isSuccessful;
    Integer resultCode;
    String resultMessage;
}
```

## Spring Web에서 `@RequestBody` 자동 매핑

```java
import lombok.Data;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@Data
@RestController
public class UserController {

    public static class User {
        private String id;
        private String name;
    }

    @PostMapping("/users")
    public String createUser(@RequestBody User user) {
        return "User created: " + user.getName();
    }
}
```

HTTP POST 요청 body의 JSON 데이터 `{"id":"123","name":"John Doe"}`는 `createUser` 메서드의 `user` 객체로 매핑된다.

## DTO 정의 시 자주 쓰는 어노테이션

- `@Data`
  - package: `lombok.Data`
- `@Json...`
  - package: `com.fasterxml.jackson.annotation.*`

## `@JsonNaming`과 `@JsonProperty`

- Jackson이 Java DTO를 JSON 문서와 매핑할 때, 필드가 매핑될 JSON key 이름을 제어하는 데 사용한다.
- `@JsonNaming`은 클래스 전체 필드에 naming convention을 일괄 적용한다.
- `@JsonProperty`는 특정 필드의 JSON key 이름을 개별 지정한다.

### Case Naming Convention

- `snake_case`
- `UPPER_SNAKE_CASE`
- `lowerCamelCase`
- `UpperCamelCase` (PascalCase)
- `lowercase`
- `kebab-case`
- `lower.dot.case`

### `@JsonNaming(<PropertyNamingStrategies.XXX.class>)`

- 참고: [PropertyNamingStrategies](https://javadoc.io/static/com.fasterxml.jackson.core/jackson-databind/2.21.0/com/fasterxml/jackson/databind/PropertyNamingStrategies.html)

사용 예:

```java
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class MyDTO { ... }
```

대상: class  
설명: Java DTO 필드가 매핑될 JSON key naming 규칙을 지정한다.

value로 사용할 수 있는 naming strategy class:

| Class 명 | Case Naming Convention Example |
|:---|:---|
| `PropertyNamingStrategy.LowerCamelCaseStrategy.class` | `lowerCamelCase` |
| `PropertyNamingStrategy.UpperCamelCaseStrategy.class` | `UpperCamelCase` (PascalCase) |
| `PropertyNamingStrategy.SnakeCaseStrategy.class` | `snake_case` |
| `PropertyNamingStrategy.UpperSnakeCaseStrategy.class` | `UPPER_SNAKE_CASE` |
| `PropertyNamingStrategy.LowerCaseStrategy.class` | `lowercase` |
| `PropertyNamingStrategy.KebabCaseStrategy.class` | `kebab-case` |
| `PropertyNamingStrategy.LowerDotCaseStrategy.class` | `lower.dot.case` |

### `@JsonProperty("<Json Field Name>")`

대상: field(멤버 변수)  
설명: Java DTO 필드가 매핑될 JSON key 이름을 직접 지정한다.

### `@JsonNaming`과 `@JsonProperty` 혼용

- 혼용 시 `@JsonProperty`로 지정한 필드명이 최종 JSON key로 적용된다.
- 처리 순서상 `@JsonNaming` 적용 후 `@JsonProperty`가 override한다.

## snake_case(JSON) ↔ lowerCamelCase(Java) 매핑 예시

- 방법 1: `@JsonNaming`으로 DTO 전체 필드명 변환

Java object:

```java
package com.dto.test.model;

import java.util.Date;
import com.fasterxml.jackson.databind.PropertyNamingStrategies;
import com.fasterxml.jackson.databind.annotation.JsonNaming;
import lombok.Data;

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
@Data
public class NodeHW {

  private Integer nodeId;
  private String biosName;

  // @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm a z")
  private Date biosDate;

  private Boolean updateStatus;
}
```

- 방법 2: `@JsonProperty`로 필드별 이름 지정

JSON data:

```json
{
  "node_id": 1,
  "bios_name": "mybios",
  "bios_date": 1704690914681,
  "update_status": true
}
```

```java
package com.dto.test.model;

import java.util.Date;
import com.fasterxml.jackson.annotation.JsonProperty;
import lombok.Data;

@Data
public class NodeHW {

  @JsonProperty("node_id")
  private Integer nodeId;

  @JsonProperty("bios_name")
  private String biosName;

  @JsonProperty("bios_date")
  // @JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm a z")
  private Date biosDate;

  @JsonProperty("update_status")
  private Boolean updateStatus;
}
```
