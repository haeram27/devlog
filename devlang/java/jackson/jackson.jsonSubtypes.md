# `@JsonTypeInfo`를 이용한 json 다형성(Polymorphic) 직렬화/역직렬화

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
    - `As.PROPERTY`: 타입 정보를 별도의 프로퍼티(필드)로 추가합니다.
    - `As.EXISTING_PROPERTY`: 이미 클래스에 존재하는 프로퍼티를 타입 식별자로 재사용합니다. (필드가 중복 추가되지 않음)
    - `As.WRAPPER_OBJECT`: `{ "타입명": { ...실제 데이터... } }` 형태로 객체를 한 번 더 감쌉니다.
    - `As.WRAPPER_ARRAY`: `[ "타입명", { ...실제 데이터... } ]` 형태의 2개짜리 배열로 감쌉니다.
  - `property`: 타입 식별자를 저장할 JSON 필드명을 지정합니다. (예: `"type"`)
  - `visible`: `true`로 설정하면 타입 식별용 프로퍼티 값을 역직렬화된 객체의 필드에도 그대로 매핑합니다. (기본값 `false`)
  - `defaultImpl`: JSON에 타입 정보가 없거나 일치하는 타입을 찾지 못했을 때 사용할 기본 구현 클래스를 지정합니다.

**`@JsonSubTypes(...)`**

- `@JsonTypeInfo(use = Id.NAME, ...)`와 함께 사용되며, 논리적 타입 이름(name)과 실제 하위 클래스(Java/Kotlin class) 간의 매핑 테이블을 정의하는 어노테이션입니다.
- 부모 클래스에 선언하며, 내부에 여러 개의 `@JsonSubTypes.Type` 항목을 배열로 나열합니다.
- `@JsonSubTypes.Type`의 주요 파라미터:
  - `value`: 매핑 대상이 되는 하위 클래스(`::class` 또는 `.class`)를 지정합니다.
  - `name`: 해당 하위 클래스에 대응하는 논리적 이름(문자열)을 지정합니다. `@JsonTypeInfo`의 `property`에 지정된 필드 값과 매칭됩니다.
  - `names`: 하나의 클래스에 여러 개의 별칭(이름)을 매핑하고 싶을 때 사용하는 문자열 배열입니다. (Jackson 2.12+)
- 동작 방식:
  - **직렬화 시**: 객체의 실제 런타임 클래스를 보고, 매핑 테이블에서 일치하는 `name`을 찾아 JSON의 타입 필드 값으로 씁니다.
  - **역직렬화 시**: JSON의 타입 필드 값을 읽고, 매핑 테이블에서 일치하는 `value`(클래스)를 찾아 해당 타입의 인스턴스로 생성합니다.

## 다형성(Polymorphic) JSON 직렬화/역직렬화 예제

### 1. 클래스 정의

```kotlin
import com.fasterxml.jackson.annotation.JsonProperty
import com.fasterxml.jackson.annotation.JsonSubTypes
import com.fasterxml.jackson.annotation.JsonTypeInfo

// 부모 타입: 다형성의 "창구" 역할
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.EXISTING_PROPERTY, property = "type")
@JsonSubTypes(
  JsonSubTypes.Type(value = Dog::class, name = AnimalType.Names.DOG),
  JsonSubTypes.Type(value = Cat::class, name = AnimalType.Names.CAT),
)
sealed class Animal(open val type: AnimalType)

// 하위 타입 1: 강아지
data class Dog(
  @JsonProperty("breed")
  val breed: String,
  override val type: AnimalType = AnimalType.DOG,
) : Animal(type)

// 하위 타입 2: 고양이
data class Cat(
  @JsonProperty("is_indoor")
  val isIndoor: Boolean,
  override val type: AnimalType = AnimalType.CAT,
) : Animal(type)

enum class AnimalType {
  DOG, CAT;

  object Names {
    const val DOG = "DOG"
    const val CAT = "CAT"
  }
}
```

### 2. 사용하는 쪽 (다형성 필드를 가진 클래스)

```kotlin
data class Zoo(
  val animals: List<Animal>,
)
```

### 3. 직렬화 (객체 → JSON)

```kotlin
val zoo = Zoo(
  animals = listOf(
    Dog(breed = "진돗개"),
    Cat(isIndoor = true),
  )
)

val json = objectMapper.writeValueAsString(zoo)
println(json)
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

### 4. 역직렬화 (JSON → 객체)

```kotlin
val json = """
{
  "animals": [
    { "type": "DOG", "breed": "진돗개" },
    { "type": "CAT", "is_indoor": true }
  ]
}
"""

val zoo: Zoo = objectMapper.readValue(json, Zoo::class.java)
```

**결과:**
```kotlin
Zoo(
  animals = listOf(
    Dog(breed = "진돗개", type = AnimalType.DOG),
    Cat(isIndoor = true, type = AnimalType.CAT),
  )
)
```

- Jackson은 `animals` 리스트의 각 원소가 선언상 `Animal` 타입(추상)이라 그 자체로는 어떤 구체 클래스를 만들어야 할지 모릅니다.
- 이때 JSON에 있는 `"type": "DOG"` 값을 읽고, `@JsonSubTypes` 매핑 테이블에서 `"DOG"` → `Dog::class`를 찾아 **`Dog` 인스턴스로 역직렬화**합니다.
- `"type": "CAT"`이면 `Cat::class`로 매핑되어 `Cat` 인스턴스가 생성됩니다.