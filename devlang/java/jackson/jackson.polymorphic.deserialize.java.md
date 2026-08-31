# Java: json 다형성(Polymorphic) 직렬화/역직렬화

`JsonTypeInfo`는 Json의 특정 필드 값 또는 클래스 이름을 기준으로 맵핑될 실제 클래스를 `JsonSubTypes`로 명시해 주는 기법이다.

## 핵심 요약

- `@JsonTypeInfo`와 `@JsonSubTypes`는 역할이 나뉩니다.
  - `@JsonTypeInfo`: 부모(추상) 타입에 선언해 "타입 식별자를 어떤 방식(`use`)으로, 어디에(`include`) 기록할지" 규칙을 정의합니다.
  - `@JsonSubTypes`: 그 규칙에서 사용할 **논리적 이름(name) ↔ 실제 하위 클래스(value)** 매핑 테이블을 정의합니다.
- 동작 흐름:
  - **역직렬화 시**: JSON에서 `@JsonTypeInfo(property = "type")`로 지정된 필드 값(예: `"DOG"`)을 읽음 → `@JsonSubTypes` 매핑 테이블에서 일치하는 클래스(`Dog.class`)를 찾음 → 해당 클래스의 인스턴스로 변환.
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
    - `As.PROPERTY`: `JsonTypeInfo(property)`로 지정한 이름의 프로퍼티(필드)를 서브 타입 클래스에 자동으로 새로 추가합니다. 서브타입 클래스가 `JsonTypeInfo(property)`와 같은 이름의 프로퍼티를 **별도로 갖고 있지 않을 때** 사용합니다. 만약 서브타입에 같은 이름의 프로퍼티가 이미 있다면 충돌하여 예외가 발생하므로, 이 경우엔 `As.EXISTING_PROPERTY`를 사용해야 합니다.
    - `As.EXISTING_PROPERTY`: 이미 클래스에 존재하는 프로퍼티를 `JsonTypeInfo(property)`로 지정된 타입 식별자로 재사용합니다. (필드가 중복 추가되지 않음) 서브타입이 `JsonTypeInfo(property)`와 같은 이름의 필드를 실제 프로퍼티(예: record 컴포넌트, business logic에서 쓰는 enum 필드)로 이미 갖고 있을 때 필수입니다.
      - **`As.PROPERTY` vs `As.EXISTING_PROPERTY` 선택 기준**: 코드 상에서 서브 타입에 있는 특정 프로퍼티 값을 기준으로 이 서브 타입이 실제로 어떤 클래스 인지 구분하려면 `As.EXISTING_PROPERTY`를 사용해야 합니다. 서브타입이 `JsonTypeInfo(property)`와 같은 이름의 필드를 실제 Java 프로퍼티로 가질 필요가 없다면(순수 JSON 다형성 처리만 목적) `As.PROPERTY`가 더 간단합니다. Java 클래스 자체(`instanceof`, sealed + switch 패턴 매칭)가 이미 타입 판별자 역할을 하므로 별도 필드를 둘 필요가 없습니다. 반대로 그 값이 Jackson과 무관한 이유로도 실제 필드/프로퍼티여야 한다면(예: sealed interface가 `type()`을 추상 메서드로 선언해 `instanceof` 없이 분기·로깅·비교하고 싶을 때, 그 값이 DB 컬럼이나 gRPC/protobuf 등 JSON 밖의 포맷에도 쓰여야 할 때, 이미 존재하는 도메인 모델 필드를 그대로 써야 할 때) `As.EXISTING_PROPERTY`로 "이미 있는 필드를 재사용해라"고 지정해야 충돌 없이 동작합니다.
    - `As.EXTERNAL_PROPERTY`: 타입 식별자를 대상 객체 내부가 아니라, 그 객체를 담고 있는 **부모(외부) 객체안에 형제(sibling) 필드**에 기록합니다. 부모(추상) 타입이 아니라 **서브 타입 필드**에 애너테이션을 선언해야 합니다.
    - `As.WRAPPER_OBJECT`: `{ "타입명": { ...실제 데이터... } }` 형태로 객체를 한 번 더 감쌉니다.
    - `As.WRAPPER_ARRAY`: `[ "타입명", { ...실제 데이터... } ]` 형태의 2개짜리 배열로 감쌉니다.
  - `property`: 타입 식별자를 저장할 JSON 필드명을 지정합니다. (예: `"type"`)
  - `visible`: `true`로 설정하면 `JsonTypeInfo(property)`로 저장된 타입 식별자 값을 역직렬화된 객체의 필드에도 그대로 매핑합니다. (기본값 `false`)
  - `defaultImpl`: JSON에 타입 정보가 없거나 일치하는 타입을 찾지 못했을 때 사용할 기본 구현 클래스를 지정합니다.

**`@JsonSubTypes(...)`**

- `@JsonTypeInfo(use = Id.NAME, ...)`와 함께 사용되며, 논리적 타입 이름(name)과 실제 하위 클래스(Java/Kotlin class) 간의 매핑 테이블을 정의하는 어노테이션입니다.
- 부모 클래스에 선언하며, 내부에 여러 개의 `@JsonSubTypes.Type` 항목을 배열로 나열합니다.
- `@JsonSubTypes.Type`의 주요 파라미터:
  - `value`: 매핑 대상이 되는 하위 클래스(`.class`)를 지정합니다.
  - `name`: 해당 하위 클래스에 대응하는 논리적 이름(문자열)을 지정합니다. `@JsonTypeInfo`의 `property`에 지정된 필드 값과 매칭됩니다.
  - `names`: 하나의 클래스에 여러 개의 별칭(이름)을 매핑하고 싶을 때 사용하는 문자열 배열입니다. (Jackson 2.12+)
- 동작 방식:
  - **직렬화 시**: 객체의 실제 런타임 클래스를 보고, 매핑 테이블에서 일치하는 `name`을 찾아 JSON의 타입 필드 값으로 씁니다.
  - **역직렬화 시**: JSON의 타입 필드 값을 읽고, 매핑 테이블에서 일치하는 `value`(클래스)를 찾아 해당 타입의 인스턴스로 생성합니다.

## 예제

`include`(`JsonTypeInfo.As`) 옵션별로 다형성 직렬화/역직렬화 예제를 정리한다.

### `As.PROPERTY`

가장 기본적인 방식. `JsonTypeInfo(property)`로 지정한 타입 식별자를 대상 객체 내부에 새 프로퍼티로 추가한다.

```java
import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.PROPERTY,
    property = "type"
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = Dog.class, name = "DOG"),
    @JsonSubTypes.Type(value = Cat.class, name = "CAT"),
})
public interface Animal {
}

public class Dog implements Animal {
    public String breed;
}

public class Cat implements Animal {
    public boolean isIndoor;
}
```

**직렬화 결과:**
```json
{ "type": "DOG", "breed": "진돗개" }
```

- `type` 필드가 `Dog`/`Cat` 클래스에 정의되어 있지 않아도, Jackson이 자동으로 추가/제거한다.

### `As.EXISTING_PROPERTY`

sealed interface + record 조합 사용. 타입 판별자(`type`)가 각 서브타입의 필드와 함께 존재하므로 `EXISTING_PROPERTY`를 사용하고, record 생성자 바인딩에 값이 채워지도록 `visible = true`를 지정한다.

`As.PROPERTY`를 대신 쓰면 안 되는 이유: `record Dog(String breed, AnimalType type)`처럼 `type`이 생성자 바인딩에 필요한 실제 컴포넌트(프로퍼티)이기 때문이다. `As.PROPERTY`는 `JsonTypeInfo(property)`로 지정한 타입 식별자를 Jackson이 별도로 관리하는 필드로 취급하는데, 클래스 자체에도 이미 같은 이름(`type`)의 프로퍼티가 존재하므로 두 관리 주체가 충돌해 예외가 발생한다. 서브타입이 `JsonTypeInfo(property)`와 같은 이름의 필드를 실제 프로퍼티로 이미 갖고 있을 때는 `As.EXISTING_PROPERTY`로 "기존 프로퍼티를 재사용하라"고 지정해야 한다.

```java
import com.fasterxml.jackson.annotation.JsonProperty;
import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

// 부모 타입: 다형성의 "창구" 역할
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.EXISTING_PROPERTY,
    property = "type",
    visible = true
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = Animal.Dog.class, name = "DOG"),
    @JsonSubTypes.Type(value = Animal.Cat.class, name = "CAT"),
})
public sealed interface Animal {

    AnimalType type();

    // 하위 타입 1: 강아지
    record Dog(
        @JsonProperty("breed") String breed,
        AnimalType type
    ) implements Animal {
        public Dog(String breed) {
            this(breed, AnimalType.DOG);
        }
    }

    // 하위 타입 2: 고양이
    record Cat(
        @JsonProperty("is_indoor") boolean isIndoor,
        AnimalType type
    ) implements Animal {
        public Cat(boolean isIndoor) {
            this(isIndoor, AnimalType.CAT);
        }
    }

    enum AnimalType {
        DOG, CAT
    }
}
```

#### 사용하는 쪽 (다형성 필드를 가진 클래스)

```java
import java.util.List;

public record Zoo(
    List<Animal> animals
) {
}
```

#### 직렬화 (객체 → JSON)

```java
Zoo zoo = new Zoo(List.of(
    new Animal.Dog("진돗개"),
    new Animal.Cat(true)
));

String json = objectMapper.writeValueAsString(zoo);
System.out.println(json);
```

**결과 JSON:**
```json
{
  "animals": [
    { "type": "DOG", "breed": "진돗개" },
    { "type": "CAT", "is_indoor": true }
  ]
}
```
- 각 객체의 실제 런타임 타입(`Dog`, `Cat`)에 따라 `type` 필드 값(`"DOG"`, `"CAT"`)이 자동으로 채워집니다.
- `@JsonSubTypes` 매핑 덕분에 Jackson이 "Dog 클래스는 DOG라는 이름으로 표시해야 한다"는 것을 알고 있습니다.

#### 역직렬화 (JSON → 객체)

```java
String json = """
{
  "animals": [
    { "type": "DOG", "breed": "진돗개" },
    { "type": "CAT", "is_indoor": true }
  ]
}
""";

Zoo zoo = objectMapper.readValue(json, Zoo.class);
```

### `As.EXTERNAL_PROPERTY`

타입 식별자를 대상 객체 내부가 아니라, 그 객체를 담고 있는 부모 객체의 형제 필드에 기록한다. 부모(추상) 타입이 아니라 **해당 필드를 감싸는 클래스**에 애너테이션을 선언해야 한다.

```java
import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

public class Container {

    @JsonTypeInfo(
        use = JsonTypeInfo.Id.NAME,
        include = JsonTypeInfo.As.EXTERNAL_PROPERTY,
        property = "animalType"
    )
    @JsonSubTypes({
        @JsonSubTypes.Type(value = Dog.class, name = "DOG"),
        @JsonSubTypes.Type(value = Cat.class, name = "CAT"),
    })
    public Animal animal;

    public String animalType;
}
```

**직렬화 결과:**
```json
{
  "animalType": "DOG",
  "animal": { "breed": "진돗개" }
}
```

### `As.WRAPPER_OBJECT`

타입명을 key로, 실제 데이터를 value로 하는 객체로 한 번 더 감싼다.


```java
import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.WRAPPER_OBJECT
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = Dog.class, name = "DOG"),
    @JsonSubTypes.Type(value = Cat.class, name = "CAT"),
})
public interface Animal {
}

public class Dog implements Animal {
    public String breed;
}

public class Cat implements Animal {
    public boolean isIndoor;
}
```

**직렬화 결과:**
```json
{ "DOG": { "breed": "진돗개" } }
```

`As.PROPERTY`와의 차이: `As.PROPERTY`는 `property`(예: `"type"`)로 지정한 필드에 타입 식별자를 넣어 대상 객체 내부에 함께 기록하지만, `As.WRAPPER_OBJECT`는 타입명을 감싸는 바깥 객체의 key로 쓰기 때문에 별도의 필드명(`property`)이 필요 없다. 그 결과 직렬화 결과의 JSON 구조 자체가 달라진다.
- `As.PROPERTY`: `{ "type": "DOG", "breed": "진돗개" }`
- `As.WRAPPER_OBJECT`: `{ "DOG": { "breed": "진돗개" } }`

### `As.WRAPPER_ARRAY`

타입명과 실제 데이터를 2개짜리 배열로 감싼다.

굳이 객체(`As.WRAPPER_OBJECT`) 대신 배열을 쓰는 이유:
- **순서 보장**: JSON 명세상 배열 원소의 순서는 항상 보장되지만, 객체의 key 순서는 명세상 보장되지 않습니다(실무에서는 대부분 입력 순서가 유지되지만). 전체를 다 파싱하기 전에 타입부터 먼저 확인해야 하는 스트리밍 파서 환경에서는 배열 구조가 구조적으로 더 안전합니다.
- **외부 포맷과의 호환**: pub/sub 이벤트 디스패치(예: Socket.IO의 `["eventName", payload]`)나 일부 RPC 프로토콜처럼, 이미 `[식별자, 데이터]` 튜플 컨벤션을 사용하는 외부 시스템과 JSON을 주고받을 때 별도 변환 없이 그 포맷에 맞출 수 있습니다.

순수하게 애플리케이션 내부에서만 주고받는 JSON이라면 필드명이 있는 `As.WRAPPER_OBJECT`가 더 읽기 쉽고 일반적인 JSON 관례에 가깝습니다.

```java
import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.WRAPPER_ARRAY
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = Dog.class, name = "DOG"),
    @JsonSubTypes.Type(value = Cat.class, name = "CAT"),
})
public interface Animal {
}
```

**직렬화 결과:**
```json
[ "DOG", { "breed": "진돗개" } ]
```

- 타입 식별자(`animalType`)가 `animal` 필드와 같은 레벨(형제 필드)에 위치한다.
- 역직렬화 시에도 Jackson이 `Container`의 `animalType` 값을 먼저 읽어 `animal` 필드를 어떤 클래스로 만들지 결정한다.

- Jackson은 `animals` 리스트의 각 원소가 선언상 `Animal` 타입(추상)이라 그 자체로는 어떤 구체 클래스를 만들어야 할지 모릅니다.
- 이때 JSON에 있는 `"type": "DOG"` 값을 읽고, `@JsonSubTypes` 매핑 테이블에서 `"DOG"` → `Dog.class`를 찾아 **`Dog` 인스턴스로 역직렬화**합니다.
- `"type": "CAT"`이면 `Cat.class`로 매핑되어 `Cat` 인스턴스가 생성됩니다.