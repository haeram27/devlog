# gradle plugin 설정: `id()`와 `alias()`

Gradle Kotlin DSL에서 plugin을 적용할 때 가장 많이 보는 문법은 `plugins {}` 블록의 `id()`와 `alias()` 이다.

이 둘은 모두 plugin을 적용하는 목적은 같지만, plugin id 와 version 을 **직접 쓸지**, 아니면 **version catalog를 통해 간접 참조할지** 에서 차이가 있다.

## 한 줄 요약

- `id()` : plugin id 와 version 을 build.gradle.kts 에 직접 적는 방식
- `alias()` : `libs.versions.toml` 에 정의한 plugin alias 를 build.gradle.kts 에서 참조하는 방식

## `id()`의 목적과 용도

`id()` 는 Gradle plugin의 실제 식별자를 직접 선언하는 가장 기본적인 문법이다.

예제:

```kotlin
plugins {
    id("org.springframework.boot") version "3.5.3"
    id("io.spring.dependency-management") version "1.1.7"
}
```

의미는 다음과 같다.

- `org.springframework.boot` plugin을 적용한다.
- 그 plugin의 버전은 `3.5.3` 이다.
- plugin id 와 version 정보가 모두 현재 build script 안에 직접 적혀 있다.

### `id()`가 적합한 경우

- 단일 프로젝트라서 plugin 수가 많지 않을 때
- version catalog를 아직 사용하지 않을 때
- plugin 버전을 build.gradle.kts 안에서 직접 관리하고 싶을 때
- 예제나 샘플 프로젝트처럼 설정을 한 파일에서 바로 보여주고 싶을 때

### `id()`의 장점

- 문법이 가장 단순하다.
- plugin 정의 위치를 바로 찾기 쉽다.
- version catalog가 없어도 항상 사용 가능하다.

### `id()`의 단점

- 여러 모듈이나 여러 프로젝트에서 같은 plugin 버전이 반복되기 쉽다.
- plugin 버전을 바꿀 때 여러 파일을 수정해야 할 수 있다.
- library 버전 관리 방식과 plugin 버전 관리 방식이 분리되기 쉽다.

## `alias()`의 목적과 용도

`alias()` 는 version catalog에 정의한 plugin alias를 가져와 적용하는 문법이다.

예를 들어 `gradle/libs.versions.toml` 에 아래처럼 정의할 수 있다.

```toml
[versions]
spring-boot = "3.5.3"
spring-dependency-management = "1.1.7"

[plugins]
spring-boot = { id = "org.springframework.boot", version.ref = "spring-boot" }
spring-dependency-management = { id = "io.spring.dependency-management", version.ref = "spring-dependency-management" }
```

그 다음 `build.gradle.kts` 에서는 아래처럼 쓴다.

```kotlin
plugins {
    alias(libs.plugins.spring.boot)
    alias(libs.plugins.spring.dependency.management)
}
```

이 코드는 개념적으로 아래와 같다.

```kotlin
plugins {
    id("org.springframework.boot") version "3.5.3"
    id("io.spring.dependency-management") version "1.1.7"
}
```

즉 `alias()` 는 plugin id 와 version 을 숨기는 것이 아니라, 그것을 `libs.versions.toml` 로 이동시켜 build.gradle.kts 에서는 alias 이름만 쓰게 하는 방식이다.

### `alias()`가 적합한 경우

- 이미 version catalog를 사용하고 있을 때
- plugin 버전과 library 버전을 한 곳에서 같이 관리하고 싶을 때
- 멀티모듈 프로젝트에서 plugin 버전을 일관되게 맞추고 싶을 때
- 여러 저장소에서 공통 catalog를 공유하고 있을 때

### `alias()`의 장점

- plugin 버전 중앙 관리가 쉽다.
- 여러 모듈에서 같은 plugin alias를 재사용할 수 있다.
- library 와 plugin 버전 정책을 같은 TOML에 모을 수 있다.
- 버전 변경 지점이 줄어든다.

### `alias()`의 단점

- 실제 plugin id 와 version 이 build.gradle.kts 에 바로 보이지 않는다.
- `libs.versions.toml` 구조를 모르면 처음에는 추적이 한 단계 더 필요하다.
- version catalog를 먼저 이해해야 한다.

## `id()`와 `alias()`의 차이점

| 항목 | `id()` | `alias()` |
| --- | --- | --- |
| 정의 위치 | build.gradle.kts | libs.versions.toml + build.gradle.kts |
| plugin id | 직접 입력 | TOML에 정의 |
| plugin version | 직접 입력 | TOML의 `version.ref` 또는 version 값 사용 |
| 적합한 상황 | 단순한 프로젝트, 직접 선언 | 멀티모듈, 공통 버전 정책 관리 |
| 재사용성 | 낮음 | 높음 |

## `id()` 예제

```kotlin
plugins {
    id("java")
    id("org.springframework.boot") version "3.5.3"
    id("io.spring.dependency-management") version "1.1.7"
}
```

설명:

- `java` 는 Gradle core plugin 이므로 별도 버전 없이 적용 가능하다.
- 외부 plugin 인 `org.springframework.boot` 는 보통 버전이 필요하다.
- 이 방식은 모든 정보가 현재 파일에 직접 들어간다.

## `alias()` 예제

`gradle/libs.versions.toml`

```toml
[versions]
spring-boot = "3.5.3"
kotlin = "2.2.0"

[plugins]
spring-boot = { id = "org.springframework.boot", version.ref = "spring-boot" }
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
```

`build.gradle.kts`

```kotlin
plugins {
    alias(libs.plugins.spring.boot)
    alias(libs.plugins.kotlin.jvm)
}
```

실제 해석 결과는 아래와 같다.

```kotlin
plugins {
    id("org.springframework.boot") version "3.5.3"
    id("org.jetbrains.kotlin.jvm") version "2.2.0"
}
```

## `alias()`와 library alias의 차이

헷갈리기 쉬운 부분은 `alias()` 와 `libs.xxx` 문법이 모두 version catalog를 쓰지만, 대상이 다르다는 점이다.

- plugin alias: `plugins { alias(libs.plugins.xxx) }`
- library alias: `dependencies { implementation(libs.xxx) }`

예를 들어:

```toml
[versions]
spring-boot = "3.5.3"

[plugins]
spring-boot = { id = "org.springframework.boot", version.ref = "spring-boot" }

[libraries]
spring-boot-starter-web = { module = "org.springframework.boot:spring-boot-starter-web", version.ref = "spring-boot" }
```

```kotlin
plugins {
    alias(libs.plugins.spring.boot)
}

dependencies {
    implementation(libs.spring.boot.starter.web)
}
```

차이는 다음과 같다.

- `libs.plugins.spring.boot` 는 `[plugins]` 섹션을 참조한다.
- `libs.spring.boot.starter.web` 는 `[libraries]` 섹션을 참조한다.

## `[bundles]`와의 차이

`[bundles]` 는 plugin alias가 아니라 여러 library alias를 묶는 기능이다.

즉 아래는 가능하다.

```kotlin
dependencies {
    implementation(libs.bundles.web.stack)
}
```

하지만 아래처럼 쓰지는 않는다.

```kotlin
plugins {
    // 불가한 개념 예시
    // alias(libs.bundles.web.stack)
}
```

정리하면:

- `alias()` 는 plugin alias용
- `libs.xxx` 는 library alias용
- `libs.bundles.xxx` 는 library bundle용

## 어떤 방식을 선택할 것인가

### `id()`를 추천하는 경우

- 짧은 샘플 프로젝트
- 단일 모듈 프로젝트
- version catalog를 도입하지 않은 프로젝트

### `alias()`를 추천하는 경우

- 멀티모듈 프로젝트
- 공통 plugin 버전 정책이 필요한 프로젝트
- library 와 plugin 버전을 한 곳에서 같이 관리하고 싶은 프로젝트
- 사내 공통 catalog artifact를 참조하는 프로젝트

## 실무 기준 추천

version catalog를 이미 사용하고 있다면 plugin도 `alias()` 로 함께 관리하는 편이 일관성 면에서 좋다.

반대로 작은 프로젝트이거나 Gradle 설정을 빠르게 설명하는 문서/예제에서는 `id()` 가 더 직관적일 수 있다.

즉 정답이 하나인 것은 아니고, 기준은 다음과 같다.

- 단순성 우선: `id()`
- 중앙 관리와 재사용성 우선: `alias()`
