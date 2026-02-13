# Jackson annotation (Json DTO, VO 구현시 사용)

## @JsonInclude(Include.NON_NULL)

- ALWAYS
- NON_NULL - level:class, non-null인 property만 직렬화
- NON_ABSENT
- NON_EMPTY - level:class, non-empty인 property만 직렬화
- NON_DEFAULT
- CUSTOM
- USE_DEFAULTS

## `@JsonIgnore`

- level:property
- 지정한 property를 json 직렬화/역직렬화에서 제외

## `@JsonIgnoreProperties([value=]{ "<property1>", "<property2>, ..." })`

- level:class
- 지정된 json의 `<property>`는 (역)직렬화에서 제외 

## `@JsonIgnoreProperties(ignoreUnknown = true)`

- level:class
- (역)직렬화 중 json과 DTO에 맵핑이 되지 않는 property가 존재하는 경우 에러를 던지지 않고 무시

## `@JsonIgnoreType`

- level:class
- level:class

## Jackson Spring web 연동시 @RequestBody 자동 맵핑

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

Http Post 메세지의 request body 에 담긴 JSON 데이터 `{"id": "123", "name": "John Doe"}`는 `createUser` 메서드의 `user` 객체에 매핑됩니다.

## DTO 정의시 자주 사용되는 annotaion

- `@DATA `
  - pakckage - lombok.Data
- `@Jsonxxx`
  - `<jackson>`

### Case Naming Convention

- snake_case
- UPPER_SNAKE_CASE
- lowerCamelCase
- UpperCamelCase (PascalCase)
- lowercase
- kebab-case
- lower.dot.case

## `@JsonNaming과 @JsonProperty`

- jackson library가 Java DTO를 Json 문서와 맵핑할 때, DTO의 필드가 맵핑될 Json 문서상 pair의 key 이름을 지정하는데 사용
- @JsonNaming은 클래스에 적용되어 class의 모든 대상 필드 이름을 특정 naming convention(ex> snake_case)로 자동 변환하는 방식을 지정하는데 사용한다.
- @JsonProperty는 DTO 클래스의 각 Field에 적용되어 지정 Field가 변환될 Json 문서상 pair의 key 이름을 직접 명시하는데 사용한다.

## `@JsonNaming(<PropertyNamingStrategies.XXX.class>)`

- [PropertyNamingStrategies](https://javadoc.io/static/com.fasterxml.jackson.core/jackson-databind/2.21.0/com/fasterxml/jackson/databind/PropertyNamingStrategies.html)

## 사용예

```java
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class MyDTO {
```

## annotation 대상: class

Java DTO의 Field가 맵핑될 Json 문서 상의 key 이름을 정하는 방식 지정
이 annotation이 적용된 클래스의 

value 가능한 case class:

|Class 명|Case Naming Convention Example |
|:---|:---|
|PropertyNamingStrategy.LowerCamelCaseStrategy.class|lowerCamelCase|
|PropertyNamingStrategy.UpperCamelCaseStrategy.class|UpperCamelCase (PascalCase)|
|PropertyNamingStrategy.SnakeCaseStrategy.class|snake_case|
|PropertyNamingStrategy.UpperSnakeCaseStrategy.class|UPPER_SNAKE_CASE|
|PropertyNamingStrategy.LowerCaseStrategy.class|lowercase|
|PropertyNamingStrategy.KebabCaseStrategy.class|kebab-case|
|PropertyNamingStrategy.LowerDotCaseStrategy.class|lower.dot.case|

## `@JsonProperty("<Json Field Name>")`

annotation 대상: field(멤버 변수)
Java DTO의 Field가 맵핑될 Json 문서 상의 key 이름을 지정

## `@JsonNaming과 @JsonProperty 혼용`

- 혼용할 경우 @JsonProperty로 지정한 filed-name이 최종 json에 기록된다.
- 단, json mapping 처리 순서는 @JsonNaming이 먼저 처리되고 @JsonProperty가 나중에 처리되므로, json 데이터가 생성될 때 @JsonProperty로 명명한 field는 @JsonNaming으로 명명된 fileld의 아래 위치로 순서가 배치 된다.

## mapping snake-case(json) word to lower-camel-case(java)

- `@JsonNaming` 어노테이션을 이용해서 DTO(java pojo)의 전체 필드명을 json 문서에 snake_case로 변환 맵핑

java object

```java
package com.dto.test.model;

import java.io.Serializable;
import java.util.Date;
import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.databind.PropertyNamingStrategies;
import com.fasterxml.jackson.databind.annotation.JsonNaming;
import lombok.Data;

@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
@Data
public class NodeHW {
 
  private Integer nodeId;

  private String biosName;

  //@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm a z")
  private Date biosDate;
  
  private Boolean updateStatus;
}
```

- `@JsonProperty` 어노테이션을 이용해서 DTO(java pojo)의 각 필드명을 json 문서에 snake_case로 변환 맵핑

json data

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

import java.io.Serializable;
import java.util.Date;
import com.fasterxml.jackson.annotation.JsonFormat;
import com.fasterxml.jackson.databind.PropertyNamingStrategies;
import com.fasterxml.jackson.databind.annotation.JsonNaming;
import lombok.Data;

@Data
public class NodeHW {

  @JsonProperty("node_id")
  private Integer nodeId;

  @JsonProperty("bios_name")
  private String biosName;

  @JsonProperty("bios_date")
  //@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm a z")
  private Date biosDate;

  @JsonProperty("update_status")
  private Boolean updateStatus;
}
```
