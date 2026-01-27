# gradle 빌드 환경 설정 파일

<https://docs.gradle.org/current/userguide/build_environment.html>
<https://docs.gradle.org/current/userguide/declaring_repositories.htm>

## .properties와 .gradle 설정 파일

gradle의 주 설정 파일은 크게 gradle 프로세스 실행 환경 설정용 파일과 프로젝트 빌드 설정용 파일로 나뉘어진다.

### gradle 프로세스 실행 환경 설정(.properties)

`${GRADLE_USER_HOME}/gradle.properties` 파일은 gradle 프로세스의 실행 환경을 설정하는데 사용하며, 실행 옵션 등을 지정해 줄 수 있다.

> `GRADLE_USER_HOME=${HOME}/.gradle`

### 프로젝트 빌드 설정용 파일(.gradle)

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
|1|Command line interface|.| In the command line using -D.|
|2|gradle.properties file|GRADLE_USER_HOME|Stored in a gradle.properties file in the GRADLE_USER_HOME.|
|3|gradle.properties file|Project Root Dir|Stored in a gradle.properties file in a project directory, then its parent project’s directory up to the project’s root directory.|
|4|gradle.properties file|GRADLE_HOME|Stored in a gradle.properties file in the GRADLE_HOME, the optional Gradle installation directory.|

`Hsw 필수`: ~/.gradle/gradle.properties

```properties
org.gradle.jvmargs=-Xmx2048M
org.gradle.parallel=true

privateMavenRepositoryUrl=
```

sample: ~/.gradle/gradle.properties

```properties
systemProp.proxySet=true
systemProp.http.keepAlive=true
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=9913
systemProp.http.nonProxyHosts=localhost|*.ahnlab.com

## Set the socket timeout (default 90000 msec, recomm 300000 msec(5 min) for proxy)
systemProp.org.gradle.internal.http.connectionTimeout=300000
systemProp.org.gradle.internal.http.socketTimeout=300000
systemProp.org.gradle.internal.http.readTimeout=300000

## the number of retries  (initial included) (default 3, recomm 10)
systemProp.org.gradle.internal.repository.max.retries=10

## the initial time before retrying(default 125 msec, recomm 500 msec)
systemProp.org.gradle.internal.repository.initial.backoff=500

#org.gradle.caching=true
#org.gradle.configuration-cache=true
#org.gradle.configureondemand=true
org.gradle.jvmargs=-Xmx2048M
org.gradle.parallel=true

privateMavenRepositoryUrl=
#privateMavenRepositoryUrl=https://private.host.com/artifactory/maven-repos/
```

위치: `GRADLE_USER_HOME=${HOME}/.gradle`

- gradle 프로세스가 참조 (.properties 확장자는 프로세스 실행 환경 설정)
- gradle 실행 환경과 관련된 설정
- gradle predefined 변수 값 설정 (distributionPath, zipStorePath, networkTimeout 등)

### [gradle 실행 퍼포먼스 개선](https://docs.gradle.org/current/userguide/performance.html)

```properties
org.gradle.parallel=true
org.gradle.daemon=true
org.gradle.jvmargs=-Xmx2048M
```

### .gradle 파일에서 공통 변수 정의

- 시스템 전역 gradle properties 파일은 `~/.gradle/gradle.properties` 임
- `gradle.properties` 파일에 선언된 변수는 "init|settings|build.gradle" 파일에서 값으로 참조 가능

gradle.properties

```properties
privateMavenRepositoryUrl=https://repo1.maven.org/maven2/
```

settings.gradle

```properties
pluginManagement { repositories { maven { url privateMavenRepositoryUrl } } }
dependencyResolutionManagement { repositories { maven { url privateMavenRepositoryUrl } } }
```

### proxy 설정

```properties
systemProp.proxySet=true
systemProp.http.keepAlive=true
systemProp.http.proxyHost=host
systemProp.http.proxyPort=port
systemProp.http.proxyUser=username
systemProp.http.proxyPassword=password
systemProp.http.nonProxyHosts=localhost|*.host.com
```

## init.gradle

- 사용자 범위 gradle 설정, 시스템내 모든 gradle 프로젝트를 대상으로 공통으로 적용되는 `.gradle` 설정을 명시하는데 사용
- 초기화 스크립트라고 하며, build시에 가장 먼저 실행되는 스크립트
- 프로젝트 범위의 스크립트에 의해서 override 됨
- 다른 .gradle 파일과 마찬가지로 plugin/dependency artifactory repositories를 설정하는데 사용할 수 있음
- 다만 init.gradle에서 plugin repository 설정시 gradle.properties의 property 값을 참조하지 못하므로 거의 사용되지 않음
- .gradle 설정의 경우 보통 프로젝트 범위의 settings.gradle 부터 많이 사용하는 편

## init script를 설정할 수 있는 세곳의 지정 위치

```properties
GRADLE_USER_HOME=/home/${USER}/.gradle
GRADLE_HOME=/opt/gradle/gradle-8.3
${GRADLE_USER_HOME}/init.gradle
${GRADLE_USER_HOME}/init.d/<any-name>.gradle
${GRADLE_HOME}/init.d/<any-name>.gradle
```

`${GRADLE_USER_HOME}` 위치에 두는 대신 gradle 명령 실행시 `--init-script` 옵션으로 지정하여 빌드에 적용할 수 도 있음

```bash
gradle --init-script <path/to/init.gradle> -q tasks
```

### sample: init.gradle / settings.gradle

```gradle
startParameter.offline=false
if (startParameter.offline) {
    println "======================"
    println "gradle in offline mode"
    println "======================"
}

// plugin artifactory repositories
// https://docs.gradle.org/current/userguide/plugins.html
// this block CAN NOT read properties from gradle.properties
settingsEvaluated { settings ->
    settings.pluginManagement {
        repositories {
            maven { url https://repo1.maven.org/maven2/ }
            gradlePluginPortal()
            mavenCentral()
        }
    }
}

// dependency artifactory repositories
initscript {
    allprojects{
        repositories {
            maven { url privateMavenRepositoryUrl }
            mavenCentral()
        }
    }
}
```

offline 모드를 command 라인 옵션으로 설정 하려면 --offline 옵션을 사용한다.

#### running the build task

```bash
gradle build --offline
```

### settings.gradle

프로젝트 범위 gradle 설정

- 참고:  https://docs.gradle.org/current/userguide/plugins.html

#### 주요 사용처

- `pluginManagement`/`dependencyResolutionManagement` repositories 설정
- 프로젝트 이름 명시
- 멀티 모듈 프로젝트일 때, 프로젝트 구조 설정

sample :

```properties
pluginManagement {
    repositories {
        maven { url privateMavenRepositoryUrl }
        gradlePluginPortal()
        mavenCentral()
    }
}

dependencyResolutionManagement {
    repositories {
        maven { url privateMavenRepositoryUrl }
        mavenCentral()
    } 
}

// offline 옵션 사용시 repository 연결을 위한 network 접속 대기를 하지 않는다.
startParameter.offline=false
if (startParameter.offline) {
        println "======================"
        println "gradle in offline mode"
        println "======================"
}

rootProject.name = 'spring'
```

### build.gradle

모듈 대상 gradle 설정
모듈을 빌드하는데 필요한 모든 설정을 명시할 수 있다.

dependency repositories(maven, ive)

## 사설 maven repository 사용하는 gradle 프로젝트 설정

## gradle 설치 후 path 설정

```bash
GRADLE_HOME=/opt/gradle/gradle-8.3
export PATH=$PATH:$GRADLE_HOME/bin
```

## `${HOME}/.gradle/gradle.properties`

- `~/.gradle` 디렉토리는 gradle 명령을 실행 했을 때 생성됨
- `.gradle/gradle.properties` 파일은 없으면 수동 생성 가능

```properties
systemProp.proxySet=true
systemProp.http.keepAlive=true
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=9913
systemProp.http.nonProxyHosts=localhost|*.ahnlab.com

org.gradle.parallel=true
org.gradle.daemon=true
org.gradle.jvmargs=-Xmx2048M

mavenRepositoryUrl=https://abis.ahnlab.com/artifactory/maven-repos
```

## optional:: ${HOME}/.gradle/init.gradle

```gradle
initscript { allprojects{ repositories { maven { url privateMavenRepositoryUrl } } } }
```

## {projectroot}/settings.gradle

주의) pluginManagement 블럭은 settings.gradle 파일의 최상위에 정의 되어야 한다.

```gradle
pluginManagement {
    repositories {
        maven { url privateMavenRepositoryUrl }
        gradlePluginPortal()
        mavenCentral()
    }
}

dependencyResolutionManagement {
    repositories {
        maven { url privateMavenRepositoryUrl }
        mavenCentral()
    }
}
```

참고) gradle의 경우 mavenLocal() repository 사용을 권

## {projectroot}/build.gradle

주의) build.gradle의 repositories 설정은 settings.gradle의  dependencyResolutionManagement:repositories 설정을 override 한다.
settings.gradle의 내용을 그대로 사용할 것이라면 build.gradle의 repository block을 삭제할 수 있다.

```gradle
repositories {
    maven { url privateMavenRepositoryUrl }
}
```

## repository 설정시 url과 artifactUrls 차이

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

## gradle build performance 설정

- https://docs.gradle.org/current/userguide/build_environment.html
- https://docs.gradle.org/current/userguide/multi_project_configuration_and_execution.html#sec:configuration_on_demand
- https://docs.gradle.org/current/userguide/command_line_interface.html#sec:command_line_performance

### performance 관련 설정 파일

file - `{GRADLE_USER_HOME}/gradle.properties`

```properties
org.gradle.configureondemand=true
org.gradle.caching=true
org.gradle.jvmargs=-Xmx2048M
org.gradle.parallel=true
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

> file - settings.gradle

```properties
org.gradle.caching=true
```

#### --configuration-cache, --no-configuration-cache

- configuration stage 캐시 활성화 옵션.
- Default is off.

file - settings.gradle

```gradle
org.gradle.configuration-cache=true
```

#### --configure-on-demand, --no-configure-on-demand

- 종속성 구성
- Default is off

file - settings.gradle

```gradle
org.gradle.configureondemand=true
```

#### --max-workers

- 최대 워커 갯수
- Default is number of processors

#### --parallel, --no-parallel

- 병렬 빌드 실행
- Default is off

file - settings.gradle

```properties
org.gradle.parallel=true
```

#### --priority

- Gradle 데몬 및 Gradle 데몬이 실행하는 모든 프로세스에 대한 일정 우선 순위를 지정
- Values are normal or low
- Default is normal

#### --profile

`${buildDir}/reports/profile`디렉토리 에 높은 수준의 성능 보고서를 생성. `--scan` is preferred.

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

repository로의 network 접속이 원할하지 않는 상태이므로, offline 모드로 동작해 본다.
offline 설정 방법:
1, 명령문 옵션 설정
> $ gradle --offline

2, settings.gradle 파일에 옵션 설정
> startParameter.offline=true

## gradle tasks

syntax:

```bash
gradle <tasks...>
```

tasks = 사용 가능한 task 리스트 출력

- assemble
- test
- build = assemble+test
- clean
