# AutoConfiguration(자동구성)이란

스프링을 구성하는 여러 라이브러리들은 각 라이브러리 별로 기본적으로 필요한 configuration 설정을 bean을 통해 참조한다.
라이브러리가 정상적으로 구동하려면 설정 bean이 정상적으로 등록되어 있어야 하는 것이다.
**자동 구성이란 라이브러리가 필요한 설정 bean의 default 버전을 자동으로 생성해 주는 기능을 말한다.**
만약 사용자가 별도의 configuration bean을 생성하지 않는다면 자동으로 defalut configuration bean을 생성하여 IOC 컨테이너에 등록한다.
spring-boot는 @AutoConfiguration 과 @ConditionalOnXXX 어노테이션을 사용하여 자동구성을 구현한다.

- **@AutoConfiguration**
   -클래스가 자동 구성용 @Configuraion이라는 것을 지정하는 어노테이션
   -이 어노테이션을 사용하는 class의 package path를 다음의 파일에 추가하여야 springboot가 정상적으로 처리한다.
  - META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
  - org.springframework.boot.autoconfigure.aop.AopAutoConfiguration

> spring-boot-autoconfigure-3.2.2.jar를 압축해제 하여 imports 파일을 살펴보면 springboot가 기본적으로 사용하는 autoconfiguration들을 확인할 수 있다.

## Spring AutoConfiguration의 실제 역할

**핵심 목적: "라이브러리가 동작하는 데 필요한 Bean을 자동으로 등록해주는 것"**

### AutoConfiguration이 하는 일

| 역할 | 예시 |
|------|------|
| **필요한 Bean 자동 등록** | `DataSource`, `JdbcTemplate`, `ObjectMapper` 등 |
| **Bean 간 연결(조립) 자동화** | `TransactionManager` ↔ `DataSource` 연결 |
| **조건부 Bean 등록** | 사용자가 이미 정의했으면 등록 안 함 (`@ConditionalOnMissingBean`) |
| **기본 설정값 적용** | `application.properties`의 `spring.datasource.*` 값을 읽어 Bean에 주입 |

### `@AutoConfiguration`

1. 어디에 붙이나
- 자동 구성용 설정 클래스에 붙입니다.
- 보통 라이브러리/스타터 모듈의 설정 클래스에 사용합니다.
- 예: DataSource, RedisTemplate, ObjectMapper 같은 인프라 Bean을 조건부로 등록하는 설정 클래스.

2. 용도
- 이 클래스가 Spring Boot 자동 구성 대상임을 표시합니다.
- 애플리케이션 시작 시, Spring Boot가 자동 구성 로딩 과정에서 이 클래스를 읽어 Bean을 등록합니다.
- 주로 ConditionalOnClass, ConditionalOnMissingBean, ConditionalOnProperty 같은 조건과 함께 사용해 기본 Bean을 제공하고, 사용자가 직접 만든 Bean이 있으면 물러나는(back-off) 동작을 구현합니다.

3. 일반 앱 코드에서의 사용 여부
- 일반 업무 애플리케이션에서는 직접 붙일 일이 거의 없습니다.
- 주로 공통 라이브러리/사내 스타터/프레임워크 확장 코드를 만들 때 사용합니다.

#### 작성 패턴

1. 기본 패턴
- AutoConfiguration 클래스 안에 Bean 메서드를 만들고,
- 그 클래스 또는 메서드에 Conditional 계열 어노테이션을 붙여
- 조건이 맞을 때만 Bean이 등록되게 합니다.

2. 가장 흔한 조합
- 클래스 레벨: ConditionalOnClass, ConditionalOnProperty 등으로 큰 범위 조건
- 메서드 레벨: ConditionalOnMissingBean 등으로 개별 Bean 등록 조건
- 즉, 클래스 조건 AND 메서드 조건이 모두 만족되어야 최종 등록됩니다.

3. 중요한 보완점
- 반드시 메서드에만 조건을 붙여야 하는 것은 아닙니다.
- 클래스에만 조건을 둘 수도 있고, 필요하면 둘 다 사용합니다.
- 또한 AutoConfiguration 클래스는 AutoConfiguration.imports에 등록되어야 로딩됩니다.


### properties와의 관계

AutoConfiguration 자체가 기본값 properties를 "정의"하는 것이 아니라:

1. **기본값 정의** → `@ConfigurationProperties` + `spring-configuration-metadata.json` 에서 담당
2. **AutoConfiguration** → 그 값을 읽어서 **Bean을 생성/등록**하는 역할

```
application.properties 값
        ↓
@ConfigurationProperties (값 바인딩)
        ↓
@AutoConfiguration (Bean 생성에 활용)
        ↓
IOC 컨테이너에 Bean 등록
```

### 정리

> AutoConfiguration은 **"사용자가 별도 설정하지 않아도 라이브러리가 바로 동작할 수 있도록 필요한 Bean을 자동으로 등록해주는 메커니즘"** 입니다. properties 기본값은 그 과정에서 활용되는 수단 중 하나입니다.

## Conditional Annotations

- **@Conditional**
  - https://www.baeldung.com/spring-conditional-annotations

- **@ConditionalOnXXX**
  - 이 어노테이션은 특정 조건을 만족할 때 annotated 대상이 Bean으로 등록 되도록 한다.
  - 그러므로 @Component나 @Bean과 함께 사용되어야 한다.

- **@ConditionalOnProperty**
  - Spring Bean 생성 annotation(? extends @Component, @Bean)과 함께 사용되어
  - Property의 설정 값 상태에 따라 Spring Bean의 등록 여부를 결정한다.
  - Property는 환경 변수, java 실행 옵션, application.yml/properties 파일 등에서 설정할 수 있다.

  - Condition이 match로 판단 되면 Bean을 생성한다.
  - Condition의 match 판단은 havingValue, matchIfMissing 속성에 따라 결정된다.

## 속성

[springdocs.autoconfigure.ConditionalOnProperty](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/autoconfigure/condition/ConditionalOnProperty.html)

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@Conditional({OnPropertyCondition.class})
public @interface ConditionalOnProperty {
   String[] value() default {};
   String prefix() default "";
   String[] name() default {};
   String havingValue() default "";
   boolean matchIfMissing() default false;
}
```

- prefix:
  - name 속성 값 앞에 붙여 property key를 완성
- name:
  - property key의 이름
- havingValue: default:""
  - name에 해당하는 property key의 value가 havingValue에 지정된 값과 같을 때 condition match(yes)로 판단
  - 단, 이 attrib(havingValue)에 지정 값을 빈 문자열("")로 지정하는 경우, 실제 propoerty 값(properties 파일 내) 문자열이 "false"인 경우를 제외하고 어떤 임의의 문자열에 대해서도 match로 판단
- matchIfMissing: default:false
  - property(properties 파일 내)가 정의 되지 않았을 때 condition을 match로 판단할 지 여부
  - default 값은 false이며 이 경우 property가 정의되어 있지 않다면 mismatch로 판단하여 빈을 생성않음
  - true인 경우 property가 정의되어 있지 않은 경우에도 match로 판단하여 빈을 생성한다.

### Property와 havingValue의 값에 따른 condition match 여부 table

| Property Value | havingValue="" | havingValue="true" | havingValue="false" | havingValue="foo" |
|----------------|----------------|--------------------|---------------------|-------------------|
| "true"         | match          | match              | no-match            | no-match          |
| "false"        | no-match       | no-match           | match               | no-match          |
| "foo"          | match          | no-match           | no-match            | match             |

- HavingValue의 값으로 문자열을 지정하는 경우, property key의 value 문자열과 havingValue의 문자열 값이 동일해야 match로 판단
- havingValue 값으로 빈 문자열("")을 지정하는 경우, propoerty key의 value값이 "false"인 경우에만 mismatch로 판단하고 그외 어떤 문자열에도 match로 판단.

### 예1) property

#### MyTestConfig.java

```java
@Configuration
@ConditionalOnProperty(prefix = "condition.prop", name = "my.test.config.enabled", havingValue = "true")
@Slf4j
public class MyTestConfig {
....
```

#### applicaiton.properties

```
condition.prop.my.test.config.enabled: true
```

## `@ConditionalOnProperty` 적용 위치에 따른 차이

`@ConditionalOnProperty`는 Bean을 생성하는 어노테이션이(Configuration, Bean, Service, Component 등) 붙은 모든 위치에 사용할 수 있지만, 일반적으로 `@Configuration`과 `@Bean`에서만 사용하는 것이 관례이다.

### `@Configuration` 클래스에 붙이는 경우

```java
@Configuration
@ConditionalOnProperty(name = "feature.payment.enabled", havingValue = "true")
public class PaymentConfig {

    @Bean
    public PaymentGateway paymentGateway() { ... }

    @Bean
    public PaymentValidator paymentValidator() { ... }
}
```

- 조건이 **mismatch**이면 `PaymentConfig` 클래스 자체가 Bean으로 등록되지 않음
- 클래스 내부의 **모든 `@Bean` 메서드가 통째로 등록되지 않음**
- 관련 Bean들을 한 번에 켜고 끄는 "그룹 스위치" 역할

---

### `@Bean` 메서드에 붙이는 경우

```java
@Configuration
public class PaymentConfig {

    @Bean
    @ConditionalOnProperty(name = "feature.payment.gateway.enabled", havingValue = "true")
    public PaymentGateway paymentGateway() { ... }

    @Bean
    @ConditionalOnProperty(name = "feature.payment.validator.enabled", havingValue = "true")
    public PaymentValidator paymentValidator() { ... }
}
```

- `PaymentConfig` 클래스는 항상 Bean으로 등록됨
- 각 `@Bean` 메서드마다 **개별적으로 조건을 평가**하여 등록 여부를 결정
- Bean 단위의 세밀한 제어 가능

---

### 비교 요약

| 구분 | `@Configuration` 클래스에 적용 | `@Bean` 메서드에 적용 |
|------|-------------------------------|----------------------|
| 조건 평가 단위 | 클래스 전체 | 개별 Bean 메서드 |
| 조건 mismatch 시 | 클래스 내 모든 Bean 미등록 | 해당 Bean만 미등록 |
| 활용 목적 | 기능 단위 그룹 on/off | Bean 단위 세밀한 제어 |
| 설정 복잡도 | 단순 (property 1개) | 복잡 (property 여러 개 가능) |

> **조합 사용도 가능하다.** `@Configuration`에 클래스 레벨 조건을 걸고, 내부 `@Bean` 메서드에 추가 조건을 걸면 "클래스 조건 AND Bean 조건"이 모두 충족되어야 해당 Bean이 등록된다.