# springdoc-openapi 사용 가이드

`org.springdoc:springdoc-openapi-starter-webmvc-ui`를 사용하면 Spring Boot 애플리케이션에서 Swagger UI와 OpenAPI 3.x 명세를 자동으로 생성할 수 있습니다.

---

## 1. 의존성 추가

### Gradle (`build.gradle`)

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:3.0.3'
}
```

> `spring-boot-starter-web`은 **필수**입니다. springdoc은 Spring MVC 웹 서버 환경에서만 동작합니다.

### Maven (`pom.xml`)

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>3.0.3</version>
</dependency>
```

---

## 2. 제약 사항: 반드시 Web Server 모드로 실행해야 함

springdoc은 **Servlet 기반 웹 서버**가 실행 중일 때만 동작합니다.  
`SpringApplication`에서 `WebApplicationType.NONE`으로 설정하면 내장 서버가 시작되지 않아 Swagger UI와 `/v3/api-docs` 엔드포인트 모두 접근 불가합니다.

### 잘못된 설정 (Web Server 비활성화)

```java
// SpringApplication.java — 내장 서버 없음, springdoc 동작 불가
new SpringApplicationBuilder(MyApp.class)
    .webApplicationType(WebApplicationType.NONE) // ❌
    .run(args);
```

### 올바른 설정 (Web Server 활성화)

```java
// SpringApplication.java — 내장 Servlet 서버 실행, springdoc 정상 동작
new SpringApplicationBuilder(MyApp.class)
    // webApplicationType 지정 생략 시 기본값 SERVLET ✅
    .run(args);
```

`application.yml`에서 포트를 지정합니다.

```yaml
server:
  port: 18080
```

의존성 추가 후 별도 설정 없이 아래 URL로 접근할 수 있습니다.

| 리소스 | URL |
|---|---|
| Swagger UI | `http://localhost:18080/swagger-ui/index.html` |
| OpenAPI JSON | `http://localhost:18080/v3/api-docs` |

---

## 3. Controller 필수 어노테이션 목록

| 어노테이션 | 위치 | 역할 |
|---|---|---|
| `@Tag` | 클래스 | Controller 단위 그룹명과 설명 |
| `@Operation` | 메서드 | API 엔드포인트 요약/상세 설명 |
| `@Parameter` | 파라미터 | 개별 파라미터 설명 및 예시 |
| `@RequestBody` (OAS) | 파라미터 | Request body 설명 (선택) |
| `@ApiResponse` | 메서드 | HTTP 응답 코드별 설명 (선택) |
| `@Schema` | DTO 필드 | 모델 필드 설명 및 예시 (선택) |

---

## 4. 어노테이션 사용 예제

### 4-1. `@Tag` — Controller 그룹 지정

```java
import io.swagger.v3.oas.annotations.tags.Tag;

@RestController
@RequestMapping("/files")
@Tag(name = "File Management", description = "APIs for S3 file upload/download/delete")
public class FileController {
    // ...
}
```

---

### 4-2. `@Operation` — 엔드포인트 설명

```java
import io.swagger.v3.oas.annotations.Operation;

@GetMapping("/presign-get")
@Operation(
    summary = "Issue GET presigned URL",
    description = "Issues a presigned GET URL for downloading a single object. URL expires after the specified duration."
)
public PresignResult presignGetObject(...) {
    // ...
}
```

---

### 4-3. `@Parameter` — 쿼리/경로 파라미터 설명

`@RequestParam`, `@PathVariable` 등 개별 파라미터에 붙입니다.

```java
import io.swagger.v3.oas.annotations.Parameter;

@GetMapping("/presign-get")
@Operation(summary = "Issue GET presigned URL", description = "Issues a presigned GET URL.")
public PresignResult presignGetObject(
    @Parameter(description = "Bucket name", example = "my-bucket")
    @RequestParam String bucket,

    @Parameter(description = "Object key", example = "images/2026/05/a.png")
    @RequestParam String key,

    @Parameter(description = "URL expiration time in seconds (max: 604800)", example = "300")
    @RequestParam(required = false) Integer expiresInSeconds
) {
    // ...
}
```

---

### 4-4. `@io.swagger.v3.oas.annotations.parameters.RequestBody` — Request Body 설명

Spring의 `@RequestBody`와 별개로 OAS 전용 어노테이션을 사용합니다.

```java
import io.swagger.v3.oas.annotations.parameters.RequestBody;

@PostMapping("/presign-put")
@Operation(summary = "Issue PUT presigned URL", description = "Issues a presigned PUT URL.")
public PresignResult presignPutObject(
    @RequestBody(description = "PUT presign request payload", required = true)
    @org.springframework.web.bind.annotation.RequestBody PutObjectUrlRequest request
) {
    // ...
}
```

> 단순 케이스에서는 `@RequestBody` (OAS)를 생략해도 springdoc이 Spring의 `@RequestBody`를 자동 인식합니다.

---

### 4-5. `@ApiResponse` — 응답 코드 명세

```java
import io.swagger.v3.oas.annotations.responses.ApiResponse;
import io.swagger.v3.oas.annotations.responses.ApiResponses;

@GetMapping("/presign-get")
@Operation(summary = "Issue GET presigned URL", description = "Issues a presigned GET URL.")
@ApiResponses({
    @ApiResponse(responseCode = "200", description = "Presigned URL issued successfully"),
    @ApiResponse(responseCode = "400", description = "Invalid request parameter"),
    @ApiResponse(responseCode = "500", description = "Internal server error")
})
public PresignResult presignGetObject(...) {
    // ...
}
```

---

### 4-6. `@Schema` — DTO 필드 설명

```java
import io.swagger.v3.oas.annotations.media.Schema;

@Schema(description = "PUT presign request")
public record PutObjectUrlRequest(

    @Schema(description = "Bucket name", example = "my-bucket")
    String bucket,

    @Schema(description = "Object key", example = "uploads/2026/05/file.jpg")
    String key,

    @Schema(description = "Content-Type of the object", example = "image/jpeg")
    String contentType,

    @Schema(description = "URL expiration time in seconds (max: 604800)", example = "300")
    Integer expiresInSeconds
) {}
```

---

## 5. 프로젝트 적용 예시 (S3PresignerController)

이 프로젝트의 `S3PresignerController.java`는 위 어노테이션을 아래와 같이 조합하여 사용합니다.

```java
@RestController
@RequestMapping
@RequiredArgsConstructor
@Tag(name = "S3 Presigner", description = "API for issuing Ceph/S3 presigned URLs")
public class S3PresignerController {

    @PostMapping("/files/presign-put")
    @Operation(
        summary = "Issue PUT presigned URL",
        description = "Issues a presigned PUT URL for uploading a single object."
    )
    public S3PresignerFacade.PresignResult presignPutObject(
        @org.springframework.web.bind.annotation.RequestBody S3PresignerFacade.PutObjectUrlRequest request
    ) { ... }

    @GetMapping("/files/presign-get")
    @Operation(
        summary = "Issue GET presigned URL",
        description = "Issues a presigned GET URL for downloading a single object."
    )
    public S3PresignerFacade.PresignResult presignGetObject(
        @Parameter(description = "Bucket name", example = "my-bucket")
        @RequestParam String bucket,

        @Parameter(description = "Object key", example = "images/2026/05/a.png")
        @RequestParam String key,

        @Parameter(description = "Response Content-Type (optional)", example = "image/png")
        @RequestParam(required = false) String responseContentType,

        @Parameter(description = "URL expiration time in seconds (max: 604800)", example = "300")
        @RequestParam(required = false) Integer expiresInSeconds
    ) { ... }
}
```

## 6. Parameter에 사용 가능한 값 목록 출력

### 6-1. Request 파라미터의 타입에 enum 타입 직접 사용

- enum 값 목록 자동 노출, 유지보수에 가장 좋음

```java
public enum Status {
    READY, RUNNING, DONE
}

@GetMapping("/jobs")
public List<JobDto> getJobs(@RequestParam Status status) {
    ...
}
```

### 6-2. enum 클래스를 직접 연결

```java
@Parameter(schema = @Schema(implementation = Status.class))
```


### 6-3. String 목록 지정

- 파라미터가 String이어야 한다면 annotation으로 명시 가능

```java
@GetMapping("/jobs")
public List<JobDto> getJobs(
    @Parameter(
        description = "상태",
        schema = @Schema(allowableValues = {"READY", "RUNNING", "DONE"})
    )
    @RequestParam String status
) {
    ...
}
```

---

## 6. 문서 확인 방법

### 6-1. 서버 실행

```bash
# Gradle로 실행
gradle bootRun

# 또는 빌드 후 JAR 직접 실행
gradle assemble
java -jar build/libs/*.jar
```

### 6-2. 브라우저에서 Swagger UI 접속

서버가 정상 기동되면 브라우저에서 아래 URL로 접속합니다.

```
http://localhost:18080/swagger-ui/index.html
```

Swagger UI에서 등록된 모든 API 엔드포인트를 확인하고, **Try it out** 버튼으로 직접 요청을 실행해볼 수 있습니다.

### 6-3. OpenAPI JSON 명세 확인

```
http://localhost:18080/v3/api-docs
```

Raw JSON 형태의 OpenAPI 3.x 명세가 반환됩니다. 이 JSON을 다른 도구(Postman, Insomnia 등)에 임포트하거나 코드 생성에 활용할 수 있습니다.

### 6-4. curl로 확인 (CLI)

```bash
# Swagger UI HTML 응답 확인
curl -sS -o /dev/null -w '%{http_code}\n' http://localhost:18080/swagger-ui/index.html
# 정상: 200

# OpenAPI 명세 JSON 다운로드
curl -sS http://localhost:18080/v3/api-docs | python3 -m json.tool | head -30
```

---

## 7. 참고 링크

- [springdoc-openapi 공식 문서](https://springdoc.org/)
- [Swagger/OAS 어노테이션 레퍼런스](https://github.com/swagger-api/swagger-core/wiki/Swagger-2.X---Annotations)
- [OpenAPI 3.x 명세](https://spec.openapis.org/oas/v3.1.0)
