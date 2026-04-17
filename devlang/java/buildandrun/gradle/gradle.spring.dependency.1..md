# gradle에서 spring 의존성 다루기

## gardle에서 mave repository의 artifact 참조 방식

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
- maven 저장소는 `artifact` 라는 단위로 데이터를 보관하며, `artifact`는 java 패키지의 배포를 위한 파일들의 구성을 의미한다.
- artifact 에서는 세가지 타입의 파일을 배포하고 있다
  - `.pom`: 배포 패키지(`.jar`와 `.module`)에서 사용하는 의존 패키지의 목록과 버전 정보를 담고 있다. ***Gradle은 `.pom` 파일의 `<dependencyManagement`안의 `<dependendency>`에서는 버전 정보만 참고 한다. top 레벨에 정의된 `<dependendencies>`안의  `<dependendency>`에 정의된 패키지들은 로컬에 다운로드 하여 사용한다.
  - `.jar`: class와 metadata file을 묶음 java 패키지 파일
  - `.module`: java 9+ 이상부터 지원된 신규 형식의 패키지 파일 `.jar`대비 더 많은 메타 정보를 담고 있음
- maven 저장소의 artifact는 주로 java 패키지 파일(`.jar` 또는 `.module`)과 의존성 정보 BOM(`.pom`) 파일을 배포하는 저장소 이다.
- JAR(Java Archive)는 클래스와 리소스를 압축한 단순 파일 포맷(라이브러리)인 반면, 모듈(Java 9+ Module)은 module-info.java를 포함하여 의존성, 공개 범위, 구성을 명시하는 더 강력한 패키지 이다.
- `platform()`과 `spring-boot-starter-xxx`는 모두 의존성 파일(BOM == `.pom`)을 가져오는 것을 목적으로 한다.
- maven repository 에는 BOM(.pom 파일)을 두가지 artifact 방식으로 배포 하고 있다.
  - `.jar`없이 `.pom` 파일만 배포 (`platform()`으로 사용 가능), 비 일반적인 artifact 구성
    - 예: [`org.springframework.boot:spring-boot-dependencies:x.x.x` artifact](https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-dependencies/4.0.5/spring-boot-dependencies-4.0.5.pom)
  - 빈(empty) `.jar` 파일과 `.pom` 파일을 함께 배포 (`spring-boot-starter-xxx` 방식), 일반적인 artifact 구성
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

### POM 파일 형식

-`.pom` 파일은 [maven repository](https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-core/2.25.4/)나 [maven repository web page의 Files](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core/2.25.4)에서 확인 가능

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>my.package.group</groupId>
  <artifactId>my-application-library</artifactId>
  <version>4.0.5</version>
  <packaging>pom</packaging>
  <name>my-library</name>
  <description>Hello World</description>
  <url>https://spring.io/projects/spring-boot</url>
  <licenses>
    ...
  </licenses>
  <developers>
    ...
  </developers>
  <scm>
    <url>https://github.com/spring-projects/spring-boot</url>
  </scm>
  <issueManagement/>
  <properties>
    <!-- dependency 에서 사용될 version 값을 변수로 정의 -->
    <spring-framework.version>7.0.6</spring-framework.version>
    ...
  </properties>
  <dependencyManagement>
    <!-- dependencyManagement 이하의 dependency 목록에서는 버전 정보만 참고함. 목록의 패키지를 사용하고자 하면 명시된 호환 버전을 사용하도록 하는 것이 목적. -->
    <dependencies>
      <!-- 수백 개의 의존성 정의 -->
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
        <version>4.0.5</version>
      </dependency>
      ...
    </dependencies>
  </dependencyManagement>
  <dependencies>
    <!-- top level에 정의된 dependencies 이하의 dependency 패키지는 빌드 도구가 로컬에 다운로드 하여 사용. 현재 패키지를 실행하는데 직접적으로 필요한 패키지 목록-->
    <dependency>
      <groupId>org.required.to.run</groupId>
      <artifactId>dep-package</artifactId>
      <version>${spring-framework.version}$</version>
    </dependency>
    ...
  </dependencies>
  <build>
    <pluginManagement>
      <plugins>
        <!-- 빌드 도구에서 사용되는 빌드용 플러그인 패키지 -->
      </plugins>
    </pluginManagement>
  </build>
</project>

```

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
