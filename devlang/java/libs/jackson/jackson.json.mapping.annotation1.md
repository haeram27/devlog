# Jackson - Json Deserialize 용 Model Class 정의

Jackson을 사용하여 jsopn을 java object로 직렬화/역직렬화 할 때 자주 사용되는 annotaion과 Model Class의 구조를 설명

## Json Mapping Annotations

- [jackson annotation api doc](https://javadoc.io/doc/com.fasterxml.jackson.core/jackson-annotations/latest/com.fasterxml.jackson.annotation/com/fasterxml/jackson/annotation/package-summary.html)

### Sample Json Mapping Class

- 다음은 Jackson의 json mapping을 위한 annotation을 설명하기 위한 샘플 Model class이다.

- `DateFormat.class`

```java
@Data
public class DateFormat implements Serializable {
    @JsonProperty("$date")
    private LocalDateTime date;
}
```

- `Employee.class`

```java
@Data
@JsonPropertyOrder({ "employee_id", "user_name", "department" })
@JsonInclude(Include.NON_NULL)
@JsonIgnoreProperties(ignoreUnknown = true)
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class Employee {
    /* json document
    {
      "employee_id": 1234,
      "user_name": "mike",
      "department": "development",
      "computer_name": "my-PC",
      "ip": "1.2.3.4",
      "client_time": { "$date": "2017-06-30T16:04:14.421Z" },
      "tz_offset": 9
    }
    */

    @JsonProperty("employee_id") private Long employeeId;

    @JsonProperty("user_name") private String userName;

    @JsonDeserialize(using = EmptyToNullStringDeserializer.class)
    @JsonProperty("department") private String department;

    @JsonProperty("computer_name") private String computerName;

    @JsonProperty("ip") private String ip;

    @JsonProperty("client_time") private DateFormat clientTime;

    @JsonIgnore
    @JsonProperty("tz_offset") private Integer tzOffset;
}
```

### 직렬화

- java object -> json

### 역직렬화

- json -> java object

### `@Data`

jackson이 json 직렬화/역직렬화 할 때, java object의 getter/setter를 사용하는데, 인스턴스화 할 때 getter/setter 자동 완성하는 lombok annotation

### `@JsonPropertyOrder({ "field", ... })`

- 대상: 클래스
- 용도: 직렬화
- json 직렬화시 json에 명시될 필드 순서 지정

### `@JsonInclude(Include.NON_NULL)`

- 대상: 클래스
- 용도: 직렬화
- java 객체를 json으로 직렬화(Serialize)할 때, java 필드 값이 non-null인 필드만 직렬화함
- null인 java 필드는 json 에 생성하지 않음

### `@JsonIgnore`

- 대상: 필드
- 용도: 직렬화
- 직렬화 할 때, 지정 필드를 무시하여 JSON 결과에 포함하지 않도록 설정

### `@JsonIgnoreProperties(ignoreUnknown = true)`

- 대상: 필드
- 용도: 역직렬화
- 일반적인 경우 JSON을 Java 객체로 역직렬화(Deserialize)할 때, Java 객체의 필드에 해당하는 JSON 필드가 존재 하지 않으면 변환시 Exception 발생함
- 이 annotaion 사용시 Java 객체의 필드에 해당하는 JSON 필드가 존재 하지 않는 경우 해당 Json 필드에 대해 역직렬화를 무시하며 Exception 발생 억제
- `JsonMapper` 생성시 `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES` 속성을 disable 하면 해당 mapper를 사용시 별도의 annotion 없이도 기본적으로 동일 기능(매칭 되지 않는 필드 발생시 Exception 억제) 제공
- `JsonMapper`에 `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES` 사용을 추천

```java
JsonMapper.builder().disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES).build();
```

### `@JsonProperty("json_field_name")`

- 대상: 필드
- 용도: 직렬화/역직렬화
- json 직렬화/역직렬화시 java object 필드에 대응하는 json 필드의 이름을 지정
- 기본적으로 json 필드 이름과 java object 필드 이름이 동일해야 매칭이 됨
- 개발 언어의 성향에 따라 json 필드 네이밍 룰이 다른 경우 있음
  - java 는 `lowerCamelCase` 선호
  - native language들은 `snake_case` 선호

### `@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)`

- 대상: 클래스
- 용도: 직렬화/역직렬화
- json의 네이밍 룰이 무엇인지 지정
- 지정된 json 네이밍 룰에 맞는 json 필드의 이름을 자동 변환하여 적절한 java 필드(lowerCamelCase)에 매칭함
- 이 annotation 사용시 `@JsonProperty`를 각 java 필드마다 별도로 지정할 필요가 없음

### `@JsonDeserialize(using = EmptyToNullStringDeserializer.class)`

- 대상: 필드
- 용도: 역직렬화
- 특정 json 필드의 역질렬화를 사용자 지정 방식을 사용해야할 때 Deserializer 클래스를 지정
- Deserializer sample: `EmptyToNullStringDeserializer.class`

```java
// json 필드 값이 빈 문자열인 경우 null 값으로 역질렬화
public class EmptyToNullStringDeserializer extends com.fasterxml.jackson.databind.JsonDeserializer<String> {
    @Override
    public String deserialize(com.fasterxml.jackson.core.JsonParser p,
                              com.fasterxml.jackson.databind.DeserializationContext ctxt)
        throws java.io.IOException {
        String v = p.getValueAsString();
        return (v == null || v.isEmpty()) ? null : v;
    }
}
```

## Mapping Model Class에 Serializable 필요 여부

- jackson을 사용하는 json mapping용 Model class에서 `Serializable`은 필요하지 않음
- `Serializable`은 Java의 일반적인 직렬화/역직렬화(network/file 전송용)시 사용되는 인터페이스이며, jackson은 이 인터페이스를 사용하지 않음

```java
public class MappingModel implements Serializable {
  private static final long serialVersionUID = -2607522337L;
  ...
}
```

## Mapping Model Class는 public class 이어야 함

- jackson은 nested class를 mapping 용도로 사용하지 못함

## 추천 Mapping Class 구현

- Java Mapping Object: `Employee.class`

```java
@Data
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class Employee {
    /* json document
    {
      "employee_id": 1234,
      "user_name": "mike",
      "department": "development",
      "computer_name": "my-PC",
      "ip": "1.2.3.4",
      "client_time": { "$date": "2017-06-30T16:04:14.421Z" },
      "tz_offset": 9
    }
    */

    private Long employeeId;

    private String userName;

    private String department;

    private String computerName;

    @JsonDeserialize(using = EmptyToNullStringDeserializer.class)
    private String ip;

    private DateFormat clientTime;

    private Integer tzOffset;
}
```