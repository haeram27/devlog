# pom에서 변수 참조

Maven [POM](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)(Project Object Model)은 java 프로젝트를 빌드하기 위한 maven의 설정 내용을 정형화한한 형식의 xml 문서로 정의한 것을 말한다.

참고:
- [introduction-to-the-pom](https://maven.apache.org/guides/introduction/introduction-to-the-pom.html)
- [pom-introduction](https://maven.apache.org/pom.html#Introduction)

Maven POM 안에서는 크게 다음 출처의 상수 값을 `${…}` 표현식으로 참조할 수 있습니다. (변수와 같이이 참조)

## **[POM 내 정의한 프로퍼티](https://maven.apache.org/pom.html#Properties)**

    ```xml
    <properties>
      <my.prop>hello</my.prop>
    </properties>
    …
    ${my.prop}  <!-- → "hello" -->
    ```

## **커맨드라인 `-D` 로 넘긴 시스템 프로퍼티**

    ```
    mvn clean install -Denv=prod -Dtimeout=30
    ```

    POM에서는

    ```xml
    ${env}      <!-- → "prod" -->
    ${timeout}  <!-- → "30"  -->
    ```

## **환경 변수**

    ```xml
    ${env.HOME}       <!-- 사용자 홈 디렉터리 -->
    ${env.PATH}       <!-- 시스템 PATH -->
    ${env.MY_SETTING} <!-- OS 환경 변수 MY_SETTING -->
    ```

## **JVM·OS 기본 시스템 프로퍼티**

    ```xml
    ${java.home}      <!-- JDK 홈 -->
    ${java.version}   <!-- JDK 버전 -->
    ${file.encoding}  <!-- 기본 문자 인코딩 -->
    ${user.home}      <!-- 사용자 홈 디렉터리 -->
    ${user.name}      <!-- 로그인 사용자 이름 -->
    ${os.name}        <!-- 운영체제 이름 -->
    ${os.arch}        <!-- 운영체제 아키텍처 -->
    ${os.version}     <!-- 운영체제 버전 -->
    ```

## **프로젝트 메타데이터 (Maven Project Object Model)**

    ```xml
    ${project.groupId}
    ${project.artifactId}
    ${project.version}
    ${project.build.directory}
    ${project.build.finalName}
    ${basedir}         <!-- POM 파일이 있는 디렉터리 -->
    ```

## **[Maven Settings / Session 정보](https://maven.apache.org/settings.html#Properties)**

    - `settings.xml` 에 `<profiles><profile><properties>` 으로 정의한 값
    - `${settings.localRepository}`
    - `${settings.activeProfiles}`
    - `${session.executionRootDirectory}` 등

## **플러그인·목표(Goal) 별 제공 프로퍼티**

    각 Maven 플러그인이 자체적으로 추가하는 프로퍼티(ex. `maven.compiler.source`, `maven.compiler.target` 등)도 `${…}`로 참조 가능합니다.

---

### 요약

|출처|POM 표현식 예시|
|---|---|
|POM `<properties>`|`${my.prop}`|
|커맨드라인 `-D`|`${env}`|
|환경 변수|`${env.MY_ENV}`|
|JVM/OS 시스템 프로퍼티|`${java.version}`, `${os.name}`|
|프로젝트 메타데이터|`${project.version}`, `${basedir}`|
|settings.xml 프로퍼티|`${settings.localRepository}`|
|세션 정보|`${session.executionRootDirectory}`|

> **팁**: POM에서 환경 변수나 시스템 프로퍼티를 직접 쓰기 어려운 경우, `<profiles>` 에서 `<activation><property>` 를 이용해 프로필을 자동 활성화하거나, `properties-maven-plugin` 같은 플러그인으로 더 복잡한 프로퍼티 조작도 가능합니다.
