# spring annotations

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