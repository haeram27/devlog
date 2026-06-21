# gradle commands

## dependencies

- [Viewing Dependencies](https://docs.gradle.org/current/userguide/viewing_debugging_dependencies.html)

```bash
# List project dependencies
./gradlew dependencies
./gradlew :<module>:dependencies

# Specifying a dependency configuration
./gradlew -q dependencies --configuration testRuntimeClasspath
./gradlew -q dependencies --configuration tRC
```

```text
> Task :app:dependencies

------------------------------------------------------------
Project ':app'
------------------------------------------------------------

annotationProcessor - Annotation processors and their dependencies for source set 'main'.
No dependencies

compileClasspath - Compile classpath for source set 'main'.
\--- com.fasterxml.jackson.core:jackson-databind:2.17.2
     +--- com.fasterxml.jackson.core:jackson-annotations:2.17.2
     |    \--- com.fasterxml.jackson:jackson-bom:2.17.2
     |         +--- com.fasterxml.jackson.core:jackson-annotations:2.17.2 (c)
     |         +--- com.fasterxml.jackson.core:jackson-core:2.17.2 (c)
     |         \--- com.fasterxml.jackson.core:jackson-databind:2.17.2 (c)
     +--- com.fasterxml.jackson.core:jackson-core:2.17.2
     |    \--- com.fasterxml.jackson:jackson-bom:2.17.2 (*)
     \--- com.fasterxml.jackson:jackson-bom:2.17.2 (*)

...
```

- '(*)': 전이 의존성(repeated occurrence) 하위 트리의 반복적인 발생을 나타냅니다. Gradle은 프로젝트당 한 번만 전이 의존성 하위 트리를 확장하고, 반복적인 발생은 하위 트리의 루트만 표시한 다음 주석을 추가합니다. `앞에서 이미 나온 의존성임을 표시`
- '(c)': 이 요소는 종속성이 아닌 종속성 제약 조건(dependency constraint)입니다. 트리의 다른 곳에서 일치하는 종속성을 찾습니다. 종속성 제약 조건(dependency constraint)이란  실제 종속성(라이브러리) artifact를 포함하고 있는 것이 아닌 필요하다고 표기해 놓은 것. 프로젝트의 어딘가에서 해당 dependency(라이브러리) artifact를 실제로 포함(include) 해야 정상 동작함
- '(n)': 해결할 수 없는 종속성 또는 종속성 구성