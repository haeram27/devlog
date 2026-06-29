# gradle 빌드 환경 설정 파일

- https://docs.gradle.org/current/userguide/build_environment.html
- https://docs.gradle.org/current/userguide/declaring_repositories.htm

## .properties와 .gradle 설정 파일

gradle의 주 설정 파일은 크게 `.properties` 파일과 `.gradle` 파일로 나뉜다.

- `.properties`은 빌드 실행 환경 변수 선언, `.gradle`은 빌드용 스크립트 파일이다. 
- `.properties` 파일은 gradle 실행시 빌드용 jvm과 스크립트(settings/build.gradle) 파일에 참조할 수 있는 system.propoery 변수를 선언한다.
- `.gradle` 파일은 DSL 스크립트 파일이며, gradle 빌드 수행시 자동 참조 되는 빌드 스크립트 파일이다.

## graldle 설정 파일의 종류

- init.gradle
- gradle.properties
- settings.gradle
- build.gradle

### gradle 프로세스 실행 환경 설정(.properties)

- `gradle.properties` 파일 위치
  - `${GRADLE_HOME}/gradle.properties`
  - `${GRADLE_USER_HOME}/gradle.properties`
  - `${workspaceFolder}/gradle.properties`
- `gradle.properties` 파일은 gradle 실행 시 환경 변수 설정하는데 사용하며, 선언된 변수는 실행 jvm 및 `settings.gradle`, `build.gradle`에서 참조된다.

> `GRADLE_USER_HOME=${HOME}/.gradle`

### 프로젝트 빌드 설정용 dsl script 파일(.gradle)

.gradle 설정 파일은 gradle 프로젝트의 빌드 관련 설정 담당하며, 사용자>프로젝트>모듈 scope로 계층적 구성을 할 수 있도록 하고 있다.

다음 세 개의 .gradle 파일을 사용할 수 있다.

- init.gradle (user scope, ${HOME}/.gradle 경로에 위치)
- settings.gradle (project scope, root 프로젝트 경로에 위치)
- build.gradle (module scope, root 프로젝트(main 모듈)과 각 sub 모듈 경로에 위치)

## gradle.properties

위치: 좁은 범위의 설정 파일이 넓은 범위의 설정을 override함
동일 설정 항목이 여러 설정에 중복 지정된 경우 우선순위에 따라 최종 적용 (낮은 번호가 최종 적용됨)

|Priority|Method|Location|Details|
|---|---|---|---|
|1|Command line interface|.| In the command line using `-D`.|
|2|gradle.properties file|Project Root Dir|Stored in a gradle.properties file in a project directory, then its parent project’s directory up to the project’s root directory.|
|3|gradle.properties file|GRADLE_USER_HOME|Stored in a gradle.properties file in the GRADLE_USER_HOME.|
|4|gradle.properties file|GRADLE_HOME|Stored in a gradle.properties file in the GRADLE_HOME, the optional Gradle installation directory.|

### User scope: `~/.gradle/gradle.properties`

- 위치: ~/.gradle/gradle.properties
- 용도: gradle 실행 환경 설정(jvm 및 predefined 변수), user defined 변수 값 설정 (distributionPath, zipStorePath, networkTimeout 등)

```properties
## https://docs.gradle.org/current/userguide/build_environment.html#sec:gradle_configuration_properties

## [PROXY SETTINGS]
## proxy on/off in gradle jvm, set true if need to enble proxy with following
systemProp.proxySet=false
## http proxy
systemProp.http.keepAlive=true
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=9913
systemProp.http.nonProxyHosts=localhost|*.internal.com
#systemProp.http.proxyUser=username
#systemProp.http.proxyPassword=password
## https Proxy (usually same as HTTP)
systemProp.https.keepAlive=true
systemProp.https.proxyHost=127.0.0.1
systemProp.https.proxyPort=9913
systemProp.https.nonProxyHosts=localhost|*.internal.com
#systemProp.https.proxyUser=username
#systemProp.https.proxyPassword=password

## CONNECTION TIMEOUT and RETRY
## set the socket timeout (default 90000 msec, recomm 300000 msec(5 min) for proxy)
systemProp.org.gradle.internal.http.connectionTimeout=300000
systemProp.org.gradle.internal.http.socketTimeout=300000
systemProp.org.gradle.internal.http.readTimeout=300000

## the number of retries  (initial included) (default 3)
systemProp.org.gradle.internal.repository.max.retries=1

## the initial time before retrying(default 125 msec, recomm 500 msec)
systemProp.org.gradle.internal.repository.initial.backoff=500

## [ENHANCE PERFORMANCE] https://docs.gradle.org/current/userguide/performance.html
# enable configuration cache, this will cache the result of configuration phase and reuse it in next build
org.gradle.configuration-cache=true
# enable parallel build tasks. useful when project is consist of multi-module, this need to be set project(s) { ... } block in settings.gradle, tasks.dependsOn(...) in build.gradle
org.gradle.parallel=true
# cache build result(.class files, jar files, etc) and reuse them in next build unless the source code is changed
org.gradle.caching=true
org.gradle.jvmargs=-Xmx2048M
org.gradle.parallel=true

## [PRIVATE REPOSITORY URL for NAT]
# if need to use private maven repository server
# privateMavenRepositoryUrl=https://repo1.maven.org/maven2/
```

### Project Scope: `{project-root}/gradle.properties`

- 위치: `{project-root}/.gradle/gradle.properties`
- 용도: 프로젝트의 setting/build.gradle에서 참조할 변수, 특히 각 library 버전의 통합 관리에 사용

```properties
# versions for dependencies and plugins
jvmVersion=25
springBootPluginVersion=3.5.10
springDependencyManagementPluginVersion=1.1.7
fooJayVersion=0.0.1
lombokVersion=1.18.42
```

### offline mode 실행

- 인터넷의 repository에 접속 할 수 없는 경우, local maven repository만 사용해야 할 때 offline mode로 실행
- offline 모드를 command 라인 옵션으로 설정 하려면 --offline 옵션을 사용한다.

방법 1.

```bash
# bash
gradle build --offline
```

방법 2.

```gradle
// settings.gradle
startParameter.offline=false
```

## init.gradle

- 초기화 스크립트라고 하며, build시에 가장 먼저 실행되는 스크립트
- init.gradle은 gradle.properties 보다 먼저 로드 되므로 property 값을 참조하지 못함
- gralde 개인 환경 설정용도 이며, 프로젝트 공통 설정을 담는 용도는 아님
  - 특정 dependency download를 위한 사설 repository 설정 등에 사용됨
- 용도: 개인 개발 환경 설정 용도
  - 프로젝트 공통 환경 설정: gradle.properties, settings.gradle, build.gradle
  - 개발 개발 환경 설정 및 보정: init.gradle

- 위치:

    ```properties
    GRADLE_USER_HOME=/home/${USER}/.gradle
    GRADLE_HOME=/opt/gradle/gradle-x.x.x
    ${GRADLE_USER_HOME}/init.gradle
    ${GRADLE_USER_HOME}/init.d/<any-name>.gradle
    ${GRADLE_HOME}/init.d/<any-name>.gradle
    ```

`${GRADLE_USER_HOME}` 위치에 두는 대신 gradle 명령 실행시 `--init-script` 옵션으로 지정하여 빌드에 적용할 수 도 있음

```bash
gradle --init-script <path/to/init.gradle> -q tasks
```

## settings.gradle

프로젝트 공통 gradle 설정

- 참고: [Gradle User Guide - plugins](https://docs.gradle.org/current/userguide/plugins.html)
- 주요 사용처
  - `pluginManagement`/`dependencyResolutionManagement` repositories(maven등 artifact 저장소) 설정
  - 프로젝트 이름(rootProject.name) 명시 가능
  - 멀티 모듈 프로젝트일 때, 프로젝트 구조 설정 (project())

### 예제: `settings.gradle`

```gradle
// plugin repositories
pluginManagement {
    repositories {
        gradlePluginPortal()
        mavenCentral()
        google()
    }
}

// dependency repositories
dependencyResolutionManagement {
    repositories {
        mavenCentral()
    }
}

// startParameter.offline=false
if (startParameter.offline) {
        println "======================"
        println "gradle in offline mode"
        println "======================"
}

rootProject.name = 'springex'
```

## build.gradle

- 모듈 대상 gradle 설정
- 위치:
  - `{project}/build.gradle`, 
  - `{project}/{module}/build.gradle`
- 용도: 모듈을 빌드하는데 필요한 모든 설정을 명시할 수 있다.
  - dependency
  - java, kotlin 버전 설정
  - task 관련 설정

- `build.gradle`에서 `repositories` 설정을 하면 `settings.gradle`의 `dependencyResolutionManagement:repositories` 설정을 override 함
- `settings.gradle`의 내용을 그대로 사용할 것이라면 `build.gradle`의 `repositories` block을 삭제할 것

## 예제

```gradle
plugins {
    id 'java'
    // https://plugins.gradle.org/plugin/org.springframework.boot
    id 'org.springframework.boot'
    // https://plugins.gradle.org/plugin/io.spring.dependency-management
    id 'io.spring.dependency-management'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

configurations {
    all {
        // exclude logback
        exclude group: 'org.springframework.boot', module: 'spring-boot-starter-logging'
    }
    compileOnly {
        extendsFrom annotationProcessor
    }
}

// module versions are automatically managed by 'io.spring.dependency-management'
dependencies {
    annotationProcessor "org.projectlombok:lombok:${lombokVersion}"
    compileOnly "org.projectlombok:lombok:${lombokVersion}"

    // ## jackson
    // jackson suport Java 8 date/time Module (o)
    implementation "com.fasterxml.jackson.dataformat:jackson-dataformat-yaml"
    implementation "com.fasterxml.jackson.datatype:jackson-datatype-jsr310"
    implementation "com.fasterxml.jackson.module:jackson-module-parameter-names"

    // ## spring
    implementation 'org.springframework.boot:spring-boot-starter-web'
    // to use async http client wrapper
    implementation 'org.springframework.boot:spring-boot-starter-log4j2'

    testAnnotationProcessor "org.projectlombok:lombok:${lombokVersion}"
    testCompileOnly "org.projectlombok:lombok:${lombokVersion}"
    testImplementation 'org.springframework.boot:spring-boot-starter-log4j2'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

// set java source compatibility for java compile task
java {
    // minimum java version to compile source code
    sourceCompatibility = JavaVersion.toVersion(jvmVersion)
    // minimum java version to run the compiled bytecode, same or bigger than sourceCompatibility
    targetCompatibility = JavaVersion.toVersion(jvmVersion)

    // foojay-resolver: Apply a specific Java toolchain to ease working on different environments.
    // foojay downloads specified JDK version if not found in .gradle/toolchains/ so gradlew can automatically setup the JDK toolchain.
    toolchain {
        languageVersion = JavaLanguageVersion.of(jvmVersion)
    }
}

// do not archive plain-jar(jar \wo dependency)
jar {
    enabled = false
}

// test library(junit) and log setting for test
test {
    useJUnitPlatform()
    testLogging {
        showStandardStreams = true
        showCauses = true
        showExceptions = true
        showStackTraces = true
        exceptionFormat = 'full'
        events "passed", "skipped", "failed", "standardOut", "standardError"
    }
}

// enable ansi color log for spring
bootRun {
    environment 'spring.output.ansi.console-available', true
}
```

### repository 설정시 url과 artifactUrls 차이

```gradle
repositories {
    maven {
        // Look for POMs and artifacts, such as JARs, here
        url "http://repo2.mycompany.com/maven2"
        // Look for artifacts here if not found at the above location
        artifactUrls "http://repo.mycompany.com/jars"
        artifactUrls "http://repo.mycompany.com/jars2"
    }
}
```

## 사설 maven repository 사용하는 gradle 프로젝트 설정

### 방법 1. init.gradle을 이용해 setting/build.gradle의 repository 설정 무시

이 방식은 init.gradle에 plugin/dependency repository 설정을 하여 setting/build.gradle의 repository를 무시하는 것이다.

- 장점: settings.gradle에 별도의 private url 사용 코드를 작성할 필요없이 깔끔하게 유지 할 수 있다는 것
- 단점: init.gradle은 gradle.properties보다 먼저 호출 되므로 gradle.properties의 변수 값을 읽을 수 없다. url 값을 하드 코드해서 사용거나 시스템 환경 변수로 설정하여 사용해야 한다.

file: `~/.gradle/init.gradle`

```gradle
settingsEvaluated { settings ->
    settings.pluginManagement {
        repositories {
            clear()
            maven {
                url = "https://repo1.maven.org/maven2/"
                // if url is NOT https
                // allowInsecureProtocol = true

                // if ahthentication is required
                // credentials {
                //     username = "user"
                //     password = "password"
                // }
            }
        }
    }

    settings.dependencyResolutionManagement {
        // ignore dependency repository in settings/build.gradle
        repositoriesMode.set(org.gradle.api.initialization.resolve.RepositoriesMode.PREFER_SETTINGS)
        // use repository following
        repositories {
            maven {
                url = "https://repo1.maven.org/maven2/"
                // if url is NOT https
                // allowInsecureProtocol = true

                // if ahthentication is required
                // credentials {
                //     username = "user"
                //     password = "password"
                // }
            }
        }
    }
}
```

`settingsEvaluated` 키워드

- `init.gradle`은 gradle이 프로젝트 설정(gradle.properties, settings.gradle, build.gradle) 평가(evaluate) 하기 전에 호출하며 호출 순서상 init.gradle의 내용과 프로젝트 설정이 겹치면 프로젝트 설정이 우선 할 수도 있다.
- `settingsEvaluated` 블럭은 프로젝트 설정 평가 이후(after) 호출되는 hook 역할을 한다. 프로젝트 설정이 마치고 나서 호출되어 프로젝트 설정을 변경 적용하는데 사용된다.

`org.gradle.api.initialization.resolve.RepositoriesMode` 상황별 선택 가이드

| 선택지 | 결과 | 추천 상황 |
|---|---|---|
| 설정 안 함 | build.gradle 설정이 우선함 | 개별 프로젝트의 자유도를 높이고 싶을 때 |
| FAIL_ON_PROJECT_REPOS | 프로젝트 설정에 repository 설정 존재 시 빌드 실패 | 전역 정책을 엄격히 강제하고 싶을 때 (기업용) |
| PREFER_SETTINGS | init.gradle 설정이 무조건 우선함 | 빌드는 유지하되 저장소만 일괄 통제하고 싶을 때 |

- 권장 사항:
  - 개인적으로 사설 저장소를 이용하고 싶다면, PREFER_SETTING 선택
  - 만약 사설 저장소(Nexus 등)를 구축해서 모든 팀원이 그것만 쓰게 하려는 의도라면, FAIL_ON_PROJECT_REPOS를 설정

### 방법 2. settings.gradle에서 private repository URL 우선 사용

이 방식은 `gradle.properties`에 private repository url을 변수로 지정하고 `settings.gradle`에 private repository url을 먼저 사용하는 것이다.

- `gradle.properties`에 private url 변수가 없으면 그 하위 지정된 repository를 차례로 시도한다.
- 단정: `setting.gradle` 파일의 구현 내용이 복잡해 지고 지저분해 보일 수 있다.

#### `${HOME}/.gradle/gradle.properties`

- `~/.gradle` 디렉토리는 gradle 명령을 실행 했을 때 생성됨
- `.gradle/gradle.properties` 파일은 없으면 수동 생성 가능

```properties
## [PRIVATE REPOSITORY URL for NAT]
# if need to use private maven repository server
privateMavenRepositoryUrl=https://repo1.maven.org/maven2/
```

#### `${project-root}/settings.gradle`

- `pluginManagement` 블럭은 settings.gradle 파일의 최상위에 정의 되어야 함
- `pluginManagement` 블럭은 '빌드 도구 확장'(플러그인) 용 repository(repository) 관리
- `dependencyResolutionManagement` 블럭은 어플리케이션의 종속성(dependency) 라이브러리 저장소(repository) 관리

```gradle
// plugin repositories
pluginManagement {
    repositories {
        if (settings.hasProperty('privateMavenRepositoryUrl') && 
            privateMavenRepositoryUrl != null && 
            !privateMavenRepositoryUrl.toString().trim().isEmpty()) {
            maven {
                url = privateMavenRepositoryUrl

                // if url is NOT https
                // allowInsecureProtocol = true

                // if ahthentication is required
                // credentials {
                //     username = "user"
                //     password = "password"
                // }
            }
        } else {
            println "pluginManagement: No private maven repository url provided, fallback to public maven repositories."
        }
        gradlePluginPortal()
        mavenCentral()
        google()
    }
}

// dependency repositories
dependencyResolutionManagement {
    repositories {
        if (settings.hasProperty('privateMavenRepositoryUrl') && 
            privateMavenRepositoryUrl != null && 
            !privateMavenRepositoryUrl.toString().trim().isEmpty()) {
            maven {
                url = privateMavenRepositoryUrl

                // if url is NOT https
                // allowInsecureProtocol = true

                // if ahthentication is required
                // credentials {
                //     username = "user"
                //     password = "password"
                // }
            }
        } else {
            println "dependencyResolutionManagement: No private maven repository url provided, fallback to public maven repositories."
        }
        mavenCentral()
    }
}
```

- mavenLocal()
  - `mavenLocal()`은 `~/.m2/repository` 위치의 로컬 maven repository를 가리킴
  - Gradle 설정에서 필요에 따라서 mavenLocal()은 사용할 수 있지만, 일반적으로 권장되지는 않음

#### `settings.gradle.kts` kotlin 버전

```kotlin
pluginManagement {
    val privateMavenRepositoryUrl = providers.gradleProperty("privateMavenRepositoryUrl").orNull
    if (privateMavenRepositoryUrl.isNullOrBlank()) {
        println("pluginManagement: No private maven repository url provided, fallback to public maven repositories.")
    }
    repositories {
        if (!privateMavenRepositoryUrl.isNullOrBlank()) {
            maven {
                url = uri(privateMavenRepositoryUrl)
            }
        }
        gradlePluginPortal()
        mavenCentral()
    }

    val kotlinVersion = providers.gradleProperty("kotlinVersion").get()
    val springBootVersion = providers.gradleProperty("springBootVersion").get()
    val fooJayVersion = providers.gradleProperty("fooJayVersion").get()
    plugins {
        id("org.jetbrains.kotlin.jvm") version kotlinVersion
        id("org.jetbrains.kotlin.plugin.spring") version kotlinVersion
        id("org.springframework.boot") version springBootVersion
        // jdk toolchain resolver plugin for jdk auto download and setup
        id("org.gradle.toolchains.foojay-resolver-convention") version fooJayVersion
    }
}

dependencyResolutionManagement {
    val privateMavenRepositoryUrl = providers.gradleProperty("privateMavenRepositoryUrl").orNull
    if (privateMavenRepositoryUrl.isNullOrBlank()) {
        println("dependencyResolutionManagement: No private maven repository url provided, fallback to public maven repositories.")
    }
    repositories {
        if (!privateMavenRepositoryUrl.isNullOrBlank()) {
            maven {
                url = uri(privateMavenRepositoryUrl)
            }
        }
        mavenCentral()
    }
}

```

## gradle build performance 설정

- https://docs.gradle.org/current/userguide/build_environment.html
- https://docs.gradle.org/current/userguide/multi_project_configuration_and_execution.html#sec:configuration_on_demand
- https://docs.gradle.org/current/userguide/command_line_interface.html#sec:command_line_performance

### performance 관련 설정 파일

file - `{GRADLE_USER_HOME}/gradle.properties`

```properties
org.gradle.configureondemand=true
org.gradle.caching=true
org.gradle.parallel=true

org.gradle.jvmargs=-Xmx2048M
```

file - `{PROJECT_ROOT}/settings.gradle`:

```properties
startParameter.offline=true
```

### performance 관련 cli 옵션

```bash
gradle clean assemble --parallel
```

#### --build-cache, --no-build-cache

빌드 캐시 활성화 옵션. Default is off.

file: settings.gradle

```properties
org.gradle.caching=true
```

#### --configuration-cache, --no-configuration-cache

- configuration stage 캐시 활성화 옵션.
- Default is off.

file: settings.gradle

```gradle
org.gradle.configuration-cache=true
```

#### --configure-on-demand, --no-configure-on-demand

- 종속성 구성
- Default is off

file: settings.gradle

```gradle
org.gradle.configureondemand=true
```

#### --max-workers

- 최대 워커 갯수
- Default is number of processors

#### --parallel, --no-parallel

- 병렬 빌드 실행
- Default is off

file: settings.gradle

```properties
org.gradle.parallel=true
```

#### --priority

- Gradle 데몬 및 Gradle 데몬이 실행하는 모든 프로세스에 대한 일정 우선 순위를 지정
- Values are normal or low
- Default is normal

#### --profile

`${buildDir}/reports/profile` 디렉토리 에 높은 수준의 성능 보고서를 생성. `--scan` is preferred.

#### --scan

Generate a build scan with detailed performance diagnostics.

#### --watch-fs, --no-watch-fs

파일 시스템 감시를 토글 합니다.
활성화되면 Gradle은 빌드 간에 파일 시스템에 대해 수집한 정보를 재사용합니다.
Default is on

|option|explanation|
|---|---|
|-g, --gradle-user-home|Specifies the Gradle user home directory<br>Defaults to ~/.gradle|
|-I, --init-script     |Specify an initialization script|
|--offline             |Execute the build without accessing network resources|
|--status              |Shows status of running and recently stopped Gradle daemon(s)|
|--stop                |Stops the Gradle daemon if it is running|

```bash
gradle test --tests '<test-class>.<test-method>'
gradle test --tests 'JunitExamApplicationTests.streamTest'
gradle test --tests 'JunitExam*.streamTest'
```

## trouble shoot

### CONFIGURING에서 0%로 1분 이상 소요…

- repository로의 network 접속이 원할하지 않는 상태이므로, offline 모드로 동작해 본다.

#### offline 설정 방법:

1. cli 명령문에 옵션으로 설정
    > gradle --offline
1. settings.gradle 파일에 옵션 설정
    > startParameter.offline=true

## gradle tasks

syntax:

```bash
gradle <tasks...>
```

### 자주 사용하는 명령

```bash
    # tasks = commands list
    $ gradle tasks --all

        Application tasks
        -----------------
        bootRun - Runs this project as a Spring Boot application.
        bootTestRun - Runs this project as a Spring Boot application using the test runtime classpath.

        Build tasks
        -----------
        assemble - Assembles the outputs of this project.
        bootBuildImage - Builds an OCI image of the application using the output of the bootJar task
        bootJar - Assembles an executable jar archive containing the main classes and their dependencies.
        build - Assembles and tests this project.
        buildDependents - Assembles and tests this project and all projects that depend on it.
        buildNeeded - Assembles and tests this project and all projects it depends on.
        classes - Assembles main classes.
        clean - Deletes the build directory.
        jar - Assembles a jar archive containing the classes of the 'main' feature.
        resolveMainClassName - Resolves the name of the application's main class.
        resolveTestMainClassName - Resolves the name of the application's test main class.
        testClasses - Assembles test classes.

    # assemble = build - text
    $ gradle assemble --refresh-dependencies
    $ gradle build -x test --refresh-dependencies

    # build = assemble + test
    $ gradle build --refresh-dependencies

    # test = run junit test
    $ gradle test --rerun-tasks --tests "ClassTests.MethodTest"

    # dependency version checked
    $ gradle dependencies
    $ gradle dependencyManagement
    $ gradle dependencyInsight --dependency httpclient5
```
