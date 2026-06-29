# gradle version catalog를 이용한 의존성 관리

Gradle에서 공식적으로 권장하는 공통 버전 관리 방식은 version catalog 이다.
Maven 진영의 BOM이 `.pom` 기반이라면, Gradle의 version catalog는 보통 `.toml` 파일을 사용하여 라이브러리와 플러그인 버전을 함께 관리한다.

이 문서는 다음 내용을 설명한다.

- 프로젝트 내부의 `libs.versions.toml` 파일을 직접 참조하는 방식
- version catalog를 별도 artifact로 발행하여 여러 프로젝트가 공통 참조하는 방식
- library 뿐 아니라 plugin 버전까지 함께 관리하는 방법
- 실제 `build.gradle.kts` 또는 `settings.gradle.kts` 에서 사용하는 예제

## version catalog란

version catalog는 의존성 좌표와 버전 별칭(alias)을 한 곳에 모아두는 Gradle의 공식 기능이다.

쉽게 말하면 다음을 한 파일에서 관리한다.

- 라이브러리 버전
- 플러그인 버전
- 의존성 alias 이름
- 여러 라이브러리를 묶는 bundle

예를 들어 기존 방식은 아래처럼 각 모듈의 빌드 파일에 버전 문자열이 반복된다.

```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web:3.5.3")
    implementation("com.fasterxml.jackson.core:jackson-databind:2.18.2")
}
```

version catalog를 쓰면 버전은 `.toml`에 모으고, 빌드 스크립트에서는 alias만 사용한다.

```kotlin
dependencies {
    implementation(libs.spring.boot.starter.web)
    implementation(libs.jackson.databind)
}
```

장점은 다음과 같다.

- 버전 변경 지점이 한 곳으로 모인다.
- 여러 모듈에서 같은 버전 정책을 강제하기 쉽다.
- plugin 버전도 함께 통합 관리할 수 있다.
- 여러 프로젝트가 같은 catalog artifact를 참조할 수 있다.

## 기본 파일 위치

가장 일반적인 방식은 프로젝트 내부에 아래 파일을 두는 것이다.

```text
gradle/libs.versions.toml
```

이 파일명과 위치를 사용하면 Gradle이 기본 catalog 이름 `libs` 로 자동 인식한다.

즉 별도 설정이 없으면 build script에서 바로 아래처럼 사용할 수 있다.

```kotlin
dependencies {
    implementation(libs.slf4j.api)
}
```

## `libs.versions.toml` 예제

아래 예제는 library 와 plugin 버전을 함께 관리하는 가장 전형적인 형태이다.

```toml
# ref: libs.versions.dotted-key
[versions]
java = "25" # libs.versions.java
spring-boot = "3.5.3"
spring-dependency-management = "1.1.7"
junit = "5.12.2"
slf4j = "2.0.17"
jackson = "2.19.1"

# ref: libs.dotted-key
[libraries]
spring-boot-starter-web = { module = "org.springframework.boot:spring-boot-starter-web", version.ref = "spring-boot" } # libs.spring.boot.starter.web
spring-boot-starter-data-jpa = { module = "org.springframework.boot:spring-boot-starter-data-jpa", version.ref = "spring-boot" }
jackson-databind = { module = "com.fasterxml.jackson.core:jackson-databind", version.ref = "jackson" }
slf4j-api = { module = "org.slf4j:slf4j-api", version.ref = "slf4j" }
junit-jupiter = { module = "org.junit.jupiter:junit-jupiter", version.ref = "junit" }

# ref: libs.plugins.dotted-key
[plugins]

spring-boot = { id = "org.springframework.boot", version.ref = "spring-boot" } # libs.plugins.spring.boot
spring-dependency-management = { id = "io.spring.dependency-management", version.ref = "spring-dependency-management" }

# ref: libs.bundles.dotted-key
[bundles]
spring-web = ["spring-boot-starter-web", "jackson-databind"] # libs.bundles.spring.web
```

각 섹션 의미는 다음과 같다.

- `[versions]`: 재사용할 버전 문자열 정의
- `[libraries]`: 실제 의존성 좌표 정의
- `[plugins]`: Gradle plugin id 와 plugin 버전 정의
- `[bundles]`: 여러 library alias를 하나로 묶음

## build.gradle.kts에서 사용하는 방법

Kotlin DSL 기준 예시는 아래와 같다.

- 참조 path 형식: `<toml-file-prefix>.<table>.<dotted-key>`
- key 이름의 `-`는 `.`으로 변경
- 테이블 이름 `libraries`는 참조 path에서 생략 가능, `vesrions`, `plugins`와 `bundles`는 반드시 표기 필요

`settings.gradle.kts`

```kotlin
val javaVersion = libs.versions.java.get()

subprojects {
    extensions.configure<JavaPluginExtension> {
        sourceCompatibility = JavaVersion.toVersion(javaVersion)
        targetCompatibility = JavaVersion.toVersion(javaVersion)
        toolchain {
            languageVersion = JavaLanguageVersion.of(javaVersion.toInt())
        }
    }
}
```

`build.gradle.kts`

```kotlin
plugins {
    alias(libs.plugins.spring.boot)
    alias(libs.plugins.spring.dependency.management)
    java
}

repositories {
    mavenCentral()
}

dependencies {
    implementation(libs.spring.boot.starter.web)
    implementation(libs.spring.boot.starter.data.jpa)
    implementation(libs.slf4j.api)

    testImplementation(libs.junit.jupiter)
}

test {
    useJUnitPlatform()
}
```

위 예제에서 중요한 점은 `libs` 객체에 직접 버전 문자열이 들어 있는 것이 아니라, `gradle/libs.versions.toml` 의 alias 정의를 통해 실제 plugin id 와 dependency coordinate 로 연결된다는 점이다.

즉 아래와 같이 해석된다.

- `alias(libs.plugins.spring.boot)` → `id("org.springframework.boot") version "3.5.3"`
- `alias(libs.plugins.spring.dependency.management)` → `id("io.spring.dependency-management") version "1.1.7"`
- `implementation(libs.spring.boot.starter.web)` → `implementation("org.springframework.boot:spring-boot-starter-web:3.5.3")`
- `implementation(libs.spring.boot.starter.data.jpa)` → `implementation("org.springframework.boot:spring-boot-starter-data-jpa:3.5.3")`
- `implementation(libs.slf4j.api)` → `implementation("org.slf4j:slf4j-api:2.0.17")`
- `testImplementation(libs.junit.jupiter)` → `testImplementation("org.junit.jupiter:junit-jupiter:5.12.2")`

### TOML 정의와 build.gradle.kts 사용의 연결 관계

예를 들어 아래 TOML 정의가 있다고 하자.

```toml
[versions]
spring-boot = "3.5.3"
slf4j = "2.0.17"

[libraries]
spring-boot-starter-web = { module = "org.springframework.boot:spring-boot-starter-web", version.ref = "spring-boot" }
slf4j-api = { module = "org.slf4j:slf4j-api", version.ref = "slf4j" }

[plugins]
spring-boot = { id = "org.springframework.boot", version.ref = "spring-boot" }
```

이때 `build.gradle.kts` 에서는 아래처럼 사용한다.

```kotlin
plugins {
    alias(libs.plugins.spring.boot)
}

dependencies {
    implementation(libs.spring.boot.starter.web)
    implementation(libs.slf4j.api)
}
```

Gradle은 이를 내부적으로 다음처럼 해석한다.

```kotlin
plugins {
    id("org.springframework.boot") version "3.5.3"
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web:3.5.3")
    implementation("org.slf4j:slf4j-api:2.0.17")
}
```

즉 핵심은 다음 두 단계이다.

1. `libs.plugins.xxx` 는 `[plugins]` 섹션의 alias를 찾는다.
2. `libs.xxx.yyy` 는 `[libraries]` 섹션의 alias를 찾고, `version.ref` 로 `[versions]` 값을 연결한다.

### plugin 버전 사용 예제

plugin alias는 `plugins {}` 블록에서만 사용한다.

```toml
[versions]
spring-boot = "3.5.3"
kotlin = "2.2.0"

[plugins]
spring-boot = { id = "org.springframework.boot", version.ref = "spring-boot" }
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
```

```kotlin
plugins {
    alias(libs.plugins.spring.boot)
    alias(libs.plugins.kotlin.jvm)
}
```

위 코드는 아래와 동일한 의미이다.

```kotlin
plugins {
    id("org.springframework.boot") version "3.5.3"
    id("org.jetbrains.kotlin.jvm") version "2.2.0"
}
```

### library 버전 사용 예제

library alias는 `dependencies {}` 블록에서 사용한다.

```toml
[versions]
jackson = "2.19.1"

[libraries]
jackson-databind = { module = "com.fasterxml.jackson.core:jackson-databind", version.ref = "jackson" }
```

```kotlin
dependencies {
    implementation(libs.jackson.databind)
}
```

위 코드는 아래와 동일한 의미이다.

```kotlin
dependencies {
    implementation("com.fasterxml.jackson.core:jackson-databind:2.19.1")
}
```

### 버전 변경이 실제 사용처에 반영되는 방식

version catalog의 장점은 실제 `build.gradle.kts` 파일을 수정하지 않고도 버전을 바꿀 수 있다는 점이다.

예를 들어 아래처럼 TOML에서만 버전을 바꾸면,

```toml
[versions]
spring-boot = "3.5.4"
```

다음 사용처들이 함께 새 버전을 참조하게 된다.

- `alias(libs.plugins.spring.boot)`
- `implementation(libs.spring.boot.starter.web)`
- `implementation(libs.spring.boot.starter.data.jpa)`

즉 동일한 `version.ref = "spring-boot"` 를 바라보는 plugin 과 library 들이 한 번에 함께 갱신된다.

### bundles 사용 예제

`[bundles]` 는 여러 개의 library alias를 하나의 이름으로 묶어서 `dependencies {}` 에서 한 번에 추가하기 위한 기능이다. `[bundles]` 는 `implementation(libs.bundles.xxx)` 처럼 `dependencies {}` 블록에서 사용한다.

예를 들어 아래처럼 TOML에 library 와 bundle 을 정의할 수 있다.

```toml
[versions]
spring-boot = "3.5.3"
jackson = "2.19.1"
slf4j = "2.0.17"

[libraries]
spring-boot-starter-web = { module = "org.springframework.boot:spring-boot-starter-web", version.ref = "spring-boot" }
jackson-databind = { module = "com.fasterxml.jackson.core:jackson-databind", version.ref = "jackson" }
slf4j-api = { module = "org.slf4j:slf4j-api", version.ref = "slf4j" }

[bundles]
web-stack = ["spring-boot-starter-web", "jackson-databind", "slf4j-api"]
```

이때 `build.gradle.kts` 에서는 아래처럼 사용한다.

```kotlin
dependencies {
    implementation(libs.bundles.web.stack)
}
```

위 코드는 개념적으로 아래와 같은 의미이다.

```kotlin
dependencies {
    implementation(libs.spring.boot.starter.web)
    implementation(libs.jackson.databind)
    implementation(libs.slf4j.api)
}
```

즉 `libs.bundles.web.stack` 하나가 여러 library alias의 묶음으로 동작한다.

실제로 최종 해석되는 dependency 좌표를 펼쳐 쓰면 아래와 비슷하다.

```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web:3.5.3")
    implementation("com.fasterxml.jackson.core:jackson-databind:2.19.1")
    implementation("org.slf4j:slf4j-api:2.0.17")
}
```

자주 쓰는 패턴은 아래와 같다.

- 웹 프로젝트 공통 묶음: web, json, logging
- 테스트 묶음: junit, mockk, assertj
- 데이터 접근 묶음: jpa, querydsl, driver

예를 들어 테스트 번들도 같은 방식으로 만들 수 있다.

```toml
[versions]
junit = "5.12.2"

[libraries]
junit-jupiter = { module = "org.junit.jupiter:junit-jupiter", version.ref = "junit" }

[bundles]
test-common = ["junit-jupiter"]
```

```kotlin
dependencies {
    testImplementation(libs.bundles.test.common)
}
```

따라서 `[bundles]` 는 library 를 구조적으로 묶어 중복 선언을 줄이는 용도이고, plugin 버전 관리를 위한 `[plugins]` 와는 역할이 다르다.

## plugin 버전도 catalog에서 관리하는 이유

많이 헷갈리는 부분이 library 와 plugin 의 차이이다.

- library: `dependencies {}` 블록에서 사용하는 일반 jar 의존성
- plugin: `plugins {}` 블록에서 사용하는 Gradle 빌드 플러그인

version catalog는 이 둘을 모두 관리할 수 있다.

예를 들어 아래 두 버전이 흩어져 있으면 관리가 어렵다.

```kotlin
plugins {
    id("org.springframework.boot") version "3.5.3"
    id("io.spring.dependency-management") version "1.1.7"
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web:3.5.3")
}
```

이것을 catalog로 모으면 plugin과 library 버전 정책을 한 군데서 통합 제어할 수 있다.

## 프로젝트 내부의 TOML을 참조하는 방식

가장 단순한 방식은 현재 프로젝트 내부의 `gradle/libs.versions.toml` 을 사용하는 것이다.

### 1. 기본 자동 인식 방식

```text
my-project/
├── settings.gradle.kts
├── build.gradle.kts
└── gradle/
    └── libs.versions.toml
```

이 위치를 사용하면 Gradle이 자동으로 `libs` catalog를 등록한다.

중요한 점은 이 자동 인식이 **기본 이름과 기본 위치의 catalog 파일** 에만 적용된다는 것이다.

즉 아래 파일은 자동 인식 대상이다.

```text
gradle/libs.versions.toml
```

반면 아래처럼 임의 이름으로 추가한 TOML 파일은 Gradle이 자동으로 등록하지 않는다.

```text
gradle/my.version.toml
gradle/test-libs.versions.toml
gradle/company.libs.toml
```

이런 파일들을 사용하려면 `settings.gradle.kts` 에서 `versionCatalogs` 로 직접 등록해야 한다.

추가 설정 없이 아래처럼 사용 가능하다.

```kotlin
plugins {
    alias(libs.plugins.spring.boot)
    java
}

dependencies {
    implementation(libs.spring.boot.starter.web)
}
```

이 예제는 다음 TOML 정의를 사용한다고 이해하면 된다.

```toml
[versions]
spring-boot = "3.5.3"

[libraries]
spring-boot-starter-web = { module = "org.springframework.boot:spring-boot-starter-web", version.ref = "spring-boot" }

[plugins]
spring-boot = { id = "org.springframework.boot", version.ref = "spring-boot" }
```

즉 `settings.gradle.kts` 에서 catalog를 등록해 두면, `build.gradle.kts` 에서는 버전 문자열 없이 alias 만으로 plugin 과 library 를 사용할 수 있다.

### 2. settings.gradle.kts에서 명시적으로 등록하는 방식

파일명을 바꾸거나 여러 catalog를 쓰고 싶으면 `settings.gradle.kts` 에서 직접 지정할 수 있다.

```kotlin
dependencyResolutionManagement {
    versionCatalogs {
        libs {
            from(files("gradle/libs.versions.toml"))
        }

        testLibs {
            from(files("gradle/test-libs.versions.toml"))
        }
    }
}
```

예를 들어 `gradle/my.version.toml` 파일을 추가로 만들었다면, 그 파일은 자동 인식되지 않으므로 아래처럼 별도 등록이 필요하다.

```kotlin
dependencyResolutionManagement {
    versionCatalogs {
        libs {
            from(files("gradle/libs.versions.toml"))
        }

        myLibs {
            from(files("gradle/my.version.toml"))
        }
    }
}
```

이렇게 등록하면 `build.gradle.kts` 에서는 catalog 이름에 맞춰 아래처럼 사용한다.

```kotlin
plugins {
    alias(myLibs.plugins.kotlin.jvm)
}

dependencies {
    implementation(myLibs.some.library)
}
```

즉 `libs` 는 기본 catalog 이름일 뿐이고, 추가 catalog는 등록할 때 지정한 이름으로 접근해야 한다.

이 방식은 다음 상황에 유용하다.

- catalog 파일명을 커스텀하고 싶을 때
- main 용, test 용 등 catalog를 분리하고 싶을 때
- 자동 등록보다 명시적 설정을 선호할 때

## 별도 BOM artifact를 생성하여 참조하는 방식

여러 프로젝트가 같은 버전 정책을 공유해야 한다면, version catalog를 별도 artifact로 발행해서 공통 참조할 수 있다.

여기서 말하는 artifact는 Maven BOM `.pom` 과는 조금 다르다.

- Maven BOM: 보통 `.pom` 기반 의존성 관리 artifact
- Gradle version catalog artifact: version catalog 정보를 담아 배포하는 Gradle용 artifact

즉 목적은 비슷하지만 포맷과 소비 방식이 다르다.

### 언제 별도 artifact 방식이 적합한가

- 여러 저장소(repo)에서 동일한 버전 정책을 써야 할 때
- 사내 표준 dependency/plugin 버전을 중앙 관리할 때
- Spring Boot, Kotlin, test, logging 버전을 공통 정책으로 강제하고 싶을 때

### catalog 발행용 프로젝트 예제

```kotlin
plugins {
    id("version-catalog")
    id("maven-publish")
}

group = "com.example.build"
version = "1.0.0"

catalog {
    versionCatalog {
        version("springBoot", "3.5.3")
        version("dependencyManagement", "1.1.7")
        version("junit", "5.12.2")

        library("spring-boot-starter-web", "org.springframework.boot", "spring-boot-starter-web").versionRef("springBoot")
        library("spring-boot-starter-data-jpa", "org.springframework.boot", "spring-boot-starter-data-jpa").versionRef("springBoot")
        library("junit-jupiter", "org.junit.jupiter", "junit-jupiter").versionRef("junit")

        plugin("spring-boot", "org.springframework.boot").versionRef("springBoot")
        plugin("spring-dependency-management", "io.spring.dependency-management").versionRef("dependencyManagement")
    }
}

publishing {
    publications {
        maven(MavenPublication) {
            from(components.versionCatalog)
        }
    }

    repositories {
        maven {
            url = uri("https://repo.example.com/maven-releases")
        }
    }
}
```

위 프로젝트를 배포하면 소비자 프로젝트는 Maven 좌표로 catalog를 가져올 수 있다.

### 소비자 프로젝트에서 catalog artifact 참조 예제

`settings.gradle.kts`

```kotlin
dependencyResolutionManagement {
    repositories {
        mavenCentral()
        maven {
            url = uri("https://repo.example.com/maven-releases")
        }
    }

    versionCatalogs {
        libs {
            from("com.example.build:company-catalog:1.0.0")
        }
    }
}
```

그 다음 `build.gradle.kts` 에서는 로컬 `libs.versions.toml` 을 쓸 때와 동일하게 사용한다.

```kotlin
plugins {
    alias(libs.plugins.spring.boot)
    alias(libs.plugins.spring.dependency.management)
    java
}

dependencies {
    implementation(libs.spring.boot.starter.web)
    testImplementation(libs.junit.jupiter)
}
```

즉 소비자 입장에서는 `from(files(...))` 대신 `from('group:artifact:version')` 을 쓴다는 점만 다르다.

## 프로젝트 내부 TOML 방식과 별도 artifact 방식 비교

| 방식 | 특징 | 적합한 경우 |
| --- | --- | --- |
| 프로젝트 내부 TOML | 설정이 단순하고 시작이 빠름 | 단일 프로젝트, 단일 저장소 |
| 별도 catalog artifact | 여러 프로젝트에 공통 정책 배포 가능 | 멀티 리포지토리, 사내 표준 빌드 정책 |

### 프로젝트 내부 TOML 방식 장점

- 설정이 가장 단순하다.
- 저장소 하나만 보면 전체 버전 정책을 이해할 수 있다.
- 외부 저장소에 catalog artifact를 발행할 필요가 없다.

### 프로젝트 내부 TOML 방식 단점

- 여러 프로젝트에 같은 정책을 복제하기 쉽다.
- 표준 버전 변경 시 프로젝트마다 반영해야 한다.

### 별도 artifact 방식 장점

- 버전 정책 중앙화가 쉽다.
- 여러 프로젝트가 동일한 library/plugin 버전을 공유한다.
- 플랫폼 팀 또는 공통 빌드 팀이 표준 catalog를 제공하기 좋다.

### 별도 artifact 방식 단점

- catalog 발행용 프로젝트와 저장소 운영이 필요하다.
- catalog 자체의 배포 버전 관리가 추가된다.

## Maven BOM 방식과의 차이

Maven BOM과 Gradle version catalog는 둘 다 버전 중앙 관리가 목적이지만 적용 범위가 다르다.

| 항목 | Maven BOM | Gradle version catalog |
| --- | --- | --- |
| 포맷 | `.pom` | `.toml` 또는 catalog component |
| 주 용도 | library 버전 관리 | library + plugin 버전 관리 |
| 참조 방식 | `platform()` | `libs.xxx`, `alias(libs.plugins.xxx)` |
| 공유 방식 | Maven artifact | 파일 참조 또는 catalog artifact |

중요한 차이는 version catalog는 plugin 버전까지 함께 관리할 수 있다는 점이다.

## 추천 기준

### 단일 프로젝트 또는 단일 멀티모듈 프로젝트

프로젝트 내부 `gradle/libs.versions.toml` 방식이 가장 단순하고 관리하기 쉽다.

### 여러 저장소에서 같은 정책을 공유해야 하는 경우

별도 catalog artifact를 발행하는 방식이 적합하다.

예를 들어 아래가 반복된다면 artifact 방식이 유리하다.

- 모든 서비스가 같은 Spring Boot 버전을 사용해야 함
- 모든 서비스가 같은 Spotless, Kotlin, JUnit plugin 버전을 사용해야 함
- 공통 로그, 테스트, DB 드라이버 버전을 중앙 관리해야 함

## 정리

- Gradle의 공식 버전 관리 방식은 version catalog 이다.
- `gradle/libs.versions.toml` 파일에서 library 와 plugin 버전을 함께 관리할 수 있다.
- 단일 프로젝트에서는 프로젝트 내부 TOML 참조 방식이 가장 단순하다.
- 여러 프로젝트에 공통 정책을 적용하려면 version catalog를 별도 artifact로 발행하여 참조할 수 있다.
- Maven BOM의 `platform()` 과 목적은 비슷하지만, version catalog는 plugin 버전까지 함께 다룰 수 있다는 점이 크다.
