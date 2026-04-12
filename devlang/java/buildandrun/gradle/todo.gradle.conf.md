# gradle 빌드 환경 설정 파일

<https://docs.gradle.org/current/userguide/build_environment.html>
<https://docs.gradle.org/current/userguide/declaring_repositories.htm>

## .properties와 .gradle 설정 파일

gradle의 주 설정 파일은 크게 `.properties` 파일과 `.gradle` 파일로 나뉜다.
- `.properties`은 빌드 실행 환경 변수 선언, `.gradle`은 빌드용 스크립트 파일이다. 
- `.properties` 파일은 gradle 실행시 빌드용 jvm과 스크립트(settings/build.gradle) 파일에 참조할 수 있는 system.propoery 변수를 선언한다.
- `.gradle` 파일은 DSL 스크립트 파일이며, gradle 빌드 수행시 자동 참조 되는 빌드 스크립트 파일이다.

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

`my 필수`: ~/.gradle/gradle.properties

```properties
org.gradle.jvmargs=-Xmx2048M
org.gradle.parallel=true

privateMavenRepositoryUrl=
```

sample: ~/.gradle/gradle.properties

```properties
systemProp.proxySet=true
## HTTP Proxy (usually same as HTTP)
systemProp.http.keepAlive=true
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=9913
systemProp.http.nonProxyHosts=localhost|*.internal.com
#systemProp.http.proxyUser=username
#systemProp.http.proxyPassword=password
## HTTPS Proxy (usually same as HTTP)
systemProp.https.keepAlive=true
systemProp.https.proxyHost=127.0.0.1
systemProp.https.proxyPort=9913
systemProp.https.nonProxyHosts=localhost|*.internal.com
#systemProp.https.proxyUser=username
#systemProp.https.proxyPassword=password

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

## if need to use private maven repository server
privateMavenRepositoryUrl=https://repo1.maven.org/maven2/
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

- User scope gradle.properties 파일은 `~/.gradle/gradle.properties` 임
- `gradle.properties` 파일에 선언된 변수는 "init|settings|build.gradle" 파일에서 값으로 참조 가능

gradle.properties

```properties
privateMavenRepositoryUrl=https://repo1.maven.org/maven2/
```

settings.gradle

```properties
pluginManagement {
    repositories {
        if (settings.hasProperty('privateMavenRepositoryUrl') && 
            privateMavenRepositoryUrl != null && 
            !privateMavenRepositoryUrl.toString().trim().isEmpty()) {
            
            maven {
                url privateMavenRepositoryUrl
                
                // if url is NOT https
                // allowInsecureProtocol = true

                // if ahthentication is required
                // credentials {
                //     username = "user"
                //     password = "password"
                // }
            }
        }
        gradlePluginPortal()
        mavenCentral()
    }
}

dependencyResolutionManagement {
    repositories {
        if (settings.hasProperty('privateMavenRepositoryUrl') && 
            privateMavenRepositoryUrl != null && 
            !privateMavenRepositoryUrl.toString().trim().isEmpty()) {
            
            maven {
                url privateMavenRepositoryUrl
                
                // if url is NOT https
                // allowInsecureProtocol = true

                // if ahthentication is required
                // credentials {
                //     username = "user"
                //     password = "password"
                // }
            }
        }
        mavenCentral()
    }
}
```

- `allowInsecureProtocol = true`
  - repository 주소에 `https`가 아닌 `http` 사용시 insecure protocol 사용에 관한 오류가 발생하며, 해당 오류를 억제하는 옵션

### proxy 설정

```properties
systemProp.proxySet=true
## HTTP Proxy
systemProp.http.keepAlive=true
systemProp.http.proxyHost=host
systemProp.http.proxyPort=port
systemProp.http.proxyUser=username
systemProp.http.proxyPassword=password
systemProp.http.nonProxyHosts=localhost|*.host.com
## HTTPS Proxy (usually same as HTTP)
systemProp.https.keepAlive=true
systemProp.https.proxyHost=host
systemProp.https.proxyPort=port
systemProp.https.proxyUser=username
systemProp.https.proxyPassword=password
systemProp.https.nonProxyHosts=localhost|*.host.com
```

## init.gradle

- 사용자 범위 gradle 설정, 시스템내 모든 gradle 프로젝트를 대상으로 공통으로 적용되는 `.gradle` script을 명시하는데 사용
- 초기화 스크립트라고 하며, build시에 가장 먼저 실행되는 스크립트
- 프로젝트 범위의 스크립트에 의해서 override 됨
- 다른 .gradle 파일과 마찬가지로 plugin/dependency artifactory repositories를 설정하는데 사용할 수 있음
- 다만 init.gradle에서 plugin repository 설정시 gradle.properties의 property 값을 참조하지 못하므로 거의 사용되지 않음
- .gradle 설정의 경우 보통 프로젝트 범위의 settings.gradle 부터 많이 사용하는 편

## init script를 설정할 수 있는 세곳의 지정 위치

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
            maven {
                url http://insecure-protocol.private.url/maven-repo/
                allowInsecureProtocol = true
            }
            gradlePluginPortal()
            mavenCentral()
        }
    }
}

// dependency artifactory repositories
// privateMavenRepositoryUrl defined in ~/.gradle/gradle.properties
initscript {
    allprojects{
        repositories {
            if (settings.hasProperty('privateMavenRepositoryUrl') && 
                privateMavenRepositoryUrl != null && 
                !privateMavenRepositoryUrl.toString().trim().isEmpty()) {
                
                maven {
                    url privateMavenRepositoryUrl
                    
                    // if url is NOT https
                    // allowInsecureProtocol = true

                    // if ahthentication is required
                    // credentials {
                    //     username = "user"
                    //     password = "password"
                    // }
                }
            }
            mavenCentral()
        }
    }
}
```

#### offline mode build

- offline 모드를 command 라인 옵션으로 설정 하려면 --offline 옵션을 사용한다.

```bash
gradle build --offline
```

### {projectroot}/settings.gradle

프로젝트 범위 gradle 설정

- 참고:  https://docs.gradle.org/current/userguide/plugins.html

#### 주요 사용처

- `pluginManagement`/`dependencyResolutionManagement` repositories 설정
- 프로젝트 이름(rootProject.name) 명시 가능
- 멀티 모듈 프로젝트일 때, 프로젝트 구조 설정

sample :

```properties
// plugin repositories
pluginManagement {
    repositories {
        if (settings.hasProperty('privateMavenRepositoryUrl') && 
            privateMavenRepositoryUrl != null && 
            !privateMavenRepositoryUrl.toString().trim().isEmpty()) {
            maven {
                url privateMavenRepositoryUrl

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

    plugins {
        id 'org.springframework.boot' version "${springBootPluginVersion}"
        id 'io.spring.dependency-management' version "${springDependencyManagementPluginVersion}"
        // jdk toolchain resolver plugin for jdk auto download and setup
        id 'org.gradle.toolchains.foojay-resolver-convention' version "${fooJayVersion}"
    }
}

// dependency repositories
dependencyResolutionManagement {
    repositories {
        if (settings.hasProperty('privateMavenRepositoryUrl') && 
            privateMavenRepositoryUrl != null && 
            !privateMavenRepositoryUrl.toString().trim().isEmpty()) {
            maven {
                url privateMavenRepositoryUrl

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

startParameter.offline=false
if (startParameter.offline) {
        println "======================"
        println "gradle in offline mode"
        println "======================"
}

rootProject.name = 'springex'
```

### {module}/build.gradle

- 모듈 대상 gradle 설정
- 모듈을 빌드하는데 필요한 모든 설정을 명시할 수 있다.

- dependency repositories(maven, ive)

- `build.gradle`의 `repositories` 설정은 `settings.gradle`의 `dependencyResolutionManagement:repositories` 설정을 override 함
- `settings.gradle`의 내용을 그대로 사용할 것이라면 `build.gradle`의 `repositories` block을 삭제할 수 있음

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

### gradle 설치 후 path 설정

```bash
GRADLE_HOME=/opt/gradle/gradle-8.3
export PATH=$PATH:$GRADLE_HOME/bin
```

### `${HOME}/.gradle/gradle.properties`

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

privateMavenRepositoryUrl=https://repo1.maven.org/maven2/
```

### optional:: ${HOME}/.gradle/init.gradle

```gradle
initscript { allprojects{ repositories { maven { url privateMavenRepositoryUrl } } } }
```

### {projectroot}/settings.gradle

- `pluginManagement` 블럭은 settings.gradle 파일의 최상위에 정의 되어야 함
- `pluginManagement` 블럭은 '빌드 도구 확장'(플러그인) 용 repository(repository) 관리
- `dependencyResolutionManagement` 블럭은 어플리케이션의 종속성(dependency) 라이브러리 저장소(repository) 관리

```gradle
pluginManagement {
    repositories {
        if (settings.hasProperty('privateMavenRepositoryUrl') && 
            privateMavenRepositoryUrl != null && 
            !privateMavenRepositoryUrl.toString().trim().isEmpty()) {
            
            maven {
                url privateMavenRepositoryUrl
                
                // if url is NOT https
                // allowInsecureProtocol = true

                // if ahthentication is required
                // credentials {
                //     username = "user"
                //     password = "password"
                // }
            }
        }
        gradlePluginPortal()
        mavenCentral()
    }
}

dependencyResolutionManagement {
    repositories {
        if (settings.hasProperty('privateMavenRepositoryUrl') && 
            privateMavenRepositoryUrl != null && 
            !privateMavenRepositoryUrl.toString().trim().isEmpty()) {
            
            maven {
                url privateMavenRepositoryUrl
                
                // if url is NOT https
                // allowInsecureProtocol = true

                // if ahthentication is required
                // credentials {
                //     username = "user"
                //     password = "password"
                // }
            }
        }
        mavenCentral()
    }
}
```

#### mavenLocal()

- `mavenLocal()`은 `~/.m2/repository` 위치의 로컬 maven repository를 가리킴
- Gradle 설정에서 필요에 따라서 mavenLocal()은 사용할 수 있지만, 일반적으로 권장되지는 않음

### {projectroot}/build.gradle

- `build.gradle`의 `repositories` 설정은 `settings.gradle`의 `dependencyResolutionManagement:repositories` 설정을 override 함
- `settings.gradle`의 내용을 그대로 사용할 것이라면 `build.gradle`의 `repositories` block을 삭제할 수 있음

```gradle
repositories {
    maven { url privateMavenRepositoryUrl }
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

- tasks = 사용 가능한 task 리스트 출력
  - assemble
  - test
  - build = assemble+test
  - clean

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
