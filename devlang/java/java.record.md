# java record

- JAVA 14 부터 지원
- Immutable VO(Value Obejct)를 정의하는 키워드

- 내용
  - 모든 인스턴스 변수는 final
  - public static 변수와 메소드 선언 가능

선언:

```java
public record Person (String name, Integer age) {}
```

사용:

```java
Person person = new Person("Joe", 22);
```

## static variable/method 정의

```java
public record Person (String name, Integer age) {
    public static String HELLO = "hello";
    public static Person foo(Integer age) {
        return new Person ("foo", age);
    }
}
```

## 인스턴스 생성시 자동 생성 항목

- getter()
  - 각 필드의 이름과 동일한 getter 메소드
- public constructor
  - 사용자 정의도 가능
- equals()
- hashCode()
- toString()

### Java record의 자동 생성 항목 예제

```java
public record AwsS3Properties(
    String endpoint,
    String region
) {}
```

자동 생성되는 것:

| 항목 | 생성 여부 | 형식 |
|------|----------|------|
| 생성자 | O | `AwsS3Properties(String endpoint, String region)` |
| 읽기 메서드 (accessor) | O | `endpoint()`, `region()` (JavaBean 형식 아님) |
| `equals()` | O | 모든 필드 비교 |
| `hashCode()` | O | 모든 필드 기반 |
| `toString()` | O | 모든 필드 포함 |
| setter | **X** | 생성 안 됨 (불변) |

#### 주의: `getXxx()` 형식이 아님

```java
record Person(String name) {}

Person p = new Person("John");
p.name();      // O - record accessor
p.getName();   // X - 존재하지 않음
```

#### Lombok과 비교

```java
// Lombok
@Getter
@Setter  // 필요
class Foo { private String name; }
// → getName(), setName()

// Record
record Foo(String name) {}
// → name() (getter만, setter 없음)
```

## 사용자 정의 가능 항목

- public constructor
- static variable
- static method

## 제한 사항

- 다른 클래스를 상속 불가능
- abstract class 로 정의 불가능
- static 을 제외한 다른 field와 method 는 사용자 정의할 수 없음
