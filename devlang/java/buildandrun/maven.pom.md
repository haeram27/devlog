# POM

POM은 Project Object Model의 약자로, 메이븐(Maven) 프로젝트의 모든 설정과 구성 정보를 담고 있는 XML 파일(pom.xml)을 말합니다

- maven 저장소는 `artifact` 라는 단위로 데이터를 보관하며, `artifact`는 java 패키지의 배포를 위한 파일들의 구성을 의미한다.
- artifact 에서 중요한 세 가지 파일:
  - `.pom`: 배포 패키지(`.jar`와 `.module`)에서 사용하는 의존 패키지의 목록과 버전 정보를 담고 있다. ***Gradle은 `.pom` 파일의 `<dependencyManagement`안의 `<dependendency>`에서는 버전 정보만 참고 한다. top 레벨에 정의된 `<dependendencies>`안의  `<dependendency>`에 정의된 패키지들은 로컬에 다운로드 하여 사용한다.
  - `.jar`: class와 metadata file을 묶음 java 패키지 파일
  - `.module`: java 9+ 이상부터 지원된 신규 형식의 패키지 파일 `.jar`대비 더 많은 메타 정보를 담고 있음
- maven 저장소의 artifact는 주로 java 패키지 파일(`.jar` 또는 `.module`)과 의존성 정보 BOM(`.pom`) 파일을 배포하는 저장소 이다.
- JAR(Java Archive)는 클래스와 리소스를 압축한 단순 파일 포맷(라이브러리)인 반면, 모듈(Java 9+ Module)은 module-info.java를 포함하여 의존성, 공개 범위, 구성을 명시하는 더 강력한 패키지 이다.
- maven repository 에는 BOM(.pom 파일)을 두가지 artifact 방식으로 배포하고 있다.
  - `.jar`없이 `.pom` 파일만 배포 (`platform()`으로 사용 가능), 비 일반적인 artifact 구성
    - 예: [`org.springframework.boot:spring-boot-dependencies:x.x.x` artifact](https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-dependencies/4.0.5/spring-boot-dependencies-4.0.5.pom)
  - 빈(empty) `.jar` 파일과 `.pom` 파일을 함께 배포 (`spring-boot-starter-xxx` 방식), 일반적인 artifact 구성
    - 예: [`org.springframework.boot:spring-boot-starter-web` artifact](https://repo1.maven.org/maven2/org/springframework/boot/spring-boot-starter-web/4.0.5/)

## POM 파일 형식

- `.pom` 파일은 [maven repository](https://repo1.maven.org/maven2/org/apache/logging/log4j/log4j-core/2.25.4/)나 [maven repository web page의 Files](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core/2.25.4)에서 확인 가능

- `<dependencyManagement>`나 top level `<dependencies>`가 항상 명시되는 것은 아니며, 목적에 따라 한 가지만 명시될 수 도 있음(대부분의 경우)

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
    <dependencies>
      <!-- 여기 dependency는 버전 정보만 참고(다운X) -->
      <dependency> ... </dependency>
      ...
    </dependencies>
  </dependencyManagement>
  <dependencies>
    <!-- 여기 dependency는 gradle/mvn이 다운로드 -->
    <dependency>
      <groupId>org.required.to.run</groupId>
      <artifactId>dep-package</artifactId>
      <version>${spring-framework.version}$</version>
    </dependency>
    ...
  </dependencies>
  <build>
      <!-- 빌드용 plugin artifact 관리 -->
      <pluginManagement>
        <plugins>
          <!-- 여기 plugin은 버전 정보만 참고(다운X) -->
          <plugin> ... </plugin>
          ...
        </plugins>
      </pluginManagement>

      <plugins>
        <!-- 여기 plugin는 gradle/mvn이 다운로드 -->
        <plugin> ... </plugin>
        ...
      </plugins>
    </build>
</project>

```
- `dependencyManagement`: `/dependencyManagement/dependencies/dependency` 들은 현재 및 자식 POM에게 패키지 호환 버전 정보를 전파함. 나열된 package들을 gradle/maven이 로컬에 다운로드 하지 않음. `/dependencies/dependency`에서 버전 없이 명시된 artifact들은 여기 명시된 버전을 자동 사용, 자식 POM에 상속됨

- `dependencies`: `/dependencies/dependency` 들은 gradle/maven이 로컬에 다운로드하여 실제 사용하는 artifact. 현재 패키지를 실행하는데 직접적으로 필요한 패키지 목록, 자식 POM에 상속됨

- `pluginManagement`: `/build/pluginManagement/plugins` 플러그인 설정 및 버전을 중앙 관리, 현재 및 자식 POM에서 버전 정보 없이 plugin 사용시 여기 명시된 버전 자동 사용, 자식 POM에 상속됨

- `plugins`: `/build/plugins`. 빌드시 실제 다운로드되어 사용될 플러그인을 선언, 자식 POM에게 자동 상속되므로 전체 프로젝트가 동일 plugin 환경으로 빌드 가능


## 자식 POM에 상속되는 항목
Maven의 POM 상속 구조에서 부모로부터 자식에게 전달되는 주요 항목들을 정리한 표입니다.

| 구분 | 항목 | 상속 및 전파 방식 |
|---|---|---|
| 의존성 | `<dependencies>` | 자동 상속. 자식 프로젝트에 라이브러리가 즉시 포함됨. |
| | `<dependencyManagement>` | 설정 상속. 버전/설정 정보만 전달되며, 자식에서 선언 시 적용됨. |
| 플러그인 | `<build><plugins>` | 자동 상속. 자식 빌드 시 해당 플러그인이 자동으로 실행됨. |
| | `<build><pluginManagement>` | 설정 상속. 플러그인 버전/설정만 전달되며, 자식에서 선언 시 적용됨. |
| 프로젝트 정보 | groupId, version | 상속 가능. 자식에서 생략 시 부모의 정보를 그대로 따름. |
| | properties | 자동 상속. 부모에 정의된 변수(${...})를 자식에서 자유롭게 사용. |
| | description, url, inceptionYear | 자동 상속. 프로젝트 설명 및 관련 정보가 전달됨. |
| 배포/환경 | `<repositories>` | 자동 상속. 라이브러리를 다운로드할 저장소 설정이 전달됨. |
| | `<distributionManagement>` | 자동 상속. 빌드 결과물을 배포할 원격 저장소 정보가 전달됨. |
| | `<profiles>` | 조건부 상속. 부모의 프로필 설정이 전달되나, 활성화 조건은 각자 판단. |

참고 사항:

* `<artifactId>`는 프로젝트의 고유 식별자이므로 상속되지 않으며 반드시 자식 POM에서 직접 지정해야 합니다.
* 상속을 원치 않는 특정 플러그인이나 설정이 있다면 `<inherited>false</inherited>` 태그를 사용하여 차단할 수 있습니다.

이 표 외에 특정 항목의 상속 제외 방법이나 실제 적용 순서(우선순위)에 대해 더 궁금한 점이 있으신가요?

