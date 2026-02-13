# Springboot용 JasonMapper Bean 생성

## javadoc

- [jackson](https://javadoc.io/doc/com.fasterxml.jackson.core)
- [JsonMapper](https://javadoc.io/static/com.fasterxml.jackson.core/jackson-databind/2.21.0/com/fasterxml/jackson/databind/json/JsonMapper.html)
- [JsonMapper.Builder](https://javadoc.io/static/com.fasterxml.jackson.core/jackson-databind/2.21.0/com/fasterxml/jackson/databind/json/JsonMapper.Builder.html)
- [DeserializationFeature(ObjectMapper ConfigFeature)](https://javadoc.io/static/com.fasterxml.jackson.core/jackson-databind/2.21.0/com/fasterxml/jackson/databind/DeserializationFeature.html)

## JsonMapper 생성

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.json.JsonMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;

@Configuration
public class JacksonConfig {

    @Bean
    public JsonMapper jsonMapper() {
        return JsonMapper.builder()
            .addModule(new JavaTimeModule())
            .enable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS)
            .enable(DeserializationFeature.USE_BIG_INTEGER_FOR_INTS)
            .disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES) // prevent UnknownPropertyException
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .build();
    }
}
```

## JsonMapper 설정

- `JsonMapper`는 `ObjectMapper`를 상속하므로 `JsonMapper.Builder`에서 `JsonReadFeature`, `JsonWriteFeature` 뿐 아니라 `ObejctMapper`의 설정인 `SerializationFeature`와 `DeserializationFeature`도 설정 가능함

- `ObjectMapper` 설정 - Object Converting 관련 설정
  - [`DeserializationFeature`](https://javadoc.io/static/com.fasterxml.jackson.core/jackson-databind/2.21.0/com/fasterxml/jackson/databind/DeserializationFeature.html)
  - [`SerializationFeature`](https://javadoc.io/static/com.fasterxml.jackson.core/jackson-databind/2.21.0/com/fasterxml/jackson/databind/SerializationFeature.html)

- `JsonMapper` 설정 - json 문법 설정
  - [`JsonReadFeature`](https://javadoc.io/doc/com.fasterxml.jackson.core/jackson-core/latest/com/fasterxml/jackson/core/json/JsonReadFeature.html)
  - [`JsonWriteFeature`](https://javadoc.io/doc/com.fasterxml.jackson.core/jackson-core/latest/com/fasterxml/jackson/core/json/JsonWriteFeature.html)

### ObjectMapper 설정

- **`DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS`**
  - **설명:** JSON의 실수(Float, Double 등) 값을 자바의 `BigDecimal` 타입으로 역직렬화하도록 설정합니다.
  - **이유:** 기본적으로는 `Double`로 변환되는데, 부동소수점 연산으로 인한 정밀도 손실(예: 0.1 + 0.2 != 0.3)을 방지하고 정확한 금융 계산 등을 수행하기 위해 사용합니다.

- **`DeserializationFeature.USE_BIG_INTEGER_FOR_INTS`**
  - **설명:** JSON의 정수 값을 자바의 `BigInteger` 타입으로 역직렬화하도록 설정합니다.
  - **이유:** 정수 값이 자바의 `int`나 `long` 범위를 초과할 수 있는 매우 큰 수일 경우, 오버플로우 없이 정확한 값을 유지하기 위해 사용합니다.

- **`DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES`**
  - **설명:** JSON 데이터에는 존재하지만 매핑하려는 자바 객체(VO/DTO)에는 없는 필드가 있을 경우 예외(`UnrecognizedPropertyException`) 발생 여부를 제어합니다.
  - **이유:** `.disable()`로 설정하면, 자바 객체에 없는 필드가 JSON에 포함되어 있어도 에러를 발생시키지 않고 무시합니다. API 응답 스펙이 변경되어 필드가 추가되어도 클라이언트 코드가 깨지지 않게 하는 유연성을 제공합니다.

- **`SerializationFeature.WRITE_DATES_AS_TIMESTAMPS`**
  - **설명:** 날짜/시간 타입(`LocalDate`, `LocalDateTime`, `Date` 등)을 직렬화할 때 숫자형 타임스탬프(예: 1672531200)로 변환할지 여부를 결정합니다.
  - **이유:** `.disable()`로 설정하면, 알아보기 힘든 숫자 대신 ISO-8601 표준 문자열 형식(예: `"2023-01-01T12:00:00"`)으로 날짜를 직렬화하여 가독성을 높입니다. (단, `JavaTimeModule` 등록 필요)
