# proguard

## 기능

shrink, optimize, obfuscate 처리를 통해서 jar 파일의 경량화와 난독화를 제공

| 단계 | 기본 옵션 | 비활성화 옵션 (`-dont...`) | 핵심 목적 | 내부에서 수행되는 일 |
|:----:|-----------|----------------------------|-----------|-----------------------|
| **1. Shrink** | `-shrink` (기본 활성) | `-dontshrink` | **쓸모없는 코드 제거**<br>JAR·클래스·메서드·필드·리소스 중에서 실제로 한 번도 참조되지 않는 항목을 “트리 쉐이킹(tree‑shaking)” 방식으로 삭제하여 파일 크기를 최소화 | *사용하지 않는 클래스/멤버 삭제<br>* 상수 전파·사라진 코드 접합<br>*리소스(이미지, properties 등) trim<br>* **주의:** 리플렉션·JNI·직렬화처럼 “코드 밖”에서 호출되는 요소는 `-keep` 규칙으로 보호해야 함 |
| **2. Optimize** | `-optimizationpasses` 1 (암묵적으로 활성) | `-dontoptimize` | **바이트코드 최적화**<br>속도와 추가적인 크기 절감을 위해 바이트코드를 다시 작성 | *메서드 인라이닝·루프 전개·상수 접합<br>* 데드‑코드 제거(더 세밀하게)<br>*필드/메서드의 접근 제어자 조정<br>* 스택 깊이·로컬 변수 재사용으로 스택프레임 축소<br>* **주의:** 디버깅/프로파일링 툴, 일부 트레이스 코드와 호환성 이슈가 있을 수 있음 |
| **3. Obfuscate** | `-obfuscate` (기본 활성) | `-dontobfuscate` | **난독화 및 심볼 최소화**<br>코드를 이해하기 어렵게 만들고 식별자 길이를 줄여 추가적인 크기 절감 | *클래스·메서드·필드·패키지를 짧은 이름(예: `a`, `b`)으로 전면 변경<br>* Inner‑class, generic, line‑number, source‑file 등의 메타데이터 정리/삭제<br>*문자열·리소스 난독화는 별도 플러그인 필요<br>* **주의:** 리플렉션·직렬화·JMX 등 이름 기반 접근은 `-keep` 혹은 `-adaptresource...` 로 예외 처리 |

실행 순서는 Shrink → Optimize → Obfuscate → Preverify(선택) 이며, 앞 단계 결과가 뒷 단계의 입력이 됩니다.
따라서 -dontshrink를 넣어도 -dontobfuscate를 넣지 않는 한 난독화는 그대로 진행됩니다.

## proguard 설정 예

- jdk version = 17
- proguard-maven-plugin version = 2.7.0 (nested proguard version 7.4.1)

### maven command

#### proguard libaray download

```bash
mvn -U -X dependency:get -Dartifact=com.github.wvengen:proguard-maven-plugin:2.7.0
mvn -U -X dependency:get -Dartifact=com.guardsquare:proguard-base:7.7.0
mvn -U -X dependency:get -Dartifact=com.guardsquare:proguard-core:9.1.10
```

#### build package

```bash
mvn -Dcheckstyle.skip -Dpmd.skip=true -Dcpd.skip=true -Dmaven.test.skip=true clean package
```

#### 실제 빌드에 참조되는 pom.xml 완성본 확인

```bash
mvn help:effective-pom
```

### pom.xml

maven용 빌드 설정 파일
application source module의 root directory에 위치

- 일반 spring boot app용 pom.xml

```xml
<?xml version="1.0"?>

<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    ...

    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <!-- maven-compiler-plugin: java 컴파일 및 non-runnable thin jar 생성--> 
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${version.jdk}</source>
                    <target>${version.jdk}</target>
                </configuration>
            </plugin>

            <!-- Spring Boot runnable fat-jar 생성 (난독화된 classes + dependency libs) -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <id>repackage-obfuscated</id>
                        <phase>package</phase>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

- proguard 적용된 pom.xml ***(중요!!!)***

```xml
<?xml version="1.0"?>

<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd"
    xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <properties>
        <!-- proguard -->
        <version.plugin.proguard>2.7.0</version.plugin.proguard>
    </properties>

    ...

    <build>
        <finalName>${project.artifactId}</finalName>
        <plugins>
            <!-- maven-compiler-plugin: java 컴파일 및 non-runnable thin jar 생성--> 
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${version.jdk}</source>
                    <target>${version.jdk}</target>

                    <!-- Java 코드 컴파일 시 소스 코드의 매개변수(파라미터) 이름을 유지:
                         리플렉션을 통해 매개변수 정보 획득 가능 -->
                    <parameters>true</parameters>
                </configuration>
            </plugin>

            <!-- proguard-maven-plugin: proguard 난독화 실행 --> 
            <plugin>
                <groupId>com.github.wvengen</groupId>
                <artifactId>proguard-maven-plugin</artifactId>
                <version>${version.plugin.proguard}</version>

                <!-- ProGuard 엔진 버전 명시가 필요한 경우만 활성,
                     명시하지 않아도 proguard-maven-plugin이 자동으로 엔진 의존성 다운로드함 -->
                <!--
                <dependencies>
                    <dependency>
                        <groupId>com.guardsquare</groupId>
                        <artifactId>proguard-base</artifactId>
                        <version>7.7.0</version>
                    </dependency>
                    
                    <dependency>
                        <groupId>com.guardsquare</groupId>
                        <artifactId>proguard-core</artifactId>
                        <version>9.1.10</version>
                    </dependency>
                </dependencies>
                -->
                <executions>
                    <execution>
                        <id>obfuscate-classes</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>proguard</goal>
                        </goals>
                        <configuration>
                            <skip>false</skip>

                            <!-- JDK 모듈: 참조하는 jdk base libries를 '라이브러리로만 취급하고 난독화에 포함 X' -->
                            <libs>
                                <lib>${java.home}/jmods</lib>
                            </libs>

                            <!-- dependency(3rd party = spring 등)은 '라이브러리'로만 취급하고,
                                 난독화 입력(InJar)에 포함하지 않음 -->
                            <includeDependency>true</includeDependency>
                            <includeDependencyInjar>false</includeDependencyInjar>

                            <!-- 입력/출력: target/ 디렉토리 기준 '상대경로'를 명시 -->
                            <injar>classes</injar>         <!-- = target/classes (컴파일된 class 디렉토리) -->
                            <outjar>obf-classes</outjar>   <!-- = target/obf-classes (난독화 결과 디렉토리) -->

                            <!-- proguard 옵션 파일 지정 -->
                            <options>
                                <option>@${project.basedir}/proguard.pro</option>
                            </options>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <!-- obf-classes → target/classes 로 교체 -->
            <plugin>
                <artifactId>maven-antrun-plugin</artifactId>
                <version>3.1.0</version>
                <executions>
                    <execution>
                        <id>swap-classes</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>run</goal>
                        </goals>
                        <configuration>
                            <target>
                                <delete dir="${project.build.outputDirectory}"/>
                                <copy todir="${project.build.outputDirectory}">
                                    <fileset dir="${project.build.directory}/obf-classes"/>
                                </copy>
                            </target>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <!-- Spring Boot runnable fat-jar 생성 (난독화된 classes + dependency libs) -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <id>repackage-obfuscated</id>
                        <phase>package</phase>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

일반 spring boot app의 pom.xml 대비하여, `prepare-package`에 `proguard-maven-plugin`와 `maven-antrun-plugin` 단계가 추가 되었다.

난독화 빌드 단계는 다음과 같다

1. maven-compiler-plugin
   - phase: compile ~
   - 소스내 .java 파일을 컴파일하여 `target/classes`에 결과 .class를 생성하고 thin jar 파일을 생성한다.

1. proguard-maven-plugin
   - phase: prepare-package
   - target/classes 디렉토리 안의 class 파일에 난독화(obfuscation)를 적용하여 obf-classes 디렉토리에 결과물 생성한다.

1. maven-antrun-plugin
   - phase: prepare-package
   - `target/classes`를 지우고 `target/obf-classes`를 `target/classes` 로 만든다.

1. spring-boot-maven-plugin
   - phase: package
   - `target/classes`를 기반으로 dependency 라이브러리들을 포함하여 runnable fat-jar를 생성한다.

### proguard.pro ***(중요!!!)***

pom.xml과 같이 application source module의 root directory에 위치

proguard에 지정할 option을 명시한다.
주로 keep 구문을 통해서 난독화 예외처리 대상을 지정하는데 사용된다.

(!) spring framework에서 예외 처리대상

- enum (?)
- dao (?)

```text
# ---- 동작 모드: 난독화만, 최적화/제거 비활성화 ----
-dontoptimize
-dontshrink
-dontpreverify

# 스택트레이스 복원을 위한 매핑 출력
-printmapping target/mapping.txt

# 유용한 메타데이터/애너테이션 유지
-keepattributes SourceFile,LineNumberTable,Exceptions,Signature,EnclosingMethod,InnerClasses,*Annotation*,Record,MethodParameters

# ---- 엔트리포인트 (메인) ----
-keep public class com.myapp.MyApplication { public static void main(java.lang.String[]); }

# ---- Spring 컴포넌트/설정/부트스트랩 ----
-keep @org.springframework.stereotype.Component class * { *; }
-keep @org.springframework.stereotype.Service class * { *; }
-keep @org.springframework.stereotype.Repository class * { *; }
-keep @org.springframework.web.bind.annotation.RestController class * { *; }
-keep @org.springframework.context.annotation.Configuration class * { *; }
-keep @org.springframework.boot.autoconfigure.SpringBootApplication class * { *; }
-keepclassmembers class * { @org.springframework.context.annotation.Bean <methods>; }  # @Bean 이름 보존
-keep @org.springframework.web.bind.annotation.ControllerAdvice class * { *; }
-keep @org.springframework.web.bind.annotation.RestControllerAdvice class * { *; }


# ---- ConfigurationProperties 바인딩 보존 ----
-keep @org.springframework.boot.context.properties.ConfigurationProperties class * { *; }

# ---- Jackson 직렬화/역직렬화 ----
-dontwarn com.fasterxml.jackson.**
-keepclassmembers class * {
  @com.fasterxml.jackson.annotation.JsonCreator <init>(...);
  @com.fasterxml.jackson.annotation.JsonProperty *;
  public <init>(...);
  public *** get*();
  public void set*(***);
}

# ---- MyBatis (매퍼 인터페이스/애너테이션 스캔) ----
-keep @org.apache.ibatis.annotations.Mapper interface * { *; }
# XML 매퍼에서 FQCN을 참조한다면 해당 패키지의 타입 이름 보존 권장(예: com.myapp.**.mapper)
#-keep class com.myapp.**.mapper.** { *; }

# ---- MongoDB(스프링 데이터) 도메인 모델 보존(필요 시) ----
# @Document로 마킹된 엔티티 클래스가 있으면 이름/필드가 바뀌면 곤란할 수 있음
-keep @org.springframework.data.mongodb.core.mapping.Document class * { *; }

# ---- 기타 유틸 ----
# ignore logger/library warnings (narrow down the scope, if necessary)
-dontwarn org.slf4j.**
-dontwarn ch.qos.logback.**
-dontwarn org.springframework.**

# enum
# --- 예외 패키지 하위의 '모든' 내부 enum 을 보존 ---
-keep enum com.myapp.enum.exception.**$* { *; }
```

참고: proguard configuaration manual - keep 설정 관련 option

- [Keep Options](https://www.guardsquare.com/manual/configuration/usage#keepoptions)
- [Class Specification](https://www.guardsquare.com/manual/configuration/usage#classspecification)

## 난독화 결과 확인

### target/mapping.txt

다음과 같이 mapping.txt에 난독화 된 class와 그 정보가 정상 포함되어 있는지 확인

```text
com.myapp.MyDataBuilder -> com.myapp.b.b$a:   # {"fileName":"MybuilderDataBuilder.java","id":"sourceFile"}
    java.lang.Integer scanId -> a
    java.lang.Integer statusId -> b
    java.lang.Integer nodeId -> d
    java.lang.String layerName -> e
    java.lang.String layerFile -> f
    java.lang.Boolean isHost -> g
    java.lang.String createTime -> i
    java.lang.String moddifyTime -> j
```

### target/obf-classes와 classes 디렉토리

- 난독화 class 생성 여부를 확인한다.

```bash
tree -d classes
tree -d obf-classes
```

### target/${project.artifactId}.jar

- fat-jar 내부 난독화 결과 확인

```bash
jar tf target/${project.artifactId}.jar | grep 'BOOT-INF/classes/.*\.class'
```

실행 결과로 다음과 같이 package 이름에 난독화가 되어 있으면

```text
BOOT-INF/classes/com/myapp/test/a/b/a$1.class
```

## 참고

### proguard-maven-plugin의 pom.xml 설정

- <https://wvengen.github.io/proguard-maven-plugin/>
- <https://wvengen.github.io/proguard-maven-plugin/proguard-mojo.html>

### proguard 옵션 설정

- <https://www.guardsquare.com/manual/configuration/usage>

### jar 파일 디컴파일러

- <https://java-decompiler.github.io/>
