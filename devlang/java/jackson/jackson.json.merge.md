# Jackson으로 JSON Merge 하기 (Shallow vs Deep)

- [Jackson으로 JSON Merge 하기 (Shallow vs Deep)](#jackson으로-json-merge-하기-shallow-vs-deep)
  - [개요](#개요)
  - [핵심 개념](#핵심-개념)
    - [Shallow Merge](#shallow-merge)
    - [Deep Merge](#deep-merge)
    - [차이 요약](#차이-요약)
    - [전체 문서에 Deep Merge를 적용하려면?](#전체-문서에-deep-merge를-적용하려면)
  - [사용 방법](#사용-방법)
    - [실행 환경](#실행-환경)
    - [예제 코드](#예제-코드)
    - [실행 방법](#실행-방법)
    - [실행 결과](#실행-결과)
  - [주의사항 / 트러블슈팅](#주의사항--트러블슈팅)
  - [검증 방법](#검증-방법)
  - [참고 자료](#참고-자료)

## 개요

- 무엇인지: Jackson(`ObjectMapper`)으로 기존 JSON(또는 POJO)에 패치 JSON을 병합(merge)하는 방법이다.
- 언제 쓰는지: 부분 업데이트(PATCH), 기본 설정 + 사용자 설정 오버라이드, 다단계 설정 합성 시 사용한다.
- 한계/주의: 기본 동작은 대부분 **얕은 병합(Shallow)** 에 가깝고, 중첩 객체를 필드 단위로 보존하려면 **깊은 병합(Deep)** 설정이 필요하다.

## 핵심 개념

### Shallow Merge

- 최상위 필드 기준으로 값을 덮어쓴다.
- `@JsonMerge`를 사용하지 않으면, 중첩 객체 필드는 내부 필드 병합 없이 객체 단위로 교체된다.
- Jackson에서는 `readerForUpdating(...).readValue(...)`를 사용했을 때, 별도 설정이 없으면 이런 패턴이 자주 나온다.

### Deep Merge

- 중첩 객체 내부까지 재귀적으로 내려가서 필드 단위로 병합한다.
- 새 JSON에 없는 하위 필드는 기존 값을 유지한다.
- Jackson에서는 대상 필드에 `@JsonMerge`를 지정해 deep merge 의도를 명시할 수 있다.

### 차이 요약

| 구분 | Shallow Merge | Deep Merge |
| --- | --- | --- |
| 병합 기준 | 최상위 필드 중심 | 중첩 필드까지 재귀 병합 |
| 중첩 객체 처리 | 객체 단위 교체(기존 하위값 유실 가능) | 하위 필드 단위 반영(기존값 보존 가능) |
| Jackson 구현 포인트 | `readerForUpdating` 기본 사용 | `@JsonMerge`로 필드별 deep merge |

### 전체 문서에 Deep Merge를 적용하려면?

- 결론: `@JsonMerge`를 문서 전체에 한 번에 적용하는 위치는 없다.
- 이유: `@JsonMerge`는 클래스 전체가 아니라 **필드(프로퍼티) 단위**로 동작한다.
- 적용 방법:
  - 루트 DTO의 중첩 객체 필드마다 `@JsonMerge`를 명시
  - 하위 DTO에서도 중첩 객체 필드가 있으면 다시 `@JsonMerge`를 명시
- 참고: `ObjectMapper`의 `configOverride(...).setMergeable(true)`로 타입 단위 기본값을 줄 수 있지만, 실무에서는 의도를 명확히 하기 위해 필요한 필드에 `@JsonMerge`를 직접 명시하는 방식이 안전하다.

## 사용 방법

### 실행 환경

- Java 17+
- Jackson 3.x (예제는 3.0.1)
- 기준: Jackson 3에서는 `ObjectMapper` 패키지가 `tools.jackson.databind`으로 변경됨

`pom.xml` 예시:

```xml
<dependencies>
  <dependency>
    <groupId>tools.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>3.0.1</version>
  </dependency>
</dependencies>

<build>
  <plugins>
    <plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>exec-maven-plugin</artifactId>
      <version>3.3.0</version>
    </plugin>
  </plugins>
</build>
```

### 예제 코드

`src/main/java/example/JacksonMergeExample.java`

```java
package example;

import com.fasterxml.jackson.annotation.JsonMerge;
import tools.jackson.databind.ObjectMapper;

public class JacksonMergeExample {

    public static class Display {
        public Integer brightness;
        public Integer fontSize;
    }

    // 기본(Shallow) 병합 대상
    public static class SettingsShallow {
        public String theme;
        public Display display;
    }

    // Deep 병합 대상: 중첩 객체 필드에 @JsonMerge 지정
    public static class SettingsDeep {
        public String theme;

        @JsonMerge
        public Display display;
    }

    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();

        String baseJson = """
            {
              "theme": "Light",
              "display": {
                "brightness": 70,
                "fontSize": 12
              }
            }
            """;

        String patchJson = """
            {
              "theme": "Dark",
              "display": {
                "brightness": 90
              }
            }
            """;

        // 1) Shallow merge
        SettingsShallow shallow = mapper.readValue(baseJson, SettingsShallow.class);
        mapper.readerForUpdating(shallow).readValue(patchJson);
        System.out.println("[SHALLOW]");
        System.out.println(mapper.writerWithDefaultPrettyPrinter().writeValueAsString(shallow));

        // 2) Deep merge (@JsonMerge)
        SettingsDeep deep = mapper.readValue(baseJson, SettingsDeep.class);
        mapper.readerForUpdating(deep).readValue(patchJson);
        System.out.println("[DEEP]");
        System.out.println(mapper.writerWithDefaultPrettyPrinter().writeValueAsString(deep));
    }
}
```

### 실행 방법

```bash
mvn -q -DskipTests compile exec:java -Dexec.mainClass=example.JacksonMergeExample
```

### 실행 결과

```json
[SHALLOW]
{
  "theme" : "Dark",
  "display" : {
    "brightness" : 90,
    "fontSize" : null
  }
}
```

```json
[DEEP]
{
  "theme" : "Dark",
  "display" : {
    "brightness" : 90,
    "fontSize" : 12
  }
}
```

결론:

- Shallow: `display`가 교체되어 `fontSize`가 사라진다.
- Deep: `display.brightness`만 갱신되고 `fontSize`는 유지된다.

## 주의사항 / 트러블슈팅

- `@JsonMerge`는 병합 의도를 필드 단위로 제어하므로, 모든 필드에 자동으로 deep merge가 적용되는 것은 아니다.
- 배열/리스트 병합 정책은 객체 병합과 다르게 동작할 수 있으므로(교체 vs 결합), 요구사항에 맞는 정책을 별도로 검증해야 한다.
- `null`을 패치로 보낼 때는 “기존값 유지”인지 “명시적 삭제”인지 API 정책을 먼저 정하고 구현해야 한다.

## 검증 방법

- Shallow 결과에서 `display.fontSize == null`인지 확인
- Deep 결과에서 `display.fontSize == 12`가 유지되는지 확인
- 동일 입력으로 여러 번 실행해도 같은 결과가 나오는지 확인(멱등성 점검)

## 참고 자료

- Jackson 3 `ObjectMapper#readerForUpdating`: https://javadoc.io/doc/tools.jackson.core/jackson-databind/latest/tools/jackson/databind/ObjectMapper.html
- Jackson `@JsonMerge`: https://javadoc.io/doc/com.fasterxml.jackson.core/jackson-annotations/latest/com/fasterxml/jackson/annotation/JsonMerge.html
- Jackson 3 마이그레이션 가이드: https://github.com/FasterXML/jackson/blob/main/jackson3/MIGRATING_TO_JACKSON_3.md
