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



java로 jar 실행 시 main class 지정 방법

java -jar -Xms4096m -Xmx4096m -Dmax.threads=512 -Dapp.processor.id=myapp /path/to/application.jar &


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


- 매니페스트에 mainClass 지정 제약사항



- build.gralde에서 main class 지정
```gradle
tasks.jar {
    manifest {
        attributes["Main-Class"] = "com.example.MainA"
    }
}
```

- pom.xml에서 main class 지정
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
- mainClass : public static void main(String[] args) 를 가진 FQCN (Fully Qualified Class Name)
- args... : String[] args 로 전달될 값들

```
java -cp "myapp.jar:lib/*" com.example.MainApp arg1 arg2 "arg 3"
```


java -cp "myapp.jar:lib/*" com.example.MainB   # Unix ':' 구분자 사용
java -cp "myapp.jar;lib/*" com.example.MainB   # Windows ';' 구분자 사사용

```sh
#!/usr/bin/env bash
java -cp "$(dirname $0)"/myapp.jar" com.example.MainB
```
