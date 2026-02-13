# TypeReference in Jackson

## jackson JsonNode를 java object로 mapping 하는 예제

```java
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.json.JsonMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;

JsonMapper jsonMapper = JsonMapper.builder()
            .addModule(new JavaTimeModule())
            .enable(DeserializationFeature.USE_BIG_DECIMAL_FOR_FLOATS)
            .enable(DeserializationFeature.USE_BIG_INTEGER_FOR_INTS)
            .disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
            .build();

JsonNode jsonNode = ...

if (node.isObject()) {
    ObjectNode node = (ObjectNode) jsonNode;
    Map<String, Object> map = mapper.convertValue(node,
        new com.fasterxml.jackson.core.type.TypeReference<Map<String,Object>>() {});
    // var map = jsonMapper.treeToValue(node, Map.class);
    list = new ArrayList<Map<String, Object>>();
    list.add(map);
} else if (node.isArray()) {
    ArrayNode node = (ArrayNode) jsonNode;
    list = mapper.convertValue(node,
        new com.fasterxml.jackson.core.type.TypeReference<List<Map<String,Object>>>() {});
}
```

## `com.fasterxml.jackson.core.type.TypeReference` 클래스

- 제네릭 타입 정보를 담기 위한 빈 클래스
- Jackson에서 제네릭 타입 정보를 런타임에 유지하려고 만든 클래스
- Jackson은 제네릭 타입을 정보를 런타임에 유지 하여 메소드 간 전달 및 reflection을 이용한 제너릭 타입 참조등에 사용한다
- 제네릭 타입 정보를 가지는 것 외에 TypeReference클래스는 어떤 정보나 특정 행위를 위한 메소드를 갖지 않는다
- 그래서 TypeReference 클래스의 구현체에는 어떠한 내용도 상속하거나 구현할 필요가 없다
- TypeReference 클래스는 제네릭 타입 정보를 메소드의 인자로 넘겨주기 용도로 자주 사용된다

## `new TypeReference<>() {}` 의 의미

- `TypeReference<T>` 는 **추상 클래스**이지만 abstract method를 갖지 않는다
- 추상 클래스는 **그 자체로 인스턴스를 만들 수 없고**, 반드시 **구현체(서브클래스)** 가 필요하다
- 따라서 `new TypeReference<>() {}` 구문은 **익명 하위 클래스(anonymous subclass)** 를 생성하는 문법이다
- `TypeReference` 제네릭 타입을 담는 그릇 용도로만 사용되며 abstract method를 갖지 않기 때문에 구현체를 생성할 때 아무런 구현 내용이 없는 익명 클래스로 생성한다.

즉,

```java
new TypeReference<>() {};
```

는 **TypeReference의 익명 하위 클래스를 정의하고 동시에 인스턴스화**하는 코드이다

---

## '{}' 사용 이유

- 만약 이렇게 쓴다면:

  ```java
  new TypeReference<>();
  ```

  → 단순히 **추상 클래스 인스턴스를 만들려는 시도**라서 컴파일 에러가 발생

- 반면,

  ```java
  new TypeReference<>() {};
  ```

  → **본문이 비어 있는 익명 클래스**를 선언해서 인스턴스를 만들기 때문에 유효하다
  (추상 메소드가 없으므로 override 할 건 없고, 그냥 껍데기 클래스가 만들어짐)

---

## Jackson에서 쓰이는 이유

Java의 **제네릭 타입은 런타임에 지워지기(타입 소거, type erasure)** 때문에, 보통은 구체적인 제네릭 정보를 얻을 수 없다

Jackson은 `TypeReference<T>` 의 **익명 하위 클래스의 제네릭 시그니처**를 리플렉션으로 읽어들여 **런타임에 타입 정보를 보존**한다

예:

```java
List<String> list = mapper.readValue(json,
    new TypeReference<List<String>>() {});
```

- 여기서 `List<String>` 이라는 타입 인자가 익명 하위 클래스의 메타데이터에 남기 때문에, Jackson이 정확히 "문자열 리스트"로 파싱할 수 있다
- 만약 그냥 `List.class` 를 넘기면 `List<Object>` 로만 처리되어 버린다
