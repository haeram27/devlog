# How to run jar

실행 가능한 jar는 main() 메소드를 포함하는

$ mvn install -Dcheckstyle.skip -Dpmd.skip=true -Dcpd.skip=true -Dmaven.test.skip=true

## 한 개의 JAR 안에 main 메서드가 여러 개 있을 때 실행 방법

| 실행 방식 | 명령 예시 | 특징 |
|-----------|-----------|-------|
| **`java -jar` (매니페스트 사용)** | `java -jar myapp.jar` | `META-INF/MANIFEST.MF` 안의 **`Main-Class`** 항목에 적힌 클래스만 실행 가능. 한 JAR 당 1 개가 고정. |
| **`java -cp` (클래스패스 직접 지정)** | `java -cp myapp.jar com.example.OtherMain` | 매니페스트를 무시하고 원하는 클래스 실행. JAR 내부에 몇 개든 `main`이 존재해도 선택 가능. |
| **스크립트/배치 파일 래퍼** | `runA.sh`, `runB.sh` 등 | 여러 실행 지점을 자주 쓰면 OS 스크립트로 래핑해 편의성 확보. |
| **별도 JAR 생성** | `maven-shade`, `gradle shadowJar` | 두 실행 파일을 각각의 fat-jar로 만들어 배포. |
| **모듈 방식(9+)** | `java -p . -m my.mod/foo.Bar` | 모듈화 프로젝트라면 `module-info.java`에 `Main-Class`를 쓰지 않고 `--module` 옵션으로 구동. |

---

## java -cp 옵션 사용법법

`-cp`(또는 `-classpath`) 옵션에는 “Java가 클래스(.class)나 리소스를 찾을 때 뒤져야 할 경로”를 지정합니다.
`-cp` 옵션 값에는 **“클래스를 포함한 폴더”**, **“JAR/ZIP 파일”**, **“와일드카드 확장”** 을 조합하여 지정할 수 있으며, OS의 경로 구분자를 지켜 나열하면 됩니다.

지정할 수 있는 값의 종류는 크게 다음과 같습니다.

1. **디렉터리**

   * 컴파일된 `.class` 파일이 들어 있는 폴더를 지정
   * 예: `-cp /home/user/myapp/bin`

2. **JAR 파일(또는 ZIP 파일)**

   * 내부에 `.class`와 리소스가 들어 있는 압축 파일
   * 예: `-cp lib/mylib.jar`

3. **여러 경로의 조합**

   * OS별 구분자(`:` on UNIX/Linux/macOS, `;` on Windows)로 나열
   * 예 (Linux/macOS):

     ```bash
     java -cp bin:lib/a.jar:lib/b.jar com.example.Main
     ```

   * 예 (Windows):

     ```cmd
     java -cp bin;lib\a.jar;lib\b.jar com.example.Main
     ```

4. **와일드카드 패턴** (Java 6 이후)

   * 디렉터리 내 모든 JAR/ZIP 파일을 한꺼번에 포함
   * 문법: `<디렉터리>/*`
   * 예:

     ```bash
     java -cp "bin:lib/*" com.example.Main
     ```

   * 주의: 하위 디렉터리까지 재귀적으로 검색하지는 않으며, 오직 지정한 디렉터리의 최상위에 있는 `*.jar`·`*.zip` 만 포함됩니다.

5. **상대 경로·절대 경로 혼용**

   * `.`(현재 디렉터리), `../` 등 상대경로도 가능
   * 예: `-cp .:../common/lib/*.jar`

6. **매니페스트(Class-Path) 무시**

   * JAR 내부 `META-INF/MANIFEST.MF` 의 `Class-Path` 선언 대신, `-cp` 에 지정된 값이 우선 적용됩니다.

### 예시

```bash
# 1) 단일 디렉터리
java -cp build/classes com.myapp.App

# 2) 디렉터리 + 개별 JAR
java -cp build/classes:lib/foo.jar:lib/bar.jar com.myapp.App

# 3) 디렉터리 + lib 디렉터리의 모든 JAR
java -cp build/classes:lib/* com.myapp.App

# 4) Windows 환경
java -cp build\classes;lib\* com.myapp.App
```

---

## java로 jar 실행 시 main class 지정 방법

```bash
java -jar -Xms4096m -Xmx4096m -Dmax.threads=512 -Dapp.processor.id=myapp /path/to/application.jar &
```

## "java -jar"로 실행 시 manifest 생성

"java -jar"를 사용하면 매니페스트 파일(META-INF/MANIFEST.MF)에 지정된 mainClass를 실행한다.
매니페이스트에 지정된 mainClass가 없다면 java는 다음과 같은 에러를 발생시키고 종료된다.
```no main manifest attribute, in myapp.jar```

예외적으로 spring-boot-maven-plugin을 쓰는 프로젝트라면 <spring-boot:repackage> 단계에서 @SpringBootApplication이 붙은 클래스를 찾아 자동으로 Start-Class(Spring Boot용 Main-Class)를 기록해 주지만, 일반 maven-jar-plugin만 쓰는 경우에는 자동 지정이 일어나지 않는다.

## 예외 처럼 보이는 경우 들

| 상황 | 왜 실행이 되나? | 실제로는… |
|------|----------------|-----------|
| **Spring Boot 프로젝트** | `spring-boot-maven-plugin`이 빌드 마지막 단계에서 JAR를 “repackage” 하면서 <br>`Start-Class`(사용자 앱의 @SpringBootApplication)와 <br>`Main-Class`(org.springframework.boot.loader.JarLauncher) 를 자동 기록 | Maven 기본 JAR이 아니라 **별도로 재가 ([java - Can't execute jar- file: "no main manifest attribute" - Stack Overflow](https://stackoverflow.com/questions/9689793/cant-execute-jar-file-no-main-manifest-attribute))
| **Shadow/Shade Plug-in 사용** | `maven-shade-plugin`의 `ManifestResourceTransformer` 등으로 `Main-Class`를 주입 | 사용자 또는 플러그인 설정에서 명시적으로 넣어 준 것 |
| **CLI로 직접 `jar -cmvf`** | 매니페스트 파일을 수동으로 포함 | 역시 사람이 지정 |

* 매니페스트에 mainClass 지정 제약사항

* build.gralde에서 main class 지정

```gradle
tasks.jar {
    manifest {
        attributes["Main-Class"] = "com.example.MainA"
    }
}
```

* pom.xml에서 main class 지정

```xml
<build>
  <plugins>
    <plugin>
    <!-- Build an executable JAR -->
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jar-plugin</artifactId>
      <version>3.4.2</version>
      <configuration>
        <archive>
          <manifest>
            <mainClass>com.example.MainApp</mainClass>
          </manifest>
        </archive>
      </configuration>
    </plugin>
  </plugins>
</build>
```

```
java -cp <class-path>[:<class-path>:...] <package.path.to.mainClass> [args...]
```

- class-path : 실행할 .jar 파일이나 클래스들이 들어있는 디렉토리 경로
* mainClass : public static void main(String[] args) 를 가진 FQCN (Fully Qualified Class Name)
* args... : String[] args 로 전달될 값들

```
java -cp "myapp.jar:lib/*" com.example.MainApp arg1 arg2 "arg 3"
```

java -cp "myapp.jar:lib/*" com.example.MainB   # Unix ':' 구분자 사용
java -cp "myapp.jar;lib/*" com.example.MainB   # Windows ';' 구분자 사사용

```sh
#!/usr/bin/env bash
java -cp "$(dirname $0)"/myapp.jar" com.example.MainB
```
