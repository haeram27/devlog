# init.gradle

init.gradle은 프로젝트 파일을 건드리지 않고, Gradle 실행 시점에 공통 규칙을 주입해야 할 때 유용합니다. 

프로젝트 프로그램 공통 설정이 아닌 사용자나 회사 개발 환경의 공통 정책을 덧씌우는 용도라는 점입니다.

**파일 위치**
1. 명령행에서 직접 지정
gradle --init-script /path/to/custom.init.gradle
또는 gradle -I /path/to/custom.init.gradle

1. 사용자 전역 위치
보통 ~/.gradle/init.gradle
여러 개로 나누려면 ~/.gradle/init.d/ 아래에 둡니다.

1. Gradle 설치 디렉터리 전역 위치
보통 GRADLE_HOME/init.d/ 아래에 둡니다.
이건 해당 Gradle 설치를 쓰는 모든 빌드에 영향을 줍니다.

**적용 우선 순위와 시점**
1. 명령행으로 넘긴 init script
2. ~/.init.gradle
3. ~/.gralde/init.d 아래 스크립트들
4. GRADLE_HOME/init.d 아래 스크립트들

같은 디렉터리 안에 여러 스크립트가 있으면 파일명 알파벳 순서로 적용됩니다.

중요한 점은, init.gradle은 settings.gradle과 build.gradle보다 먼저 실행된다는 것입니다. 그래서 프로젝트 자체 스크립트를 바꾸지 않고도 저장소 설정, 자격 증명, 공통 로깅, 해상도 전략 같은 전역 규칙을 주입할 수 있습니다.

실무 판단 기준은 간단합니다.

- 이번 한 번만 강제로 넣고 싶다: 명령행의 --init-script
- 내 PC에서 항상 적용하고 싶다: ~/.gradle/init.gradle 또는 ~/.gradle/init.d/
- 특정 Gradle 배포본을 쓰는 환경 전체에 강제하고 싶다: GRADLE_HOME/init.d/

**주의할 점**

- init.gradle은 프로젝트 내부 설정보다 앞에서 개입하므로, 너무 많은 정책을 넣으면 왜 빌드가 그렇게 동작하는지 추적하기 어려워집니다.
- 팀 공통으로 반드시 재현되어야 하는 설정은 보통 프로젝트 안의 settings.gradle, build.gradle, gradle.properties에 두는 편이 맞습니다.
- 개인 환경, 사내망, 인증정보, 임시 우회 규칙은 init.gradle 쪽이 더 적절합니다.
- 한 줄로 요약하면, 프로젝트 안에 남겨야 하는 규칙은 프로젝트 스크립트에 두고, 사용자나 실행 환경에만 귀속되는 규칙은 init.gradle에 둡니다.


## 예시

### 회사 공통 사설 저장소 강제
모든 프로젝트가 사내 Nexus나 Artifactory를 먼저 보게 만들고 싶을 때 씁니다. 각 저장소의 settings.gradle, build.gradle을 일일이 수정하지 않아도 됩니다.

예:
```gradle
allprojects {
    repositories {
        maven {
            url = uri("https://nexus.company.local/repository/maven-public")
        }
        mavenCentral()
    }
}
```

이런 경우는 특히 외부 인터넷이 제한된 사내망에서 유용합니다.

### CI나 로컬에서만 자격 증명 주입
프로젝트에 아이디, 토큰, 내부 URL을 커밋하면 안 될 때 init.gradle로 주입할 수 있습니다.

예:
```gradle
allprojects {
    repositories {
        maven {
            url = uri(System.getenv("PRIVATE_MAVEN_URL"))
            credentials {
                username = System.getenv("PRIVATE_MAVEN_USER")
                password = System.getenv("PRIVATE_MAVEN_PASSWORD")
            }
        }
    }
}
```

이 패턴은 민감정보를 저장소 밖으로 빼는 데 적합합니다.

### 전사 공통 빌드 정책 적용
모든 프로젝트에 대해 특정 태스크 옵션, 로깅, 리포트, 프록시 설정을 일괄 적용할 수 있습니다.

예:
```gradle
gradle.settingsEvaluated {
    println("company init script loaded")
}

allprojects {
    tasks.withType(Test).configureEach {
        testLogging {
            events "failed", "skipped"
        }
    }
}
```

여러 팀이 제각각 설정하지 않게 통일할 때 쓸 만합니다.

### 레거시 프로젝트를 건드리지 않고 임시 보정
외부에서 받은 오래된 Gradle 프로젝트를 당장 수정하기 어렵지만, 빌드만 되게 만들어야 할 때 유용합니다.

예:
```gradle
allprojects {
    configurations.all {
        resolutionStrategy {
            force "commons-io:commons-io:2.16.1"
        }
    }
}
```

의존성 충돌을 임시로 우회하는 데 자주 씁니다. 다만 이건 장기 해법보다는 응급처치에 가깝습니다.

### 로컬 개발자 편의 설정
특정 개발자 PC에서만 프록시, 미러 저장소, 디버그 출력, 빌드 스캔 같은 옵션을 켜고 싶을 때 좋습니다.

예:
```gradle
gradle.beforeProject {
    println("Building ${it.name} on local machine")
}
```

팀 공통 설정이 아니라 개인 작업 환경 보정에 잘 맞습니다.

지금 프로젝트 기준으로 보면, 버전 중앙 관리는 init.gradle보다 gradle.properties와 settings.gradle, build.gradle에 두는 편이 맞습니다. 이유는 버전 정보가 프로젝트 자체의 계약이기 때문입니다. 반대로 사설 저장소 URL, 인증정보, 회사 네트워크 규칙처럼 프로젝트 밖 환경에 속하는 것은 init.gradle 후보입니다.

실무 기준으로 한 줄 정리하면 이렇습니다.

- 프로젝트가 알아야 하는 것: settings.gradle, build.gradle, gradle.properties
- 내 PC나 회사 환경만 알아야 하는 것: init.gradle


## 

init.gradle에서 프로젝트 스크립트 평가 이후에 다시 개입하는 hook 패턴을 예제

- afterProject
- projectsEvaluated
- settingsEvaluated

init.gradle이 먼저 로드되더라도, 그 안에서 나중에 실행될 hook을 등록할 수 있다는 점입니다. 그러면 실제 값 변경은 settings.gradle이나 build.gradle이 평가된 뒤에 일어나므로, 사실상 뒤에서 덮어쓰는 효과를 낼 수 있습니다.

가장 이해하기 쉬운 예부터 보겠습니다.

프로젝트의 build.gradle이 이런 상태라고 가정하겠습니다.
```gradle
    repositories {
        mavenCentral()
    }

    configurations.all {
        resolutionStrategy {
            force "commons-io:commons-io:2.11.0"
        }
    }
```

그런데 init.gradle에서 나중에 강제로 바꾸고 싶다면 이렇게 할 수 있습니다.
```gradle
    gradle.afterProject { project ->
        project.configurations.all {
            resolutionStrategy {
                force "commons-io:commons-io:2.16.1"
            }
        }
    }
```

이 경우 흐름은 이렇습니다.

1. init.gradle이 먼저 로드된다
2. 하지만 바로 값을 바꾸는 대신 afterProject hook만 등록한다
3. 각 프로젝트의 build.gradle 평가가 끝난 뒤 afterProject가 실행된다
4. 그래서 build.gradle에서 넣은 2.11.0 위에 init.gradle의 2.16.1이 나중에 다시 적용된다

즉, 먼저 로드됐지만 나중에 실행되게 만들어서 최종값을 덮는 것입니다.

저장소 설정도 비슷하게 할 수 있습니다.

build.gradle이 이렇게 되어 있어도

```gradle
    repositories {
        mavenCentral()
    }
```

init.gradle에서 뒤에 사설 저장소를 추가할 수 있습니다.

```gradle
    gradle.afterProject { project ->
        project.repositories.clear()
        project.repositories.maven {
            url = uri("https://nexus.company.local/repository/maven-public")
        }
        project.repositories.mavenCentral()
    }
```

이건 꽤 강한 방식입니다. 기존 저장소를 지우고 다시 채우므로, 프로젝트 쪽 설정을 사실상 무시하게 됩니다. 실무에서는 강제력이 큰 대신 추적이 어려워질 수 있어서 조심해야 합니다.

전체 프로젝트 평가가 모두 끝난 뒤 한 번에 손대고 싶으면 projectsEvaluated를 씁니다.

```gradle
    gradle.projectsEvaluated {
        rootProject.allprojects { project ->
            project.tasks.withType(Test).configureEach {
                testLogging {
                    events "failed", "skipped", "passed"
                }
            }
        }
    }
```


이건 각 build.gradle이 다 읽힌 뒤 테스트 태스크 설정을 마지막에 통일하는 패턴입니다. 팀 공통 정책을 나중에 덮어쓸 때 자주 씁니다.

settings 단계에 개입하는 예도 있습니다.

```gradle
    gradle.settingsEvaluated { settings ->
        println("settings evaluated: " + settings.rootProject.name)
    }
```

이 hook은 settings.gradle 평가가 끝난 직후에 동작합니다. 예를 들어 pluginManagement나 dependencyResolutionManagement를 본 뒤 후속 처리를 붙이고 싶을 때 씁니다. 다만 settings 객체와 project 객체는 다르기 때문에, 저장소나 dependency 강제처럼 프로젝트 단위 설정을 바꾸려면 보통 afterProject나 projectsEvaluated가 더 직접적입니다.

언제 어떤 hook을 쓰면 되냐면 보통 이렇게 보면 됩니다.

1. 각 프로젝트가 평가될 때마다 바로 개입
   afterProject

2. 모든 프로젝트 평가가 끝난 뒤 한 번에 통일
   projectsEvaluated

3. settings.gradle이 끝난 직후 개입
   settingsEvaluated

실무적으로 가장 중요한 포인트는 이것입니다.

- init.gradle 안에서 그냥 즉시 설정하면 나중에 build.gradle이 다시 덮어쓸 수 있다
- init.gradle 안에서 hook을 등록하면, 더 늦은 시점에 실행되도록 만들어 오히려 최종값을 잡을 수 있다

짧게 비교하면:

즉시 설정 예:
이건 나중에 덮일 수 있음

```gradle
    allprojects {
        version = "1.0"
    }
```

나중 hook 설정 예:
이건 뒤에서 다시 덮을 수 있음

```gradle
    gradle.afterProject { project ->
        project.version = "2.0"
    }
```

원하면 다음으로
1. repository 설정 충돌 예제
2. dependency 버전 강제 예제
3. 현재 프로젝트 기준으로 init.gradle 샘플 하나

중 하나를 이어서 구체적으로 보여드릴 수 있습니다.