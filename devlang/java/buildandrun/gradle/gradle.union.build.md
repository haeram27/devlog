# gradle 멀티 프로젝트 통합 빌드

여러 개의 프로젝트릃 한번에 빌드 구성 하는 방법

1. `include` (멀티프로젝트)
- 하위 모듈: build.gradle(또는 `build.gradle.kts`) 필요
- 하위 모듈: settings.gradle은 보통 없음
- 루트 settings.gradle 하나가 전체 모듈 선언 담당

2. `includeBuild` (컴포지트 빌드, 현재 방식)
- 하위 프로젝트: 독립 Gradle 빌드여야 함
- 그래서 하위에 settings.gradle + build.gradle이 각각 있는 경우가 일반적

## 멀티 프로젝트 구성

핵심 단계:

1. 루트 settings.gradle 에 하위 모듈을 include로 등록
2. 모듈이 폴더 깊숙이 있으면 projectDir로 실제 경로 매핑
3. 모듈 간 의존은 includeBuild가 아니라 project(:모듈명)으로 연결

예시

1) 루트 settings.gradle
```gradle
    rootProject.name = "my-parent"
    
    include(":common", ":api", ":batch")
    
    project(":common").projectDir = file("modules/common")
    project(":api").projectDir = file("services/api")
    project(":batch").projectDir = file("jobs/batch")
```

2) 루트 build.gradle (공통 설정)
```gradle
    allprojects {
        group = "com.example"
        version = "1.0.0"
    
        repositories {
            mavenCentral()
        }
    }
    
    subprojects {
        apply plugin: "java"
    
        tasks.withType(Test).configureEach {
            useJUnitPlatform()
        }
    }
```

3) 하위 모듈 build.gradle 예시 (api 모듈)
```gradle
    dependencies {
        implementation project(":common")
        testImplementation "org.springframework.boot:spring-boot-starter-test"
    }
```

중요 포인트

1. 멀티프로젝트에서는 루트 settings.gradle 하나가 기준입니다.
2. 하위 프로젝트에 개별 settings.gradle이 있으면 충돌/혼란이 생기므로 보통 제거하거나 무시되게 정리합니다.
3. Spring Boot 모듈과 라이브러리 모듈을 분리하는 게 좋습니다.
4. 실행은 루트에서 한 번에:
   ./gradlew build
   또는 모듈별:
   ./gradlew :api:build

## 컴포지트 빌드 구성

최상위 디렉토리에 `settings.gradle`, `build.gradle`를 작성하고 하위 프로젝트들을 `includeBuild`로 collectoin을 만들어 각 프로젝트에 순차로 gradle 명령을 전달하는 방식

### 예시

settings.gradle

```gradle
// settings.gradle
rootProject.name = 'spring-union'

includeBuild 'spring-batch5'
includeBuild 'spring-clould-eureka'
includeBuild 'spring-data-mongo'
includeBuild 'spring-empty'
includeBuild 'spring-httpclient'
includeBuild 'spring-httpclient-kotlin'
includeBuild 'spring-lib-vanila'
includeBuild 'spring-quartz'
includeBuild 'spring-security'
includeBuild 'spring-service'
includeBuild 'spring-web'
includeBuild 'spring-web-kotlin'
includeBuild 'spring-kafka/spring-kafka-consumer-console'
includeBuild 'spring-kafka/spring-kafka-consumer-syslog'
includeBuild 'spring-kafka/spring-kafka-producer'
includeBuild 'spring-kafka/spring-service-logger-test'

```

build.gradle

```groovy
// build.gradle
tasks.register('assembleAll') {
    group = 'assemble'
    description = 'Assembles all included Gradle builds without running tests.'
    dependsOn gradle.includedBuilds.collect { it.task(':assemble') }
}

tasks.register('cleanAll') {
    group = 'build'
    description = 'Cleans all included Gradle builds.'
    dependsOn gradle.includedBuilds.collect { it.task(':clean') }
}

tasks.register('buildAll') {
    group = 'build'
    description = 'Builds all included Gradle builds with tests.'
    dependsOn gradle.includedBuilds.collect { it.task(':build') }
}

tasks.register('testAll') {
    group = 'verification'
    description = 'Runs tests in all included Gradle builds that expose :test.'
    dependsOn gradle.includedBuilds.collect { it.task(':test') }
}
```

실행:
- 루트에서 ./gradlew buildAll
- 루트에 wrapper가 없으면 gradle wrapper 먼저 1회 실행


