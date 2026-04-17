# gradle에서 springboot 프로젝트의 의존성 관리

- `dependencies { platform()` 방식 (최신 권장 방식)
- `io.spring.dependency-management` plugin 사용
- 버전 직접 입력 방식 : `spring-boot-dependencies-${springBootVersion}.pom` 파일에 명시되지 않은 라이브러리들에 사용

## 버전 직접 입력

```gradle
// build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-log4j2:2.23.1'
}
```

## `platform()` 사용 - 최신 gradle에서 권장 사용 방법

platform()으로 BOM 파일을 참조하여 지원하는 라이브러의 버전을 입력하지 않고 통합 관리하는 방법

BOM 파일에 명시 되지 않은 라이브러리들은 버전을 직접 입력하여야 함

### 예시

```gradle
// build.gradle
// springBootVersion=4.0.5
dependencies {
    annotationProcessor platform('org.springframework.boot:spring-boot-dependencies:${springBootVersion}')
    compileOnly platform('org.springframework.boot:spring-boot-dependencies:${springBootVersion}')
    annotationProcessor 'org.projectlombok:lombok'
    compileOnly 'org.projectlombok:lombok'

    // Spring Boot BOM을 가져와 버전 관리 (Spring Boot 3.x 이상 권장 방식)
    implementation platform('org.springframework.boot:spring-boot-dependencies:${springBootVersion}')
    // 버전을 명시하지 않아도 BOM에 정의된 ${springBootVerstion} 버전을 따름
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'

    testImplementation platform('org.springframework.boot:spring-boot-dependencies:${springBootVersion}')
    testImplementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}
```

- `platform(...)`: 의존성 버전 정보만 가져옴 실제 라이브러리 파일은 가져오지 않음). BOM(Bill of Materials) 파일(Maven BOM=POM, `.pom` 파일)을 인식하여, 해당 파일 안에 선언된 라이브러리 버전들을 현재 프로젝트의 dependencyManagement에 추가합니다.
- `.pom`(POM) 파일은 library `.jar` 파일에 포함되어 있지 않으며 artifact repository(maven)에서 다운로드 합니다.
- **중요**: `platform(...)` 구문은 모든 빌드 단계(implementation, testImplementation 등)에 통합 적용이 되지 않으므로 각 단계별 별도로 추가 해야 함
- implementation: 해당 라이브러리를 컴파일 및 런타임에 포함시킵니다.
- Spring Boot 스타터 활용: `org.springframework.boot:spring-boot-starter-xxx` 의존성들은 `org.springframework.boot:spring-boot-dependencies`와 같은 `${springBootVersion}` 버전을 사용하게 되며, starter들은 `${springBootVersion}`에 호환되는 라이브러리 버전(.pom파일에 명시된 library의 버전)을 다운로드 하게 된다.

## `io.spring.dependency-management` plugin 사용

`io.spring.dependency-management` 플러그인도 내부적으로 `spring-boot-dependencies-x.x.x.pom` 파일(BOM)을 다운로드하여 참조

```gradle
//--- gradle.properties
springBootVersion=4.0.5
springDependencyManagementVersion=1.1.7

//--- settings.gradle
pluginManagement {
    repositories {
        gradlePluginPortal()
        mavenCentral()
        google()
    }

    plugins {
        id 'org.springframework.boot' version "${springBootVersion}"
        id 'io.spring.dependency-management' version "${springDependencyManagementVersion}"
    }
}

//--- build.gradle
plugins {
    id 'java'
    // https://plugins.gradle.org/plugin/org.springframework.boot
    id 'org.springframework.boot'
    // https://plugins.gradle.org/plugin/io.spring.dependency-management
    id 'io.spring.dependency-management'
}
```

## `platform()` 방식과 `io.spring.dependency-management` 방식 차이

둘 다 스프링 부트의 버전 사전(BOM)을 활용한다는 목적은 같지만, 작동 방식과 기능 범위에서 큰 차이가 있습니다.
결론부터 말씀드리면, platform()은 Gradle의 기본 기능이고, io.spring.dependency-management는 별도의 플러그인입니다.

### 1. implementation platform(...) (Gradle 기본 방식)

Gradle 5.0부터 도입된 권장 방식입니다.

- 작동 방식: Maven의 BOM(Bill of Materials) 개념을 Gradle 내장 기능으로 처리합니다.
- 장점: 별도의 플러그인 설치가 필요 없고, Gradle의 고유 기능이므로 빌드 성능이 상대적으로 최적화되어 있습니다.
- 특징: platform()으로 선언된 버전들은 "권장 버전"입니다. 만약 프로젝트의 다른 곳에서 더 높은 버전을 직접 명시하면 Gradle의 충돌 해결 규칙에 따라 버전이 올라갈 수 있습니다.

### 2. io.spring.dependency-management (플러그인 방식)

스프링 팀에서 Gradle의 의존성 관리 기능이 부족하던 시절에 만든 플러그인입니다.

- 작동 방식: Maven과 거의 동일한 방식으로 동작하도록 Gradle의 동작을 재정의합니다.
- 장점 (차별점):
- 강력한 버전 고정: BOM에 정의된 버전을 매우 엄격하게 적용합니다. 직접 버전을 명시하지 않는 한, BOM의 버전을 우선시합니다.
  - 속성(Property) 제어: ext['lombok.version'] = '1.18.24' 처럼 간단한 속성 설정만으로 BOM에 정의된 특정 라이브러리의 버전만 갈아끼우는 기능이 매우 편리합니다.
- 단점: Gradle의 기본 동작 방식과 다르기 때문에, 다른 최신 Gradle 기능들과 혼용할 때 예상치 못한 설정 충돌이 발생할 수 있습니다.

### 비교

| 구분 | platform() | dependency-management 플러그인 |
|---|---|---|
| 정체성 | Gradle 순정 기능 | 외부 플러그인 |
| 버전 결정 | 다른 의존성과 협상 (최신 버전 선호) | BOM 설정 강제 (엄격함) |
| 커스터마이징 | enforcedPlatform() 등을 별도 사용 | ext 속성 선언으로 간편하게 변경 |
| 권장 여부 | 현재 Gradle에서 권장하는 방식 | 레거시나 특정 편의 기능이 필요할 때 사용 |

### 권장 방식

최신 스프링 부트 프로젝트(3.x 이상)를 새로 시작하신다면 platform() 사용을 권장합니다. 하지만 기존에 사용하던 ext 속성 기반의 편리한 버전 교체 방식이 익숙하시다면 플러그인을 계속 사용하셔도 무방합니다.

## BOM과 POM

- BOM(Bill of Materials) : 소프트 웨어를 구성하는 의존 소프트웨어들의 정보, 쉽게 말하면 소프트웨어를 만들기 위해 바이너리에 포함된 라이브러리 목록
- POM(Project Object Model): 메이븐(Maven) 빌드 시스템의 기본 설정 파일인 pom.xml을 의미, 의존성 관리 정보(프로젝트에서 사용하는 라이브러리(Jar 파일)들의 이름과 버전을 정의) 및 소프트웨어를 정의하는 여러 메타 정보(프로젝트 이름, 버전 등)를 포함 하며, maven 프로젝트에서 BOM으로서 사용됨

예: spring-boot-dependencies-4.0.5.pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>4.0.5</version>
  <packaging>pom</packaging>
  <name>spring-boot-dependencies</name>
  <description>Spring Boot Dependencies</description>
  <url>https://spring.io/projects/spring-boot</url>
  <licenses>
    <license>
      <name>Apache License, Version 2.0</name>
      <url>https://www.apache.org/licenses/LICENSE-2.0</url>
    </license>
  </licenses>
  <developers>
    <developer>
      <name>Spring</name>
      <email>ask@spring.io</email>
      <organization>VMware, Inc.</organization>
      <organizationUrl>https://www.spring.io</organizationUrl>
    </developer>
  </developers>
  <scm>
    <url>https://github.com/spring-projects/spring-boot</url>
  </scm>
  <issueManagement/>
  <properties>
    ...
    <jackson-bom.version>3.1.0</jackson-bom.version>
    <maven-source-plugin.version>3.3.1</maven-source-plugin.version>
    ...
  </properties>
  <dependencyManagement>
    <dependencies>
      ...
      <dependency>
        <groupId>tools.jackson</groupId>
        <artifactId>jackson-bom</artifactId>
        <version>${jackson-bom.version}</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>  
      ...
    </dependencies>
  </dependencyManagement>
  <build>
    <pluginManagement>
      <plugins>
        ...
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-source-plugin</artifactId>
          <version>${maven-source-plugin.version}</version>
        </plugin>
        ...
      </plugins>
    </pluginManagement>
  </build>
</project>
```

## `spring-boot-dependencies-x.x.x.pom` 파일 찾아 보기

- maven repo에서 확인
  - [maven repo artifact 경로에서 보기](https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-dependencies/4.0.5)
  - [maven repo 웹UI 사이트](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-dependencies/4.0.5) > 버전 > [Files : pom](https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-dependencies/4.0.5/spring-boot-dependencies-4.0.5.pom)

- 빌드 후 다운로드된 파일 확인

```bash
// spring boot 프로젝트 빌드 후
find ~/.gradle/caches/ -name "spring-boot-dependencies*.pom"
```