# Protobuf에서 가변 JSON 문서를 하나의 member로 전달할 때의 선택지

## 한 줄 결론

가변 JSON을 protobuf 메시지의 한 필드로 전달할 때, 보통은 다음 6가지 방식을 고려합니다.

- `oneof`
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
//   "meta": {
//     "source": "mobile"
//   }
// }
```

Java 예시:

```java
import com.google.protobuf.Struct;
import com.google.protobuf.Value;

Struct payload = Struct.newBuilder()
    .putFields("theme", Value.newBuilder().setStringValue("dark").build())
    .putFields("retries", Value.newBuilder().setNumberValue(3).build())
    .putFields("meta", Value.newBuilder().setStructValue(
        Struct.newBuilder()
            .putFields("source", Value.newBuilder().setStringValue("mobile").build())
            .build())
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
