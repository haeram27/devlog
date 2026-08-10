# `map<string, google.protobuf.Struct>` 형식 사용

## Protobuf 정의 방식 (.proto)

다음과 같이 외부 struct.proto를 import하여 맵의 값으로 정의하면 됩니다.

```plain
syntax = "proto3";
import "google/protobuf/struct.proto";
message FlexiblePayload {
  // key: 식별자 문자열, value: 임의의 구조화된 JSON 객체
  map<string, google.protobuf.Struct> dynamic_data = 1;
}
```

## Java에서 컴파일된 결과 및 사용법

이 구조를 Java 코드로 컴파일하면 `java.util.Map<String, com.google.protobuf.Struct>` 형태로 다룰 수 있게 됩니다. 데이터를 넣고 매핑하는 방법은 다음과 같습니다.

### 1. 송신: 데이터 빌드 및 추가 (Java)

```java
import com.google.protobuf.Struct;import com.google.protobuf.Value;
// 1. 개별 Struct (JSON 역할) 생성
Struct userConfig = Struct.newBuilder()
    .putFields("theme", Value.newBuilder().setStringValue("dark").build())
    .putFields("retries", Value.newBuilder().setNumberValue(3).build())
    .build();
// 2. map<string, Struct>에 데이터 삽입
FlexiblePayload payload = FlexiblePayload.newBuilder()
    .putDynamicData("user_123", userConfig)
    .build();
```

### 2. 수신: 특정 Java Type으로 변환 (Jackson 활용)

이전 질문과 마찬가지로, 맵에서 꺼낸 Struct 객체들을 각각의 `Java 특정 클래스(POJO)`나 `Map<String, Object>`로 변환할 수 있습니다.

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.protobuf.Struct;
import com.google.protobuf.util.JsonFormat;

public class ProtobufMapper {
    private static final ObjectMapper objectMapper = new ObjectMapper();

    // Struct to POJO 변환
    public static <T> T convertStructToPojo(FlexiblePayload payload, String key, Class<T> targetClass) throws Exception {
        // 1. 맵에서 특정 키의 Struct 가져오기
        Struct struct = payload.getDynamicDataOrDefault(key, null);
        if (struct == null) return null;

        // 2. Struct를 JSON 문자열로 변경 후 Jackson 매핑
        String jsonString = JsonFormat.printer().print(struct);
        return objectMapper.readValue(jsonString, targetClass);
    }

    // Struct to Map<String, Object> 변환
    public static Map<String, Object> convertStructToPojo(FlexiblePayload payload, String key) throws Exception {
        // 1. 맵에서 특정 키의 Struct 가져오기
        Struct struct = payload.getDynamicDataOrDefault(key, null);
        if (struct == null) return null;

        // 2. Struct를 JSON 문자열로 변경 후 Jackson 매핑
        String jsonString = JsonFormat.printer().print(struct);
        return objectMapper.readValue(jsonString, new TypeReference<Map<String, Object>>() {});
    }
}
```

### 3. 참고: `com.google.protobuf.Struct` 타입의 Java Type 변환 도구

```java
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.protobuf.Struct;
import com.google.protobuf.util.JsonFormat;
import java.util.Map;
import java.util.List;

public class ProtobufMapper {
    private static final ObjectMapper objectMapper = new ObjectMapper();

    // Struct to POJO 변환
    public static <T> T convertStructToPojo(Struct struct, Class<T> targetClass) throws Exception {
        // 1. Struct 객체를 JSON 문자열로 변환
        String jsonString = JsonFormat.printer().print(struct);
        
        // 2. Jackson을 이용해 JSON 문자열을 특정 Type으로 매핑
        return objectMapper.readValue(jsonString, targetClass);
    }

    // Struct to Map<String, Object> 변환
    public static Map<String, Object> convertStructToMap(Struct struct) throws Exception {
        // 1. Struct 객체를 JSON 문자열로 변환
        String jsonString = JsonFormat.printer().print(struct);
        
        // 2. Jackson TypeReference를 사용하여 Map으로 변환
        return objectMapper.readValue(jsonString, new TypeReference<Map<String, Object>>() {});
    }
}
```

### 4. 참고: Java 객체(POJO)를 Struct로 한번에 변환 (역방향)

필드마다 `putFields`로 채우지 않고, Jackson으로 JSON 문자열을 만든 뒤 `JsonFormat.parser()`로 `Struct.Builder`에 바로 파싱하면 한 번에 변환됩니다.

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.protobuf.Struct;
import com.google.protobuf.util.JsonFormat;

public class PojoToStructMapper {
    private static final ObjectMapper objectMapper = new ObjectMapper();

    public static Struct convertPojoToStruct(Object pojo) throws Exception {
        // 1. Java 객체 -> JSON 문자열
        String jsonString = objectMapper.writeValueAsString(pojo);

        // 2. JSON 문자열 -> Struct.Builder에 파싱
        Struct.Builder structBuilder = Struct.newBuilder();
        JsonFormat.parser().ignoringUnknownFields().merge(jsonString, structBuilder);
        return structBuilder.build();
    }
}
```

`Map<String, Object>`도 동일한 방식으로 바로 Struct에 넣을 수 있습니다.

> 주의: `Struct`는 JSON 값 타입만 지원(`string`, `number`(double), `bool`, `null`, `struct`, `list`)하므로, Java의 `long`, `BigDecimal`, `LocalDateTime` 등은 직렬화된 형태로 저장되며 큰 `long` 값은 `double` 변환 시 정밀도 손실(약 $2^{53}$ 초과 시)이 발생할 수 있습니다.

## 활용 시나리오 추천

이 패턴은 멀티테넌트(Multi-tenant) 시스템이나 설정(Configuration) 관리 서버에서 자주 쓰입니다.

* Key: 서비스 ID, 유저 ID, 모듈 이름
* Value (Struct): 해당 모듈이나 유저가 사용하는 제각각의 자유로운 JSON 설정값

이렇게 구성하시면 gRPC 통신에서도 유연성을 확보하면서 데이터를 깔끔하게 분류할 수 있습니다.
이 맵 구조를 가지고 Spring Controller 등에서 JSON으로 최종 직렬화하여 클라이언트에 내려주는 과정이나, 더 복잡한 중첩 변환이 필요하시면 언제든 말씀해 주세요!
