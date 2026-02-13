# Jackson 라이브러리

Java Object 형태의 데이터(VO, DTO)를 JSON 형식 데이터와 상호 변환을 지원하는 라이브러리

- 스트림 방식으로 속도가 빠르며 유연하고, annotation 방식으로 메타 데이터를 기술할 수 있음
- JsonMapper API를 사용하여 객체에 데이터 세팅을 함
- Json 이외 XML/YAML/CSV 등 다양한 형식 지원할 수 있지만 별도의 dataformat 모듈 의존성을 추가 해야함
  - [Group: FasterXML Jackson Dataformat](https://mvnrepository.com/artifact/com.fasterxml.jackson.dataformat)

- jackson은 Java Object(DTO)를 Json Document로 직렬화/역직렬화 하기 위한 도구 이다.
  - DTO는 Java Object(POJO)이며, json 문서의 data와 object의 각 fileld를 맵핑하기 위해서 사용된다.
  - RequestDTO는 Http request body에 담긴 json을 맵핑하기 위한 DTO이다.
  - RequestDTO는 Http response body에 담긴 json을 맵핑하기 위한 DTO이다.

- DTO에서 json data와 맵핑될 필드는 반드시 lowerCamelCase 로 작성되어야 한다.
  - DTO는 setter(private 멤버 접근) 또는 reflection(생성자 필요)하다.
  - @Data lombok annotation을 사용하면 getter/setter 메소드와 reflection에 필요한 non-param 생성자를 자동 생성해준다.

## Jackson 종속성 별도로 추가하여 사용하기

- spring-boot-starter-web (v3.0 이후) 종속성에 기본적으로 `jackson.core` 모듈을 포함되어 있습니다. 단, core가 아닌 `datatype`이나 `module` 등은 필요시 dependencies에 명시적으로 포함하여야 합니다.
- 일반 spring-boot-starter 나 java 어플리케이션에서 jackson을 사용하려면 별도로 build.gradle에 모든 종속성을 포함해야 합니다.

### build.gradle

```gradle
ext {
    jacksonVersion = '2.20.1'
}

dependencies {
    // ## jackson
    // jackson base library (m): support JsonMapper
    implementation "com.fasterxml.jackson.core:jackson-databind:${jacksonVersion}"
    // jackson core library (m)
    implementation "com.fasterxml.jackson.core:jackson-core:${jacksonVersion}"
    // jackson support annotation (m)
    implementation "com.fasterxml.jackson.core:jackson-annotations:${jacksonVersion}"
    // jackson suport Java 8 date/time Module (o)
    implementation "com.fasterxml.jackson.datatype:jackson-datatype-jsr310:${jacksonVersion}"
    implementation "com.fasterxml.jackson.module:jackson-module-parameter-names:${jacksonVersion}"
    // jackson support YAML Pretty Print (o)
    implementation "com.fasterxml.jackson.dataformat:jackson-dataformat-yaml:${jacksonVersion}"
    // jackson support kotlin (o)
    // implementation "com.fasterxml.jackson.module:jackson-module-kotlin:${jacksonVersion}"
}
```

- jackson-core
  - low-level 스트리밍 API 정의 및 JSON 별 구현
- jackson-annotations
  - 표준 Jackson annotation 포함
- jackson-databind
  - 스트리밍 패키지에 대한 데이터 바인딩 지원 구현. 스트리밍 및 annotation 패키지에 의존
- jackson-datatype-jsr310
  - Java 8 날짜 및 시간 API 지원
- jackson-module-parameter-names 
  - 파라미터 이름 매핑 지원, @RequestBody 마크된 파라미터에 대해 DTO내 변수 이름으로 json을 자동 매핑
- jackson-module-kotlin
  - Kotlin 지원