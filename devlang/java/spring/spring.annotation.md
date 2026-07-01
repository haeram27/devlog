# spring annotations

1. [spring annotations](#spring-annotations)
   1. [lombok](#lombok)
      1. [@AllArgsConstructor](#allargsconstructor)
      2. [@NoArgsConstructor](#noargsconstructor)
      3. [@RequiredArgsConstructor](#requiredargsconstructor)
      4. [@Getter](#getter)
      5. [@Setter](#setter)
         1. [@Getter, @Setter 함께 사용](#getter-setter-함께-사용)
         2. [@Getter/@Setter의 추가 옵션](#gettersetter의-추가-옵션)
      6. [@Data](#data)
         1. [자동 생성 메소드](#자동-생성-메소드)
         2. [사용 예시](#사용-예시)
         3. [@Data와 @Builder 함께 사용](#data와-builder-함께-사용)
         4. [@Data 사용 시 주의점](#data-사용-시-주의점)
      7. [@Slf4j](#slf4j)
      8. [@ToString](#tostring)
      9. [@Builder](#builder)
         1. [자동 생성 메소드](#자동-생성-메소드-1)
         2. [사용 예시](#사용-예시-1)
         3. [@Builder와 기본값](#builder와-기본값)
         4. [@Builder의 장점](#builder의-장점)
   2. [junit5](#junit5)
      1. [@TEST](#test)
      2. [@EnabledIfEnvironmentVariable](#enabledifenvironmentvariable)


## lombok

lombok은 compile 타임에 보일러 플레이트 코드를 자동 생성하는 용도의 라이브러리 이다.

### @AllArgsConstructor

- 주요 대상: 클래스
- 기능: 클래스의 모든 필드를 매개변수로 받는 생성자를 생성
- 사용 목적: 주로 의존성 주입이나 객체 생성 시 모든 필드를 초기화해야 할 때 사용

### @NoArgsConstructor

- 주요 대상: 클래스
- 기능: 기본 생성자(매개변수가 없는 생성자)를 생성
- 사용 목적: 주로 JPA 같은 ORM 프레임워크에서 엔티티를 생성할 때 필요하거나, 빈 객체를 생성해야 할 때 사용

### @RequiredArgsConstructor

- 주요 대상: 클래스
- 기능: `private final` 속성의 필드를 매개 변수로 받는 생성자를 생성
- 사용 목적: 필요한 필드만 매개변수로 받는 생성자를 생성

### @Getter

- 주요 대상: 클래스 또는 필드
- 기능: 클래스의 모든 필드(또는 지정된 필드)에 대한 getter 메소드를 자동 생성
- 자동 생성 메소드: 각 필드 `field`에 대해 `getField()` 메소드 생성 (boolean 필드는 `isField()`)
- 사용 목적: 수동으로 getter를 작성하는 보일러 플레이트 코드 제거

```java
@Getter
public class User {
    private String name;
    private int age;
    private boolean active;
}

// 자동 생성되는 메소드:
// public String getName() { return this.name; }
// public int getAge() { return this.age; }
// public boolean isActive() { return this.active; }  // boolean은 is 접두어
```

### @Setter

- 주요 대상: 클래스 또는 필드
- 기능: 클래스의 모든 필드(또는 지정된 필드)에 대한 setter 메소드를 자동 생성
- 자동 생성 메소드: 각 필드 `field`에 대해 `setField(Type field)` 메소드 생성
- 사용 목적: 수동으로 setter를 작성하는 보일러 플레이트 코드 제거

```java
@Setter
public class User {
    private String name;
    private int age;
}

// 자동 생성되는 메소드:
// public void setName(String name) { this.name = name; }
// public void setAge(int age) { this.age = age; }
```

#### @Getter, @Setter 함께 사용

```java
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class User {
    private String name;
    private int age;
    private String email;
}

// 사용
User user = new User();
user.setName("John");      // setter 호출
user.setAge(30);
System.out.println(user.getName());  // getter 호출
```

#### @Getter/@Setter의 추가 옵션

```java
@Getter
@Setter
public class Config {
    private String host;
    
    @Getter(AccessLevel.PROTECTED)  // protected 레벨의 getter만 생성
    private String password;
    
    @Setter(AccessLevel.PRIVATE)    // private 레벨의 setter만 생성
    private String secret;
}
```

### @Data

- 주요 대상: 클래스
- 기능: 다음의 롬복 어노테이션을 모두 한 번에 적용하는 편의 어노테이션
  - `@Getter`: 모든 필드의 getter 메소드 생성
  - `@Setter`: 모든 필드의 setter 메소드 생성
  - `@ToString`: toString() 메소드 생성
  - `@EqualsAndHashCode`: equals()와 hashCode() 메소드 생성
  - `@RequiredArgsConstructor`: private final 필드를 매개변수로 받는 생성자 생성
- 사용 목적: DTO(Data Transfer Object), 엔티티 등 데이터 전달용 클래스를 간결하게 작성

#### 자동 생성 메소드

```java
@Data
public class User {
    private String name;
    private int age;
    private String email;
}

// 자동 생성되는 메소드:
// 1. Getter: getName(), getAge(), getEmail()
// 2. Setter: setName(), setAge(), setEmail()
// 3. toString(): "User(name=..., age=..., email=...)" 형식
// 4. equals(): 모든 필드 값 비교
// 5. hashCode(): 모든 필드 기반 해시 생성
// 6. private final 필드가 있으면 생성자 생성
```

#### 사용 예시

```java
@Data
public class Product {
    private Long id;
    private String name;
    private double price;
    private boolean available;
}

// 사용
Product product = new Product();
product.setName("Laptop");
product.setPrice(999.99);
System.out.println(product);  // Product(id=null, name=Laptop, price=999.99, available=false)

// equals와 hashCode 자동 제공
Product product2 = new Product();
product2.setName("Laptop");
product2.setPrice(999.99);
System.out.println(product.equals(product2));  // true (같은 필드값)
```

#### @Data와 @Builder 함께 사용

```java
@Data
@Builder
public class Config {
    private String host;
    
    @Builder.Default
    private int port = 8080;
}

// 사용
Config config = Config.builder()
    .host("localhost")
    .build();

System.out.println(config);  // Config(host=localhost, port=8080)
```

#### @Data 사용 시 주의점

1. **모든 필드에 대해 생성됨**: 민감한 필드(password, secret 등)도 toString()과 equals()에 포함되므로 주의

   ```java
   @Data
   public class User {
       private String name;
       private String password;  // 위험: toString()에 노출될 수 있음
   }
   ```

2. **필드를 제외하고 싶으면 직접 메소드 작성**: @Data와 함께 해당 메소드를 명시적으로 정의하면 자동 생성 메소드를 오버라이드 가능

3. **상속 환경에서 주의**: equals()와 hashCode()가 모든 필드를 포함하므로 상속 클래스와 함께 사용할 때는 `callSuper = true` 옵션을 고려

### @Slf4j

각 class에서 slf4j의 Logger 생성 라인을 자동 구현

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class MyClass implements {}
```

`@Slf4j`는 다음의 구현을 컴파일 타임에 자동 생성한다.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public final class MyClass {
    private static final Logger LOGGER = LoggerFactory.getLogger(MyClass.class);
    ...
}
```

### @ToString

- 주요 대상: 클래스
- 기능: 클래스내 모든 필드를 human readble 형식으로 출력하는 toString() 메소드 생성

### @Builder

- 주요 대상: 클래스
- 기능: Builder 패턴을 자동으로 구현하여 유연하고 읽기 좋은 객체 생성 방식 제공
- 사용 목적: 많은 필드를 가진 객체를 안전하고 명확하게 생성하거나, 선택적 필드만 설정하고 싶을 때 사용

#### 자동 생성 메소드

```java
@Builder
public class User {
    private String name;
    private int age;
    private String email;
}

// 자동 생성되는 구조:

// 1. 정적 팩토리 메소드 (static method)
public static UserBuilder builder() { ... }

// 2. Builder 클래스 (내부 클래스)
public static class UserBuilder {
    private String name;
    private int age;
    private String email;
    
    // 각 필드마다 메소드 생성
    public UserBuilder name(String name) {
        this.name = name;
        return this;
    }
    
    public UserBuilder age(int age) {
        this.age = age;
        return this;
    }
    
    public UserBuilder email(String email) {
        this.email = email;
        return this;
    }
    
    // 최종 객체 생성
    public User build() {
        return new User(this.name, this.age, this.email);
    }
}
```

#### 사용 예시

```java
// Builder 패턴으로 객체 생성
User user = User.builder()
    .name("John")
    .age(30)
    .email("john@example.com")
    .build();

// 선택적 필드만 설정
User user2 = User.builder()
    .name("Jane")
    .build();  // age와 email은 기본값(null 또는 0)으로 설정됨
```

#### @Builder와 기본값

**기본값을 지정하지 않은 경우:**

각 필드 타입의 기본값이 자동으로 설정됩니다.

```java
@Builder
public class User {
    private String name;      // 기본값: null
    private int age;          // 기본값: 0
    private boolean active;   // 기본값: false
    private Double balance;   // 기본값: null (참조 타입)
}

// 사용
User user = User.builder()
    .name("John")
    .build();
    
// 결과: name="John", age=0, active=false, balance=null
```

**기본값을 명시적으로 지정하려면 `@Builder.Default` 사용:**

필드에 기본값을 설정하려면 `@Builder.Default` 어노테이션을 사용합니다.

```java
@Builder
public class Config {
    private String host;
    
    @Builder.Default
    private int port = 8080;
    
    @Builder.Default
    private boolean ssl = true;
}

// 사용
Config config = Config.builder()
    .host("localhost")
    // port와 ssl은 명시적 기본값으로 설정됨 (8080, true)
    .build();
```

**정리: 기본값 규칙**

| 타입 | 기본값 지정 안 함 | @Builder.Default 사용 |
|------|-----------------|----------------------|
| 참조 타입 (String, Object) | null | 명시된 값 |
| 원시 타입 (int, boolean 등) | 0 또는 false | 명시된 값 |
| 컬렉션 (List, Set 등) | null | 명시된 값 (e.g., new ArrayList<>()) |

#### @Builder의 장점

1. **유연성**: 모든 필드를 설정하거나, 일부만 설정 가능
2. **가독성**: 메소드 체인으로 순서 상관없이 명확하게 표현
3. **타입 안전성**: 컴파일 타임에 에러 감지
4. **기본값 지원**: 선택적 필드의 기본값 쉽게 설정

## junit5

### @TEST

- import org.junit.jupiter.api.AfterAll;
- import org.junit.jupiter.api.AfterEach;
- import org.junit.jupiter.api.BeforeAll;
- import org.junit.jupiter.api.BeforeEach;

### @EnabledIfEnvironmentVariable

특정 환경 변수 환경 변수가 지정된 값(정규식,regex)과 일치할 때만 JUnit 5 테스트를 실행하게 해주는 조건부 실행 어노테이션

```java
import org.junit.jupiter.api.condition.EnabledIfEnvironmentVariable;

@Test
@EnabledIfEnvironmentVariable(named = "RUN_INTEGRATION_TEST", matches = "(?i)true")
class AppIntegrationTests {
    void someTest() {
        System.out.println("이 테스트는 ENV 환경변수가 true일 때만 실행됩니다.");
    }
}
```

test run command:

```bash
RUN_INTEGRATION_TEST=test gradle :module:test --tests AppIntegrationTest.someTest --rerun-tasks --no-daemon --console=plain
```