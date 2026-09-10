# grpc data types

## 구분

gRPC의 Protocol Buffers(proto3)에서 사용할 수 있는 데이터 타입은 크게 스칼라(Scalar), 복합(Composite), 특수(Special) 타입으로 나뉩니다.

## 1. 스칼라 기본 타입 (Scalar Value Types)

가장 기본이 되는 데이터 형식입니다. 각 언어별로 매핑되는 타입이 다르므로 주의해야 합니다.

| Type | 설명 | Go | Java | Python |
|---|---|---|---|---|
| double | 64비트 부동소수점 | float64 | double | float |
| float | 32비트 부동소수점 | float32 | float | float |
| int32 | 32비트 정수 (가변 길이 인코딩) | int32 | int | int |
| int64 | 64비트 정수 (가변 길이 인코딩) | int64 | long | int/long |
| uint32 | 부호 없는 32비트 정수 | uint32 | int | int |
| uint64 | 부호 없는 64비트 정수 | uint64 | long | int/long |
| sint32 | 음수 효율이 좋은 32비트 정수 | int32 | int | int |
| fixed32 | 4바이트 고정 정수 (큰 값에 효율적) | uint32 | int | int |
| bool | 논리값 | bool | boolean | bool |
| string | UTF-8 또는 7비트 ASCII 문자열 | string | String | str |
| bytes | 임의의 바이트 시퀀스 | []byte | ByteString | bytes |

## 2. 복합 및 구조적 타입 (Composite Types)

데이터를 묶거나 나열할 때 사용합니다.

- Message: 다른 메시지 타입을 필드로 포함하여 중첩 구조를 만듭니다.
- Enum: 열거형 타입입니다. 첫 번째 값은 반드시 0이어야 합니다 (기본값).
  
```protobuf
enum Corpus {
  UNIVERSAL = 0;
  WEB = 1;
}
```

- Repeated: 배열이나 리스트 형태입니다. (예: repeated string tags = 1;)
- Map: Key-Value 쌍을 가집니다. (예: map<string, Project> projects = 3;)
- Key는 string이나 정수 타입만 가능하며, float이나 bytes는 불가합니다.

## 3. 특수 타입 (Special Types)

구조적 유연성을 위해 사용됩니다.

- Any: 정의되지 않은 임의의 메시지 타입을 담을 수 있습니다. (google/protobuf/any.proto import 필요)
- Oneof: 여러 필드 중 단 하나만 값을 가질 수 있는 경우 사용합니다. 메모리를 절약할 수 있습니다.

oneof contact_info {
  string email = 1;
  string phone_number = 2;
}

- Wrapper Types: int32 같은 기본 타입에 null 상태를 허용하고 싶을 때 사용합니다. (google/protobuf/wrappers.proto 사용)

## 4. Well-Known Types (표준 라이브러리)

Google에서 기본으로 제공하는 유용한 타입들입니다.

- Timestamp: 날짜와 시간 (google/protobuf/timestamp.proto)
- Duration: 시간 간격 (google/protobuf/duration.proto)
- Empty: 인자나 반환값이 없을 때 사용하는 빈 메시지 (google/protobuf/empty.proto)

## 5. proto3 `optional` 적용 가능 타입

`optional`은 **단일(singular) 필드**에만 사용할 수 있으며, 기본값과 별개로 "값이 설정되었는지"를 구분하고 싶을 때 사용합니다.

| 구분 | optional 사용 |
|---|---|
| 스칼라(숫자, bool, string, bytes) | 가능 |
| enum | 가능 |
| message | 보통 불필요 (optional 없이도 "설정됨/미설정" presence 확인 가능) |
| repeated | 불가 |
| map | 불가 |
| oneof 내부 필드 | 불가 |

> 여기서 message 설명의 presence는 **필드 값 자체**가 아니라 **필드가 설정되었는지 여부(has/set)** 를 추적한다는 의미입니다.
> 따라서 message 필드는 proto3에서 기본적으로 optional과 유사한 존재 여부 확인이 가능합니다.

- 스칼라( int32 ,  string  등): 기본 proto3에서는 값만 보이고, “미설정”과 “기본값(0, 빈문자열)” 구분이 안 됨 → 그래서  optional 이 필요
- message 필드:  optional  없이도 “필드가 아예 없음” vs “필드가 존재함”을 구분 가능 (언어별로  hasX() ,  nil/null 여부  등으로 확인)

즉, message 가 문법적으로  optional  키워드를 꼭 써야 한다기보다, 존재 여부 추적 기능이 기본 제공된다고 이해하면 됩니다.

예시:

```protobuf
message UserProfile {
  optional int32 age = 1;       // 가능
  optional string nickname = 2; // 가능
  optional Status status = 3;   // 가능 (enum)

  Profile profile = 4;          // optional 없이도 존재 여부 확인 가능
  repeated string tags = 5;     // optional 불가
  map<string, string> meta = 6; // optional 불가
}
```
