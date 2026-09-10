# Protobuf에서 다형성 JSON 문서를 하나의 member로 전달할 때의 선택지

## 다형성 JSON 예제

다형성 JSON은 key가 동일한 경우(Value만 다형성)와 key/value 모두 변경될 수 있는 경우(key/value로 모두 변환) 두 가지로 나뉜다.

key/value가 모두 변경되는 경우는 보통 `oneof` type으로 정의가 가능하며 여러 type중 하나가 지정될 수 있는 방식이다.

### key가 동일한 다형성 Json 예제 (Value만 다형성)

다음은 key가 `payload`로 고정되며, `payload`의 Value가 가변인 다형성 예제이다.

- type 1

```json
{
  "type": "user_config",
  "payload": {
    "theme": "dark",
    "retries": 3
  }
}
```

- type 2

```json
{
  "type": "user_config",
  "payload": {
    "color": "red",
    "number": 1
  }
}
```

### key/value 모두 가변인 경우(key/value로 모두 변환)

Json문서에 payload1, payload2 중 1개만 포함될 수 있는 다형성 예제이다.

- type 1

```json
{
  "type": "user_config",
  "payload1": {
    "theme": "dark",
    "retries": 3
  }
}
```

- type 2

```json
{
  "type": "user_config",
  "payload2": {
    "color": "red",
    "number": 1
  }
}
```

## 한 줄 결론

> sw comment: GRPC로 Value만 다형성인 Json을 전달 해야 할 때, Value를 stirng으로 전달하고 프로그래밍 적으로 Type 검사를 하는 방식이 가장 유리한 것 같다.

가변 JSON을 protobuf 메시지의 한 필드로 전달할 때, 보통은 다음 6가지 방식을 고려합니다.
이 중 `oneof`는 key/value 모두 변환하는 다형성 케이스에 사용되며 그외에는 value만 다형성인 케이스에 사용됩니다.

- `oneof` : key/value 모두 변환하는 다형성 케이스에 사용
- `string`
- `google.protobuf.Struct`
- `google.protobuf.Value`
- `map<string, google.protobuf.Value>`
- `google.protobuf.Any`

중요한 기준은 다음입니다.

- 값의 종류가 정해져 있는가?
- JSON 문자열 그대로를 넘기려는가?
- 동적 객체 구조를 표현하려는가?
- 타입을 런타임 확장할 수 있게 하려는가?

추가적으로 숫자 표현시 정밀도 문제를 고려해야 합니다.

Struct / Value : 숫자가 내부적으로 number_value (double)로 표현되어 정밀도 이슈가 생길 수 있으므로, 정밀도 계약이 중요한 숫자(큰 ID, 금액, 고정 소수점 값)는  Struct/Value 의 number로 다루지 말아야 합니다.

- 정수는  2^53 - 1 (9007199254740991)까지만 정확합니다. 그보다 크면 정밀도 손실이 납니다.
- 소수는 범위 안이어도(예:  0.1 ) 정확히 표현되지 않는 값이 많아 반올림 오차가 생길 수 있습니다.
- 너무 큰 절대값(약  1.8e308  초과)은  inf 로 가는 범위 문제도 있습니다.

---

## 요약 표

| 방식 | 핵심 아이디어 | 장점 | 단점 | 가장 권장되는 상황 |
|---|---|---|---|---|
| `oneof` | 정해진 타입들 중 하나를 선택 | 타입 안전성, 명확한 계약 | 스키마 확장 비용 큼 | 허용 값이 명확히 정해져 있을 때 |
| `string` | JSON을 문자열로 전달 | 구현 단순 | 파싱/검증 부담 큼 | 원문 JSON이 필요하거나 임시 호환용 |
| `Struct` | JSON 객체를 protobuf로 표현 | 동적 필드 표현이 좋음 | 타입 안전성 약함 | 설정, 메타데이터, 동적 옵션 |
| `Value` | JSON 값 1개를 표현 | 문자열/숫자/배열/객체 가능 | 검증이 느슨함 | 단일 JSON 값 하나를 넘길 때 |
| `map<string, Value>` | 동적 속성 맵 표현 | 자유로운 키-값 확장 | 키 관리 책임 있음 | 속성/설정/플래그 맵 |
| `Any` | 임의의 protobuf 메시지 타입 담기 | 런타임 타입 확장 가능 | 디코딩 복잡, 관리 비용 큼 | 플러그인/확장 인터페이스 |

---

## 공통 기준 JSON 예제

다음의 단순한 다형성 JSON을 기준으로 각 protobuf 타입이 수용할 수 있는 형태를 비교합니다.

```json
{
  "type": "user_config",
  "payload": {
    "theme": "dark",
    "retries": 3
  }
}
```

이 예제는 `payload`가 여러 종류의 설정 중 하나가 될 수 있고, 현재는 `user_config` 타입이 선택된 상황을 가정합니다. 각 방식은 같은 의미를 유지하면서 타입 정보를 표현하는 JSON 구조가 달라집니다.

## 방식별 수신 JSON 예제

### 1. `oneof`

`oneof`는 타입 식별자를 별도의 `type` 필드로 보내기보다, 선택된 protobuf 필드명 자체로 타입을 구분합니다.

> **핵심:** `oneof`의 JSON key는 선택된 필드에 선언한 **message 타입의 이름이 아니라 `oneof` 필드명**과 매핑됩니다.  
> 예를 들어 `UserConfig user_config = 1;`에서 JSON key는 message 이름인 `UserConfig`가 아니라 protobuf 필드명 `userConfig`입니다.

```proto
message UserConfig {
  string theme = 1;
  int32 retries = 2;
}

message SystemConfig {
  bool enabled = 1;
  string region = 2;
}

message DynamicPayload {
  oneof payload {
    UserConfig user_config = 1;
    SystemConfig system_config = 2;
  }
}
```

***`oneof`에서는 message 이름을 JSON key로 사용되지 앖으며,  `oneof`의 변수 이름이 JSON key가 됩니다.***

그러므로 `oneof`는 `body에 다형성 Json` 조건에 대한 대응이라기 보단, `여러 JSON key중 하나만 전송`의 조건이 더 명확합니다..

위 protobuf 메시지에 대응하는 JSON은 다음과 같습니다.

```json
{
  "userConfig": {
    "theme": "dark",
    "retries": 3
  }
}
```

`system_config`을 선택하면 다음과 같이 전송합니다.

```json
{
  "systemConfig": {
    "enabled": true,
    "region": "ap-northeast-2"
  }
}
```

- 장점: 허용 가능한 타입이 JSON 필드명과 protobuf 스키마에 명확히 드러남
- 주의: `oneof`는 JSON의 임의 `type`/`payload` 구조를 자동으로 판별하지 않음
- `protojson`을 사용하면 기본적으로 protobuf 필드명이 lowerCamelCase로 변환됨

### 2. `string`

JSON 전체를 문자열로 감싸서 전달합니다. JSON 구조가 아니라 문자열 값이므로 내부 따옴표를 escape해야 합니다.

```proto
message DynamicPayload {
  string payload_json = 1;
}
```

```json
{
  "payloadJson": "{\"type\":\"user_config\",\"payload\":{\"theme\":\"dark\",\"retries\":3}}"
}
```

- 장점: 원문 JSON을 그대로 보존하고 수신할 수 있음
- 주의: gRPC/protobuf 레이어에서는 내부 JSON의 타입이나 유효성을 검증하지 않음
- 수신 후 애플리케이션에서 다시 JSON 파싱 및 `type` 검증이 필요함

### 3. `google.protobuf.Struct`

`Struct`는 JSON 객체 전체를 그대로 표현합니다. 객체 내부의 `type`과 `payload`가 일반적인 JSON 필드로 수신됩니다.

```proto
import "google/protobuf/struct.proto";

message DynamicPayload {
  google.protobuf.Struct payload = 1;
}
```

```json
{
  "payload": {
    "type": "user_config",
    "payload": {
      "theme": "dark",
      "retries": 3
    }
  }
}
```

- 장점: 중첩 객체와 동적 필드를 자연스럽게 수용함
- 주의: `type` 값에 따른 하위 구조 검증은 애플리케이션 책임임
- 생성 방식 주의: 정적 DTO처럼 자동 매핑되는 구조가 아니라, 보통 `putFields(key, Value...)` 형태로 key/value를 조립하는 코드(또는 공통 헬퍼 메서드)가 필요함
- 숫자는 내부적으로 `google.protobuf.Value.number_value`의 `double`로 표현되므로 정밀도가 중요한 값에는 사용하지 않음
- 객체가 항상 최상위 값이라면 `Struct`가 `Value`보다 의도를 명확하게 표현함

#### `Struct`를 사용하지 말아야 하는 경우

`Struct`의 숫자 값은 임의 정밀도의 정수가 아니라 IEEE 754 배정밀도 부동소수점(`double`)으로 저장됩니다. 따라서 숫자의 유효 자릿수가 약 15~16자리를 넘어가거나, 정수가 `2^53 - 1`을 초과하면 원래 값과 다르게 변환될 수 있습니다.

```text
안전하게 정확히 표현 가능한 최대 정수:
9,007,199,254,740,991 (= 2^53 - 1)
```

예를 들어 다음과 같은 값은 `Struct`에 숫자로 넣지 않는 것이 안전합니다.

```json
{
  "user_id": 9007199254740993,
  "amount": 123456789012345.67,
  "order_number": 20260910111200123456
}
```

- **대규모 정수 ID/식별자**: DB의 `BIGINT`, snowflake ID, 주문번호, 계좌번호 등은 `string`으로 전달
- **금액/정밀 소수**: 원 단위 또는 decimal scale이 중요한 금액은 정수 최소 단위(`int64`)나 문자열로 전달
- **고정밀 측정값**: 나노초 타임스탬프, 과학/통계 계산값처럼 유효 자릿수가 많은 값은 전용 protobuf 타입이나 문자열 사용
- **정확한 소수점 자릿수가 필요한 값**: `0.1`처럼 부동소수점으로 정확히 표현되지 않는 값은 `Struct`의 숫자만으로 정확성을 보장하지 않음

다만 값이 화면 표시용이거나 약간의 반올림 오차를 허용할 수 있는 일반적인 수치라면 `Struct`를 사용할 수 있습니다. 숫자 정밀도가 계약의 일부라면 `Struct` 대신 명시적인 protobuf 필드(`int64`, `fixed64`, `sint64`, 별도 decimal 메시지 등)를 정의하는 것이 권장됩니다.

### 4. `google.protobuf.Value`

`Value`는 객체뿐 아니라 문자열, 숫자, 배열, 불리언, null까지 하나의 JSON 값으로 수신할 수 있습니다.

```proto
import "google/protobuf/struct.proto";

message DynamicPayload {
  google.protobuf.Value payload = 1;
}
```

객체를 전달하는 경우 JSON은 다음과 같습니다.

```json
{
  "payload": {
    "type": "user_config",
    "payload": {
      "theme": "dark",
      "retries": 3
    }
  }
}
```

같은 필드로 단순 문자열이나 배열도 수신할 수 있습니다.

```json
{
  "payload": "maintenance"
}
```

```json
{
  "payload": [
    "dark",
    3
  ]
}
```

- 장점: payload의 최상위 JSON 형태가 런타임에 달라질 수 있음
- 주의: 객체만 허용하려는 계약이라면 `Struct`가 더 명확함

### 5. `map<string, google.protobuf.Value>`

동적 속성을 메시지의 최상위 map 항목으로 펼쳐서 수신합니다. 공통 JSON 예제의 `payload` 내부 필드를 map으로 표현하는 형태입니다.

```proto
import "google/protobuf/struct.proto";

message DynamicPayload {
  map<string, google.protobuf.Value> attributes = 1;
}
```

```json
{
  "attributes": {
    "type": "user_config",
    "theme": "dark",
    "retries": 3
  }
}
```

중첩 객체도 map 값으로 수신할 수 있습니다.

```json
{
  "attributes": {
    "type": "user_config",
    "options": {
      "theme": "dark",
      "retries": 3
    }
  }
}
```

- 장점: 속성 집합이 동적으로 늘어나는 JSON에 적합함
- 주의: `type`과 실제 속성이 같은 레벨에 놓이므로 payload 경계가 필요한 경우 `Struct` 또는 `Value`가 더 적절함
- map의 모든 값은 `Value`로 변환되므로 타입 검증은 애플리케이션에서 수행해야 함

### 6. `google.protobuf.Any`

`Any`는 JSON 객체의 임의 필드를 추론하는 방식이 아니라, 실제 protobuf 메시지 타입을 `@type`으로 식별해 수신합니다.

```proto
import "google/protobuf/any.proto";

message UserConfig {
  string theme = 1;
  int32 retries = 2;
}

message SystemConfig {
  bool enabled = 1;
  string region = 2;
}

message DynamicPayload {
  google.protobuf.Any payload = 1;
}
```

`UserConfig`를 담은 `Any`의 protobuf JSON 표현은 다음과 같습니다.

```json
{
  "payload": {
    "@type": "type.googleapis.com/example.UserConfig",
    "theme": "dark",
    "retries": 3
  }
}
```

`SystemConfig`를 담으면 `@type`과 payload 필드가 달라집니다.

```json
{
  "payload": {
    "@type": "type.googleapis.com/example.SystemConfig",
    "enabled": true,
    "region": "ap-northeast-2"
  }
}
```

- 장점: 타입이 protobuf 메시지로 정의되어 타입 안전성과 확장성을 함께 확보할 수 있음
- 주의: 수신 측이 type URL에 해당하는 protobuf 타입을 알고 있어야 함
- 임의 JSON을 그대로 담는 타입이 필요하면 `Any` 안에 별도의 `Struct` 메시지를 넣는 설계를 고려해야 함
- 숫자 정밀도 주의: `Any` 자체가 정밀도를 깨뜨리지는 않지만, 내부 메시지가 `Struct`/`Value`의 `number_value(double)`를 쓰거나 JSON을 JS `number`로 파싱하면 큰 정수(`int64/uint64`)에서 손실이 발생할 수 있음  
  (ProtoJSON에서는 보통 `int64/uint64`를 문자열로 표현해 이를 회피함)

## 같은 의미를 표현하는 JSON 구조 비교

위의 동일한 설정 내용을 각 방식으로 표현하면 다음과 같습니다.

| 방식 | 수신 JSON |
|---|---|
| `oneof` | `{ "userConfig": { "theme": "dark", "retries": 3 } }` |
| `string` | `{ "payloadJson": "{\"type\":\"user_config\",\"payload\":{\"theme\":\"dark\",\"retries\":3}}" }` |
| `Struct` | `{ "payload": { "type": "user_config", "payload": { "theme": "dark", "retries": 3 } } }` |
| `Value` | `{ "payload": { "type": "user_config", "payload": { "theme": "dark", "retries": 3 } } }` |
| `map<string, Value>` | `{ "attributes": { "type": "user_config", "theme": "dark", "retries": 3 } }` |
| `Any` | `{ "payload": { "@type": "type.googleapis.com/example.UserConfig", "theme": "dark", "retries": 3 } }` |

`Struct`와 `Value`의 예제 JSON은 객체 형태일 때 동일해 보이지만, `Value`는 최상위 값으로 문자열/숫자/배열/null도 허용한다는 점이 다릅니다. `Any`는 애플리케이션 임의 문자열인 `type`이 아니라 protobuf type URL을 사용합니다.

---

## 1. oneof 사용

### 개념

`oneof`는 메시지 안의 여러 필드 중 하나만 실제 값으로 들어가도록 정의하는 문법입니다.

즉, 값의 종류가 이미 정해져 있을 때 가장 잘 맞습니다.

### 예제

```proto
syntax = "proto3";

message UserConfig {
  string theme = 1;
  int32 retries = 2;
}

message SystemConfig {
  bool enabled = 1;
  string region = 2;
}

message DynamicPayload {
  oneof payload {
    string raw_json = 1;
    UserConfig user_config = 2;
    SystemConfig system_config = 3;
  }
}
```

### 장점

- 타입 안전성 높음
- API 계약이 명확함
- 서버 로직 분기 처리가 쉬움
- 문서화와 협업이 좋음

### 단점

- 새 타입을 추가하면 `.proto` 수정이 필요함
- 실제로 계속 변하는 JSON 객체 전체는 표현하기 적합하지 않음
- 타입이 많아지면 관리가 복잡해짐

### 권장 상황

- 값 종류가 이미 정해져 있음
- `create`, `update`, `delete` 같은 명확한 대안이 있음
- 서버가 각 타입별로 다른 로직을 수행해야 함

### 추천도

높음. 정해진 대안 중 하나를 표현할 때 가장 좋습니다.

---

## 2. string 사용

### 개념

JSON 문서를 문자열 그대로 전달하는 방식입니다.

```proto
syntax = "proto3";

message Request {
  string payload_json = 1;
}
```

### 예제

```json
{
  "payload_json": "{\"theme\":\"dark\",\"retries\":3}"
}
```

Java 예시:

```java
Request request = Request.newBuilder()
    .setPayloadJson("{\"theme\":\"dark\",\"retries\":3}")
    .build();
```

### 장점

- 구현이 가장 단순함
- JSON 원문을 그대로 전달 가능
- 외부 시스템/게이트웨이와 쉽게 연동 가능
- 스키마 제약이 거의 없음

### 단점

- 타입 검증이 불가함
- 잘못된 JSON은 런타임에서 실패할 수 있음
- 파싱/검증 코드가 누적됨
- 프로토콜 수준에서 의미 설명이 약함

### 권장 상황

- 외부 시스템이 이미 JSON 문자열을 보낼 때
- 임시 인터페이스나 호환성 유지용 통신
- 로그/감사/원문 추적용 데이터
- 내부 로직에서 별도 파싱을 명시적으로 관리할 때

### 추천도

중간. 빠르게 구현해야 할 때는 좋지만, 장기적으로는 검증 부담이 큽니다.

---

## 3. google.protobuf.Struct 사용

### 개념

`google.protobuf.Struct`는 JSON 객체를 protobuf로 자연스럽게 표현하는 표준 타입입니다.

```proto
syntax = "proto3";

import "google/protobuf/struct.proto";

message Request {
  google.protobuf.Struct payload = 1;
}
```

### 예제

```proto
// payload = {
//   "theme": "dark",
//   "retries": 3,
//   "enabled": true,
//   "meta": {
//     "source": "mobile"
//   },
//   "tags": ["a", "b"]
// }
```

Java 예시:

```java
import com.google.protobuf.ListValue;
import com.google.protobuf.Struct;
import com.google.protobuf.Value;

Struct payload = Struct.newBuilder()
    .putFields("theme", Value.newBuilder().setStringValue("dark").build())
    .putFields("retries", Value.newBuilder().setNumberValue(3).build())
    .putFields("enabled", Value.newBuilder().setBoolValue(true).build())
    .putFields("meta", Value.newBuilder().setStructValue(
        Struct.newBuilder()
            .putFields("source", Value.newBuilder().setStringValue("mobile").build())
            .build()).build())
    .putFields("tags", Value.newBuilder().setListValue(
        ListValue.newBuilder()
            .addValues(Value.newBuilder().setStringValue("a").build())
            .addValues(Value.newBuilder().setStringValue("b").build())
            .build()).build())
    .build();

Request request = Request.newBuilder()
    .setPayload(payload)
    .build();
```

### 장점

- 동적 JSON 객체를 가장 자연스럽게 표현 가능
- 키-값이 자유로운 설정/메타데이터에 적합
- JSON 구조를 유지하면서 gRPC 메시지로 보낼 수 있음
- 스키마를 고정하지 않고 유연하게 확장 가능

### 단점

- 타입 안전성이 약함
- 값 검증 부족 시 런타임 오류 가능
- `Struct`는 JSON 값 타입으로만 제한되므로 엄밀한 typed 모델 설계가 어려움
- 정적 모델처럼 필드 자동 생성/매핑이 되지 않아 key/value 조립 로직(또는 변환 유틸)을 별도로 관리해야 함
- 숫자는 보통 double 기반으로 다뤄져 정밀도 이슈가 있을 수 있음

### 권장 상황

- 설정값, 옵션, 플래그, 메타데이터
- 동적으로 확장되는 필드가 많음
- 사용자/운영자가 자유롭게 값을 넣어야 함

### 추천도

높음. 동적 JSON 객체를 표현할 때 가장 표준적입니다.

---

## 4. google.protobuf.Value 사용

### 개념

`google.protobuf.Value`는 JSON의 값 하나를 표현하는 타입입니다.

즉, 객체, 배열, 문자열, 숫자, bool, null까지 모두 하나의 값으로 담을 수 있습니다.

```proto
syntax = "proto3";

import "google/protobuf/struct.proto";

message Request {
  google.protobuf.Value payload = 1;
}
```

### 예제

```proto
// payload 값 예시
// "hello"
// 42
// true
// [1, 2, 3]
// { "theme": "dark", "retries": 3 }
```

Java 예시:

```java
import com.google.protobuf.Value;
import com.google.protobuf.Struct;

Value payload = Value.newBuilder()
    .setStructValue(
        Struct.newBuilder()
            .putFields("theme", Value.newBuilder().setStringValue("dark").build())
            .putFields("retries", Value.newBuilder().setNumberValue(3).build())
            .build())
    .build();

Request request = Request.newBuilder()
    .setPayload(payload)
    .build();
```

### 장점

- JSON 값 하나를 가장 범용적으로 표현 가능
- 문자열, 숫자, 객체, 배열 모두 허용
- `Struct`보다 더 일반적인 JSON scalar/object/array 표현에 가깝다

### 단점

- 타입 안전성이 약함
- 값 검증이 느슨해서 잘못된 값이 들어올 수 있음
- 값의 의미를 코드에서 파악하려면 추가적인 타입 체크가 필요함

### 권장 상황

- 단일 JSON 값 하나를 넘기고 싶을 때
- 값이 문자열, 숫자, 배열, 객체 중 어떤 형태가 될지 런타임에 결정될 때

### 추천도

중상. `Struct`보다 더 범용적이지만, 의미를 엄격하게 제한하기는 어렵습니다.

---

## 5. map<string, google.protobuf.Value> 사용

### 개념

`map<string, Value>`는 동적인 속성 키-값 맵을 표현할 때 매우 편리합니다.

```proto
syntax = "proto3";

import "google/protobuf/struct.proto";

message Request {
  map<string, google.protobuf.Value> attributes = 1;
}
```

### 예제

```proto
// attributes = {
//   "theme": "dark",
//   "retries": 3,
//   "enabled": true,
//   "meta": { "source": "mobile" }
// }
```

Java 예시:

```java
import com.google.protobuf.Value;
import com.google.protobuf.Struct;

Map<String, Value> attrs = new HashMap<>();
attrs.put("theme", Value.newBuilder().setStringValue("dark").build());
attrs.put("retries", Value.newBuilder().setNumberValue(3).build());
attrs.put("enabled", Value.newBuilder().setBoolValue(true).build());

Request request = Request.newBuilder()
    .putAllAttributes(attrs)
    .build();
```

### 장점

- 동적 속성을 매우 자연스럽게 표현 가능
- 사용자/설정/플래그/메타데이터에 적합
- 개별 필드가 아니라 속성 맵으로 확장하기 좋음

### 단점

- 키 문자열 관리가 필요함
- 이름 충돌/오타/스펙 미준수 문제가 생길 수 있음
- 의미의 엄격성은 낮음

### 권장 상황

- 설정값, 사용자 프로필 속성, 플래그 집합, 태그 메타데이터
- 필드명이 미리 정해지지 않고 동적으로 늘어나는 상황

### 추천도

중상. 동적 속성 집합을 관리할 때 매우 유용합니다.

---

## 6. google.protobuf.Any 사용

### 개념

`google.protobuf.Any`는 임의의 protobuf 메시지 타입을 담을 수 있는 타입입니다.

즉, "값의 타입이 런타임에 결정된다"는 상황에 적합합니다.

```proto
syntax = "proto3";

import "google/protobuf/any.proto";

message Request {
  google.protobuf.Any payload = 1;
}
```

### 예제

```proto
message UserConfig {
  string theme = 1;
  int32 retries = 2;
}

message SystemConfig {
  bool enabled = 1;
  string region = 2;
}
```

실제 전송 시에는 어떤 메시지 타입이 들어오든 `Any`로 보관할 수 있습니다.

### 장점

- 매우 확장 가능한 타입 설계
- 메시지 타입을 미리 정하지 않고 런타임에 유연하게 처리 가능
- 플러그인/모듈형 아키텍처에 적합

### 단점

- 타입 디코딩이 복잡함
- 타입 URL, 타입 등록, 타입 체크 로직이 필요함
- JSON처럼 자유롭지만, 디버깅과 유지보수가 어려움

### 권장 상황

- 동적으로 확장되는 서비스 인터페이스
- 플러그인 구조 또는 확장형 모듈 설계
- "오늘은 A, 내일은 B"와 같이 타입이 실행 시간에 바뀌는 경우

### 추천도

중상. 매우 유연하지만, 타입 인식/등록 로직을 잘 설계해야 합니다.

---

## 어떤 방식을 선택해야 할까?

### 1) 타입이 정해져 있다면: `oneof`

- `UserConfig`/`SystemConfig`처럼 허용 대안이 명확할 때
- 서버가 각 타입마다 다른 로직을 수행할 때
- API 계약이 엄격해야 할 때

### 2) 단순히 JSON 문자열이 필요하다면: `string`

- 외부 시스템과의 임시 통합
- 로그/추적/원문 전달
- 빠른 프로토타입 개발

### 3) 동적인 JSON 객체가 필요하다면: `Struct`

- 설정값, 옵션, 메타데이터
- 사용자 수준 옵션
- 동적으로 추가/삭제되는 필드

### 4) 값 하나를 자유롭게 넘기고 싶다면: `Value`

- 문자열, 숫자, 배열, 객체가 모두 허용될 때
- “하나의 JSON 값” 자체를 전달할 때

### 5) 속성 집합이 자유롭다면: `map<string, Value>`

- 속성/플래그/태그/설정 맵
- 필드명이 불안정하고 확장 가능해야 할 때

### 6) 타입 자체를 런타임에서 확장해야 한다면: `Any`

- 플러그인, 확장 모듈
- 타입이 여러 가지로 급변하는 인터페이스

---

## 실무 권장 가이드

어떤 값을 전달해야 할지 결정할 때는 다음 기준을 사용합니다.

> **동일한 JSON `key`에 대해서 value가 다형성인 가변 구조 조거에서 value를 실제 JSON 값으로 다뤄야 하면 `Value`, JSON 객체만 필요하면 `Struct`, 단순 원문 전달이면 `string`을 선택하면 됩니다.**

- 명확한 대안이 있으면 `oneof`를 선택
- JSON 원문 그대로 전달이 필요하면 `string` 사용
- 동적 객체 구조라면 `Struct` 사용
- 단일 값 하나를 담고 싶으면 `Value` 사용
- 속성 중심 데이터라면 `map<string, Value>` 사용
- 타입 확장이 핵심이라면 `Any` 사용

---

## 최종 결론

가변 JSON을 protobuf 메시지의 한 member로 다룰 때, 가장 기본적인 원칙은 다음입니다.

- 정해진 대안만 허용한다면 `oneof`
- 원문 JSON 문자열이 필요하다면 `string`
- 동적 JSON 객체가 필요하다면 `Struct`
- 단일 JSON 값이 필요하다면 `Value`
- 속성 맵이 필요하다면 `map<string, Value>`
- 런타임 타입 확장이 핵심이라면 `Any`

즉, `oneof`는 계약 중심 설계, `string`은 빠른 적응성, `Struct`는 동적 유연성, `Value`/`map`은 JSON 데이터 중심 설계, `Any`는 확장형 설계에 각각 강점이 있습니다.

프로젝트 요구사항이 "안정성"인지, "호환성"인지, "유연성"인지에 따라 선택하면 됩니다.
