# gradle outline

## references

- [gradle official documents](https://docs.gradle.org/current/userguide/userguide.html)
- [blog](https://blog.naver.com/hj_kim97/222936667366)


## gradle.properties

**gradle.properties**는 Gradle 빌드의 **실행 환경 변수**를 선언하는 파일
- `.properties` 형식의 key=value 쌍으로 변수를 정의
- Gradle 실행 시 JVM과 빌드 스크립트(settings.gradle, build.gradle)에서 참조 가능

### 파일 위치 (우선순위 순)

1. **명령줄** (`-D` 옵션) - 가장 높은 우선순위
2. **${workspaceFolder}/gradle.properties** - 프로젝트 루트
3. **${GRADLE_USER_HOME}/gradle.properties** - `~/.gradle/` 디렉토리
4. **${GRADLE_HOME}/gradle.properties** - Gradle 설치 경로

> 💡 좁은 범위의 설정이 넓은 범위의 설정을 **override**합니다.

### 변수를 settings/build.gradle에 전달하는 방법

gradle.properties에 정의된 변수 prefix에 따라 다른 위치에서 참조 될 수 있음

| 형식 | 등록 대상 | 접근 방법 |
|------|---------|---------|
| `systemProp.*` | JVM 시스템 프로퍼티 | `System.getProperty("http.proxyHost")` |
| `org.gradle.*` | Gradle 자체 설정 | Gradle 내부에서 사용 |
| 기타 변수 | 프로젝트 프로퍼티 | `project.customVersion` |

```properties
# gradle.properties

# systemProp prefix: system property, graldle을 실행하는 JVM 시스템 프로퍼티로 등록
systemProp.http.proxyHost=127.0.0.1
systemProp.http.proxyPort=9913
systemProp.https.proxyHost=127.0.0.1
systemProp.org.gradle.internal.http.connectionTimeout=300000

# org.gradle prefix: gradle property, gradle 자체 설정
org.gradle.jvmargs=-Xmx2048M
org.gradle.parallel=true

# no prefix: project property, settings/build.gradle에서 직접 참조 가능
customVersion=1.0.0
privateMavenUrl=https://private.repository.com:443/repo
```



따라서 **`systemProp` 접두사가 있어야만** 실행 중인 JVM과 Java 애플리케이션에서 `System.getProperty()`로 접근 가능합니다.

### 주요 용도

- **JVM 옵션**: `org.gradle.jvmargs` (힙 메모리 설정)
- **빌드 성능**: `org.gradle.parallel=true` (병렬 빌드)
- **프록시 설정**: `systemProp.http.proxyHost` 등
- **커스텀 변수**: 버전, URL, 경로 등 프로젝트 특정 값

gradle.properties는 민감한 정보(API 키, 비밀번호)를 관리하는 데도 자주 사용되며, .gitignore에 추가하여 버전 관리에서 제외합니다.

## setting.gradle과 build.gradle

- settings.gradle: 빌드 전체의 구조와 전역 규칙 정의
- build.gradle: 각 프로젝트(모듈)의 실제 빌드 방법 정의

### gradle의 각 파일 평가 순서
먼저 settings.gradle.kts가 읽혀서 빌드의 틀(프로젝트 구조, 플러그인/저장소 규칙)을 만듭니다.
그다음 각 build.gradle.kts가 읽혀서 실제 빌드 동작을 만듭니다.

### settings.gradle
- 멀티모듈 구성(어떤 프로젝트를 포함할지 - include로 멀티모듈 구성)
- pluginManagement로 플러그인 저장소/버전 기준 설정(프로젝트 전역 plugin 버전 관리)
- dependencyResolutionManagement로 의존성 저장소 전역 정책 정의
- rootProject.name 같은 루트 메타 정보 설정

### build.gradle
- plugins로 해당 모듈에 실제 적용할 플러그인 지정
- 의존성(dependencies), 태스크(task), 컴파일 옵션, 테스트 설정(test), packaging(jar) 등 실질 빌드 로직 작성



