# properties의 객체 바인딩

1. [properties의 객체 바인딩](#properties의-객체-바인딩)
   1. [property 바인딩 객체 생성](#property-바인딩-객체-생성)
      1. [class 방식 - 기본 생성자 + getter/setter](#class-방식---기본-생성자--gettersetter)
         1. [예시](#예시)
      2. [권장: record (불변 객체) 방식 - 생성자 바인딩](#권장-record-불변-객체-방식---생성자-바인딩)
         1. [예시 (record)](#예시-record)
         2. [주의사항](#주의사항)
      3. [정리](#정리)
   2. [EnableConfigurationProperties 사용](#enableconfigurationproperties-사용)
      1. [EnableConfigurationProperties 사용 예](#enableconfigurationproperties-사용-예)
   3. [ConfigurationPropertiesScan 사용](#configurationpropertiesscan-사용)
      1. [ConfigurationPropertiesScan 사용 예](#configurationpropertiesscan-사용-예)


- property 바인딩 객체는 `@ConfigurationProperties`로 명시한다.
- `@ConfigurationProperties`로 명시된 객체 선언을 Bean으로 등록하는 것은 `@EnableConfigurationProperties` 또는 `@ConfigurationPropertiesScan` 으로 활성화한다. `@EnableConfigurationProperties` 또는 `@ConfigurationPropertiesScan`을 사용해야 `@ConfigurationProperties` 객체가 Bean으로 생성된다.
- `@EnableConfigurationProperties` 또는 `@ConfigurationPropertiesScan`을 동시에 사용하는 것을 지양하고 하나의 방식으로 통일하여 사용한다.
- `@ConfigurationProperties`가 많다면 `@ConfigurationPropertiesScan`를 사용할 것을 권장한다.

## property 바인딩 객체 생성

예제의 `application.yml` properties 정의

```yml
ceph:
  aws:
    s3:
      endpoint: http://127.0.0.1:7480
      region: us-east-1
      access-key: admin
      secret-key: admin
      path-style-access-enabled: true
```

### class 방식 - 기본 생성자 + getter/setter

- 가장 전통적이고 단순
- lombok의 Getter/Setter가 필요

#### 예시

`AwsS3Properties.java`

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
@ConfigurationProperties(prefix = "ceph.aws.s3")
public static class AwsS3Properties {
    private String endpoint = "http://127.0.0.1:7480";
    private String region = "us-east-1";
    private String accessKey = "admin";
    private String secretKey = "admin";
    private boolean pathStyleAccessEnabled = true;
}
```

### 권장: record (불변 객체) 방식 - 생성자 바인딩

- Setter 불필요
- 운영 코드에서 더 안전한 편
- Spring Boot 3에서는 단일 생성자면 별도 ConstructorBinding 없이 동작
- Java 17+면 record가 깔끔

#### 예시 (record)

`AwsS3Properties.java`

```java
import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "ceph.aws.s3")
public record AwsS3Properties(
    String endpoint,
    String region,
    String accessKey,
    String secretKey,
    boolean pathStyleAccessEnabled
) {}
```

#### 주의사항

- record는 필드 기본값을 직접 줄 수 없습니다.
- 기본값이 필요하면 application.yml에서 주는 것이 일반적입니다.

### 정리

- Lombok 없이도 완전히 가능
- 빠르게 가려면 JavaBean getter/setter 직접 작성
- 장기적으로는 record 기반 불변 바인딩 추천

## EnableConfigurationProperties 사용

`@EnableConfigurationProperties`는 `@ConfigurationProperties` 클래스를 명시적으로 스프링 빈으로 등록하고, 외부 설정 값을 바인딩하도록 활성화하는 어노테이션입니다.

**`@EnableConfigurationProperties`은 보통 `@Configuration` 클래스 내부에 `@ConfigurationProperties` 객체를 두고자 할때 사용하는 방식입니다.**

핵심 역할
1. 명시 등록: 지정한 `@ConfigurationProperties` 클래스를 빈으로 등록합니다.
2. 바인딩 대상 활성화: `@ConfigurationProperties`가 등록된 클래스가 `@ConfigurationProperties`에 지정된 `prefix` 규칙에 따라 외부 설정 바인딩 대상이 되도록 합니다.
3. 설정 객체 주입 가능: 서비스/설정 클래스에서 타입 안전하게 주입받아 사용합니다.

왜 쓰는가
1. 스캔 없이 특정 프로퍼티 클래스만 정확히 등록하고 싶을 때 유용합니다.
2. 라이브러리/모듈에서 필요한 설정 객체를 명시적으로 노출할 수 있습니다.
3. 설정 클래스 수가 적은 프로젝트에서 동작이 명확하고 추적이 쉽습니다.

주의점
1. `@ConfigurationProperties`만 붙이고 `@EnableConfigurationProperties` 또는 `@ConfigurationPropertiesScan`이 없으면 빈 등록이 되지 않습니다.
2. `@EnableConfigurationProperties`와 `@ConfigurationPropertiesScan`을 동시에 사용하면 중복 설정이 되므로 한 가지 방식으로 통일하는 것이 좋습니다.
3. 클래스 기반 바인딩에서는 setter 누락 시 바인딩 실패가 날 수 있으므로, JavaBean 방식이면 getter/setter를 모두 확인하거나 record 기반 생성자 바인딩을 사용합니다.
4. 필드명과 설정 키의 매핑 규칙(예: `access-key` -> `accessKey`)을 맞추지 않으면 값이 null/기본값으로 남을 수 있습니다.

### EnableConfigurationProperties 사용 예

```java
@Configuration
@EnableConfigurationProperties(AwsSdkConfig.AwsS3Properties.class)
public class AwsSdkConfig {

    @Bean
    public AwsCredentialsProvider cephAwsCredentialsProvider(AwsS3Properties properties) {
        return StaticCredentialsProvider.create(
            AwsBasicCredentials.create(properties.getAccessKey(), properties.getSecretKey())
        );
    }

    @ConfigurationProperties(prefix = "ceph.aws.s3")
    public record AwsS3Properties(
        String endpoint,
        String region,
        String accessKey,
        String secretKey,
        boolean pathStyleAccessEnabled
    ) {}
}
```

## ConfigurationPropertiesScan 사용

`@ConfigurationPropertiesScan`의 역할은 애플리케이션 시작 시 `@ConfigurationProperties`가 붙은 클래스를 자동으로 찾아서 application.yml 또는 application.properties 값을 바인딩하고 스프링 빈으로 등록해주는 것입니다.

**`@ConfigurationPropertiesScan`을 사용하면 별도로 `@EnableConfigurationProperties`를 사용하지 않아도 됩니다.**

핵심 역할
1. 자동 탐색: 지정한 패키지 범위에서 ConfigurationProperties 클래스를 스캔합니다.
2. 빈 등록: 찾은 클래스를 스프링 컨테이너에 등록합니다.
3. 값 바인딩: prefix 기준으로 외부 설정 값을 객체 필드에 주입합니다.

왜 쓰나
1. EnableConfigurationProperties로 클래스 하나씩 등록하지 않아도 됩니다.
2. 설정 프로퍼티 클래스가 많아질수록 관리가 쉬워집니다.

주의점
1. 스캔 범위 밖에 있으면 등록되지 않습니다.
2. 따라서 메인 클래스 패키지 구조나 basePackages 설정이 중요합니다.

한 줄 요약
ConfigurationPropertiesScan은 설정 프로퍼티 클래스를 자동 등록 + 자동 바인딩해 주는 스캔 기능입니다.

### ConfigurationPropertiesScan 사용 예

핵심은
1. 메인 클래스에 ConfigurationPropertiesScan 추가
2. 프로퍼티 클래스는 ConfigurationProperties만 유지
3. 기존 EnableConfigurationProperties는 제거 가능

Main 클래스

```java
    package com.example.properties.server;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.boot.context.properties.ConfigurationPropertiesScan;

    @SpringBootApplication
    @ConfigurationPropertiesScan(basePackages = "com.example.properties")
    public class SpringApplicationMain {
        public static void main(String[] args) {
            SpringApplication.run(SpringApplicationMain.class, args);
        }
    }
```

프로퍼티 클래스

```java
package com.example.properties.server.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
@ConfigurationProperties(prefix = "ceph.aws.s3")
public static class AwsS3Properties {
    private String endpoint = "http://127.0.0.1:7480";
    private String region = "us-east-1";
    private String accessKey = "admin";
    private String secretKey = "admin";
    private boolean pathStyleAccessEnabled = true;
}
```

설정(Configuration) 클래스에서는 Properties 클래스를 주입해서 사용

```java
@Configuration
public class AwsSdkConfig {

    @Bean
    public AwsCredentialsProvider cephAwsCredentialsProvider(AwsS3Properties properties) {
        return StaticCredentialsProvider.create(
            AwsBasicCredentials.create(properties.getAccessKey(), properties.getSecretKey())
        );
    }
}
```

정리:
1. ConfigurationPropertiesScan을 쓰면 EnableConfigurationProperties(AwsS3Properties.class)는 보통 제거해도 됩니다.
2. 단, 프로퍼티 클래스가 스캔 범위 밖에 있으면 바인딩 안 되므로 basePackages를 맞춰야 합니다.
3. 둘 다 동시에 써도 동작은 하지만 중복 설정이라 한 가지 방식으로 통일하는 게 좋습니다.