
# ClassFinal

## 소개

ClassFinal은 Java 클래스 파일을 안전하게 암호화하는 도구입니다.  
jar 또는 war 패키지를 프로젝트 코드 수정 없이 바로 암호화할 수 있으며, Spring Framework와 호환됩니다.  
이를 통해 소스 코드 유출이나 바이트코드 디컴파일을 방지할 수 있습니다.

##### Gitee: https://gitee.com/roseboy/classfinal

## 프로젝트 모듈 설명

- **classfinal-core**: ClassFinal의 핵심 모듈로, 대부분의 암호화 로직을 포함  
- **classfinal-fatjar**: 독립 실행 가능한 fat-jar 형태로 패키징된 실행 파일  
- **classfinal-maven-plugin**: Maven 플러그인 형태로 제공되는 암호화 도구  

## 기능 특징

- 원본 프로젝트 코드를 수정할 필요 없이, 컴파일된 jar/war만 이 도구로 암호화  
- 암호화된 프로젝트 실행 시 Tomcat, Spring 등 소스 코드 수정 불필요  
- 일반 jar, Spring Boot fat-jar, 일반 Java 웹 프로젝트의 war 패키지 지원  
- Spring Framework, Swagger 등 시작 시 어노테이션 스캔이나 바이트코드 생성이 필요한 프레임워크 호환  
- Maven 플러그인 지원으로 `mvn package` 과정에서 자동 암호화  
- `WEB-INF/lib` 또는 `BOOT-INF/lib` 하위 의존 JAR 암호화 가능  
- 머신 바인딩 기능: 암호화된 프로젝트는 특정 머신에서만 실행  
- Spring Boot 설정 파일도 암호화  

## 환경 요구사항

- JDK 1.8 이상

## 사용 방법

### 다운로드

[여기를 클릭하여 암호화 도구 다운로드](https://repo1.maven.org/maven2/net/roseboy/classfinal-fatjar/1.2.1/classfinal-fatjar-1.2.1.jar)

### 암호화

다음 명령어를 실행합니다:

```bash
java -jar classfinal-fatjar.jar \
  -file yourproject.jar \
  -libjars a.jar,b.jar \
  -packages com.yourpackage,com.yourpackage2 \
  -exclude com.yourpackage.Main \
  -pwd 123456 \
  -Y
```

#### 매개변수 설명

- `-file`  
    암호화할 jar/war 파일의 전체 경로
    
- `-packages`  
    암호화 대상 패키지(여러 개일 경우 쉼표로 구분, 비워둘 수도 있음)
    
- `-libjars`  
    `lib` 디렉터리 아래 암호화할 JAR 파일명(여러 개일 경우 쉼표로 구분)
    
- `-cfgfiles`  
    암호화할 설정 파일(주로 `classes` 디렉터리의 `.yml` 또는 `.properties`), 쉼표로 구분
    
- `-exclude`  
    암호화에서 제외할 클래스명(쉼표로 구분)
    
- `-classpath`  
    외부 의존 JAR 디렉터리(예: `/tomcat/lib`), 쉼표로 구분
    
- `-pwd`  
    암호화 비밀번호. `#`을 지정하면 무비밀번호 모드
    
- `-code`  
    머신 코드(머신 바인딩 시 생성), 암호화 후 해당 머신에서만 실행 가능
    
- `-Y`  
    확인 프롬프트 생략. 지정하지 않으면 암호화 정보 확인을 요청
    

> 위 예시는 커맨드라인 인자 방식입니다. 생략 시 `java -jar classfinal-fatjar.jar` 만 실행하면 단계별 프롬프트로도 입력할 수 있습니다.

### Maven 플러그인 방식

`pom.xml`에 다음 플러그인 설정을 추가하세요 (최신 버전: `1.2.1`).

```xml
<plugin>
  <groupId>net.roseboy</groupId>
  <artifactId>classfinal-maven-plugin</artifactId>
  <version>1.2.1</version>
  <configuration>
    <password>000000</password>       <!-- 암호화된 JAR 안에서 pom.xml은 제거되므로, 암호 노출 걱정 없음 -->
    <packages>com.yourpackage,com.yourpackage2</packages>
    <cfgfiles>application.yml</cfgfiles>
    <excludes>org.spring</excludes>
    <libjars>a.jar,b.jar</libjars>
  </configuration>
  <executions>
    <execution>
      <phase>package</phase>
      <goals>
        <goal>classFinal</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

이후 `mvn package` 실행 시 `target/yourproject-encrypted.jar`이 생성됩니다.

### 무비밀번호 모드

- 암호화 시 `-pwd #` 로 설정하면, 실행 시 비밀번호 입력 없이 자동 진행
    
- war 패키지 실행 시 `-nopwd` 옵션을 추가하면 비밀번호 입력 없이 건너뜀
    

### 머신 바인딩

암호화된 패키지는 특정 머신에서만 실행됩니다.

1. 바인딩할 머신에서 머신 코드를 생성:
    
    ```bash
    java -jar classfinal-fatjar.jar -C
    ```
    
2. 암호화 시 `-code` 옵션으로 위에서 생성된 머신 코드를 지정하면, 해당 머신에서만 실행됩니다.  
    (머신 코드 + 비밀번호 방식도 지원)
    

### 암호화된 JAR 실행하기

암호화된 JAR은 `javaagent` 옵션으로 실행해야 합니다.  
실행 중 메모리 내에서만 복호화되며, 디스크에는 복호화된 클래스가 남지 않습니다.

```bash
java -javaagent:yourproject-encrypted.jar='-pwd 0000000' -jar yourproject-encrypted.jar
```

- `-pwd` 암호화 비밀번호
    
- `-pwdname` 환경 변수에 저장된 비밀번호 키
    

비밀번호를 커맨드라인에 포함하지 않으려면:

```bash
java -javaagent:yourproject-encrypted.jar -jar yourproject-encrypted.jar
```

실행 시 콘솔 입력 또는 GUI 입력이 가능합니다.  
~~GUI가 지원되지 않는 환경에서는 프로젝트 루트의 `classfinal.txt` 또는 `yourproject-encrypted.classfinal.txt` 파일에 비밀번호를 작성하면 자동 읽고 파일을 지워 줍니다.~~

비밀번호 읽기 순서:

1. 커맨드라인 인자
    
2. 환경 변수
    
3. 비밀번호 파일
    
4. 콘솔 입력
    
5. GUI 입력
    
6. 종료
    

### Tomcat에서 암호화된 WAR 실행하기

암호화된 WAR 파일을 `tomcat/webapps`에 배치하고, `catalina` 스크립트에 `javaagent` 옵션을 추가하세요.

- **Linux (`catalina.sh`)**
    
    ```bash
    CATALINA_OPTS="$CATALINA_OPTS -javaagent:classfinal-fatjar.jar='-pwd 0000000'"
    export CATALINA_OPTS
    ```
    
- **Windows (`catalina.bat`)**
    
    ```bat
    set JAVA_OPTS="-javaagent:classfinal-fatjar.jar='-pwd 000000'"
    ```
    

옵션 설명:

- `-pwd` 암호화 비밀번호
    
- `-nopwd` 무비밀번호 모드
    
- `-pwdname` 환경 변수 비밀번호 키
    

---

> 이 도구는 AES 알고리즘으로 클래스 파일을 암호화합니다.  
> 비밀번호는 암호 해독 방지의 핵심이므로 안전하게 보관하시고, 절대 유출되지 않도록 주의하세요.

> 비밀번호를 잊으면 복구할 수 없으므로 반드시 기억하세요.

> 암호화 후 원본 클래스 파일은 메서드 본문이 빈 상태로 남아 있습니다.  
> 메서드 시그니처와 어노테이션 정보만 유지되어, Spring·Swagger 등 어노테이션 스캔 프레임워크와 호환됩니다.  
> 실제 메서드 본문은 JVM이 로드 시점에 메모리 내에서 복호화하여 삽입하므로, 디스크에는 남지 않습니다.

> 런타임 보안을 위해 JVM 시작 시 `-XX:+DisableAttachMechanism` 옵션을 사용하세요.

## 버전 설명

- **v1.2.1** 버그 수정
    
- **v1.2.0** `packages`, `libjars`, `cfgfiles`, `exclude` 매개변수의 와일드카드 지원
    
- **v1.1.7** Spring Boot 설정 파일 암호화 지원 및 환경 변수 비밀번호 읽기 기능 추가
    
- **v1.1.6** 머신 바인딩 기능 추가
    
- **v1.1.5** 무비밀번호 모드 추가 (보안상 권장하지 않음)
    
- **v1.1.4** 순수 커맨드라인 실행 시 설정 파일에서 비밀번호 읽고 파일 삭제
    
- **v1.1.3** 비밀번호 입력용 GUI 팝업 지원
    
- **v1.1.2** Windows 암호화 후 실행 문제 수정
    
- **v1.1.1** JAR 실행 시 콘솔에서 비밀번호 입력
    
- **v1.1.0** 암호화된 JAR에 복호화 코드를 내장, 별도 JAR 불필요
    
- **v1.0.0** 첫 정식 버전 릴리즈
    

## 라이선스

[Apache-2.0](http://www.apache.org/licenses/LICENSE-2.0)