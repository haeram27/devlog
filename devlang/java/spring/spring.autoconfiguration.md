# Autoconfiguraion(자동구성)이란
스프링을 구성하는 여러 라이브러리들은 각 라이브러리 별로 기본적으로 필요한 configuration 설정을 bean을 통해 참조한다.
라이브러리가 정상적으로 구동하려면 설정 bean이 정상적으로 등록되어 있어야 하는 것이다.
자동 구성이란 라이브러리가 필요한 설정 bean의 default 버전을 자동으로 생성해 주는 기능을 말한다.
만약 사용자가 별도의 configuration bean을 생성하지 않는다면 자동으로 defalut configuration bean을 생성하여 IOC 컨테이너에 등록한다.
spring-boot는 @AutoConfiguration 과 @ConditionalOnXXX 어노테이션을 사용하여 자동구성을 구현한다.

- **@AutoConfiguration**
   -클래스가 자동 구성용 @Configuraion이라는 것을 지정하는 어노테이션
   -이 어노테이션을 사용하는 class의 package path를 다음의 파일에 추가하여야 springboot가 정상적으로 처리한다.
  - META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
  - org.springframework.boot.autoconfigure.aop.AopAutoConfiguration

> spring-boot-autoconfigure-3.2.2.jar를 압축해제 하여 imports 파일을 살펴보면 springboot가 기본적으로 사용하는 autoconfiguration들을 확인할 수 있다.

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