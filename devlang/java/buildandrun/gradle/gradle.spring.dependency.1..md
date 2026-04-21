# gradle에서 spring 의존성 다루기

## gardle에서 maven repository의 artifact 참조 방식

```gradle
# build.gradle

dependencies {
    // platform(): BOM(.pom)에 정의된 의존성을 참고하여 라이브러리(.jar) 또는 또 다른 의존성 파일(.pom)을 다운로드
    implementation(platform("org.springframework.boot:spring-boot-dependencies:4.0.5"))

    // 버전을 명시하지 않아도 platform()에 의해 다운로드 된 BOM의 버전이 자동 적용
    // `spring-boot-starter-xxx`은 `.pom`파일을 `.jar` 형태로 감싸 배포 하는 artifactory
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework:spring-data-jpa'
}
```

- [`"org.springframework.boot:spring-boot-dependencies:4.0.5"`](https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-dependencies/4.0.5/spring-boot-dependencies-4.0.5.pom)는 maven 저장소(repository)의 artifact를 가리키는 URI 이다. `:` 구분자는 maven 저장소의 url에서 `/`로 변환되어 사용된다.
- `platform()`과 `spring-boot-starter-xxx`는 모두 의존성 파일(BOM == `.pom`)을 가져오는 것을 목적으로 한다.
- maven repository 에는 BOM(.pom 파일)을 두가지 artifact 방식으로 배포 하고 있다.
  - `.jar`없이 `.pom` 파일만 배포 (`platform()`으로 사용 가능)
    - 예: [`org.springframework.boot:spring-boot-dependencies:x.x.x` artifact](https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-dependencies/4.0.5/spring-boot-dependencies-4.0.5.pom)
  - 빈(empty) `.jar` 파일과 `.pom` 파일을 함께 배포 (`spring-boot-starter-xxx` 방식)
    - 예: [`org.springframework.boot:spring-boot-starter-web` artifact](https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-starter-web/4.0.5/)
- `platform()`은 maven repository에 직접 배포된 `.pom`파일을 다루기 위한 gradle 함수 이다.
- `implementation(platform())`은 `platform()`에 정의된 의존성을 graldle 프로젝트의 `implementation` 단계에서 참조하겠다는 의미이다.

### `platform()`과 `spring-boot-starter-xxx` 차이

Maven 저장소에서 아티팩트는 두 가지 형태로 배포

| 아티팩트 | `.jar` 존재 | `.pom` 존재 |
|---|---|---|
| `spring-boot-dependencies` | ❌ 없음 | ✅ 있음 |
| `spring-boot-starter-web` | ✅ 있음 | ✅ 있음 |

#### `platform()`이 필요한 이유

Gradle은 의존성 함수들은 기본적으로 인자로 전달되는 artifactory로 부터  **`.jar` 파일을 기대** 합니다.

- implementation `spring-boot-starter-web` → `.jar` 있음 → 정상 처리
- impolementation `spring-boot-dependencies` → `.jar` 없음 → 처리 불가

→ `platform()`으로 **"이건 JAR가 없는 버전(의존성) 관리용 POM이야"** 라고 Gradle에게 명시적으로 알려주는 것

```gradle
// .jar 없는 아티팩트 → platform() 필요
implementation(platform("org.springframework.boot:spring-boot-dependencies:4.0.5"))

// .jar 있는 아티팩트 → platform() 불필요
implementation 'org.springframework.boot:spring-boot-starter-web'
```

### Groovy DSL 주요 의존성 설정 함수들

| 함수 | 설명 |
|------|------|
| `implementation(...)` | 런타임 + 컴파일, 외부 미노출(라이브러리 생성 프로젝트인 경우) |
| `api(...)` | 런타임 + 컴파일, 외부 노출(라이브러리 생성 프로젝트인 경우) |
| `compileOnly(...)` | 컴파일만, 런타임 제외 |
| `runtimeOnly(...)` | 런타임만, 컴파일 제외 |
| `testImplementation(...)` | 테스트 전용 |

일반 앱 개발에서는 `api` vs `implementation` 차이가 없으며,
패키지(`.jar` or `.module`)을 만들어서 **다른 프로젝트에 배포**할 때만 중요한 개념

일반 Spring Boot 앱 개발의 경우 그냥 `implementation` 사용

## `platform("org.springframework.boot:spring-boot-dependencies:x.x.x"))` 상세 설명

- `platform()`은 maven repository의 단일 `.pom`파일로 구성된 artifact를 다루기 위한 gradle 함수 이다. `platform()`으로 가져온 버전 정보(`.pom` 파일)에 포함된 라이브러리들을 `dependencies{}`에서 참조할 경우 버전을 명시할 필요가 없다.

**장점:**

- 수동 버전 관리 불필요
- 호환성 보장
- 유지보수 용이 (한 곳에서 버전 제어)
- 의존성 지옥(dependency hell) 해결

### BOM 확장자 및 패키징 형식

- **확장자**: `.pom` (XML 형식 문서)
- **실제 패키지**: Maven/Gradle에서 사용하는 `.jar`와 함께 배포되지만, BOM의 핵심은 **POM(Project Object Model) 파일**
- **저장소**: Maven Central Repository 등에서 `.pom` 파일로 제공

### `platform()` 함수의 역할

`platform()`은 Gradle 5.0+에서 제공하는 **BOM 가져오기 메커니즘** 을 구현한 함수:

```gradle
dependencies {
    // 방식 1: 괄호 미사용 (권장)
    implementation platform('org.springframework.boot:spring-boot-dependencies:3.2.0')
    
    // 방식 2: 괄호 사용 (이전 방식)
    // implementation(platform('org.springframework.boot:spring-boot-dependencies:3.2.0'))
}
```

#### 정의

- Gradle 5.0+에서 제공
- BOM(Bill of Materials)을 가져오는 함수
- 인자: 라이브러리 좌표 (groupId:artifactId:version)
- 반환: BOM의 버전 정보를 프로젝트에 주입

#### 내부 동작

- platform() 함수가 BOM/POM 파일을 다운로드
- POM의 `<dependencyManagement>` 섹션 파싱
- 그 안의 모든 라이브러리 버전을 현재 프로젝트에 적용

#### 정확한 동작

| 기능 | 설명 |
|------|------|
| **버전 관리** | BOM에 정의된 모든 라이브러리의 버전을 프로젝트에 주입 |
| **버전 자동 설정** | 직접 버전 지정 안 해도 호환되는 버전 자동 적용 |
| **의존성 제약** | 버전 충돌 방지, 일관된 빌드 보장 |
| **Transitivity 차단** | 실제 JAR를 추가하지 않음 (버전 정의만) |

#### Gradle 공식 문서

- [Gradle Platform Plugin](https://docs.gradle.org/current/userguide/platforms.html)
