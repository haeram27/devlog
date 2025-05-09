
# tomcat http error page configuration

## add common-error.html

path: `{project.basedir}/resource/public/error/error.html`

```html
<!DOCTYPE html>
<html>

<head>
    <title>Error</title>
    <style>
        html {
            color-scheme: light dark;
        }

        body {
            height: 100%;
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
</head>

<body>
    <h1>An error occurred.</h1>
    <p>Sorry, the page you are looking for is currently unavailable.<br /> Please try again later.</p>
    <p>If you are the system administrator of this resource then you should check the error log for details.</p>
</body>

</html>
```

## disable whitelabel error page on spring-boot-web application

If error-page settings are enabled in both of `tomcat` and `spring-boot-web` then spring-boot-web handle error firstly. so if tomcat needs to handle error-page then `spring-boot-web` http error handling SHOLD be disabled.

- application.yml
```yml
server:
  error:
    whitelabel:
      enabled: false
# error handler default path (default /error)
# server.error.path=/error
```

## configure error-page settings programatically (jar)

Spring Boot의 실행 가능한 JAR(내장 Tomcat) 환경에서는 더 이상 `WEB-INF/web.xml`을 사용하지 않고, 다음 두 가지 방법으로 에러 페이지를 대체합니다.

---

## 1. 정적 HTML 배치로 간단히 처리하기

Spring Boot는 기본적으로 **`/error`** 경로로 들어오는 요청을 처리하며, `src/main/resources/{static,public,resources,META-INF/resources}` 아래에 다음과 같은 위치에 HTML 파일을 두면 자동으로 매핑해 줍니다. 즉, `src/main/resources/static/error/error.html` 로 파일을 두면, 외부에서 `/error/error.html` URL 로 접근할 수 있어야 합니다.

```
src/main/resources/
 ├─ static/
 │   └─ error/
 │       ├─ 404.html
 │       ├─ 500.html
 │       └─ error.html   ← (기본 예외 핸들러용)
 └─ templates/
     └─ error/
         ├─ 404.html
         ├─ 500.html
         └─ error.html
```

- **404 Not Found** → `/error/404.html`
- **500 Internal Server Error** → `/error/500.html`
- 그 외 예외    → `/error/error.html`

추가로 `application.properties` 에서 흰색 라벨 페이지를 끄고 경로를 조정할 수 있습니다.

```properties
# 스프링 기본 에러 페이지 비활성화
server.error.whitelabel.enabled=false
# 에러 핸들러 기본 경로 (기본값 /error)
# server.error.path=/error
```

---

## 2. Java Config로 커스터마이징 (ErrorPage 등록)

더 세밀하게 상태코드나 예외별로 경로를 매핑하고 싶다면, `WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>` 빈을 등록해서 `ErrorPage` 를 직접 추가할 수 있습니다.

```java
import org.springframework.boot.web.server.*;
import org.springframework.boot.web.servlet.server.*;
import org.springframework.context.annotation.*;
import org.springframework.http.HttpStatus;

@Configuration
public class ErrorPageConfig {

    @Bean
    public WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> webServerFactoryCustomizer() {
        return factory -> {
            // 상태 코드별 매핑
			factory.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.UNAUTHORIZED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.PAYMENT_REQUIRED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.FORBIDDEN, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.METHOD_NOT_ALLOWED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.NOT_ACCEPTABLE, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.PROXY_AUTHENTICATION_REQUIRED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.REQUEST_TIMEOUT, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.CONFLICT, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.GONE, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.LENGTH_REQUIRED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.PRECONDITION_FAILED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.PAYLOAD_TOO_LARGE, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.URI_TOO_LONG, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.UNSUPPORTED_MEDIA_TYPE, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.REQUESTED_RANGE_NOT_SATISFIABLE, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.EXPECTATION_FAILED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.I_AM_A_TEAPOT, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.UNPROCESSABLE_ENTITY, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.LOCKED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.FAILED_DEPENDENCY, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.TOO_EARLY, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.UPGRADE_REQUIRED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.PRECONDITION_REQUIRED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.TOO_MANY_REQUESTS, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.REQUEST_HEADER_FIELDS_TOO_LARGE, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.UNAVAILABLE_FOR_LEGAL_REASONS, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.NOT_IMPLEMENTED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.BAD_GATEWAY, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.SERVICE_UNAVAILABLE, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.GATEWAY_TIMEOUT, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.HTTP_VERSION_NOT_SUPPORTED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.VARIANT_ALSO_NEGOTIATES, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.INSUFFICIENT_STORAGE, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.LOOP_DETECTED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.BANDWIDTH_LIMIT_EXCEEDED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.NOT_EXTENDED, "/resources/public/error/error.html"));
			factory.addErrorPages(new ErrorPage(HttpStatus.NETWORK_AUTHENTICATION_REQUIRED, "/resources/public/error/error.html"));
            // Exception Error mapping
			factory.addErrorPages(new ErrorPage(Throwable.class, "/resources/public/error/error.html"));
            // default error page
			factory.addErrorPages(new ErrorPage("/resources/public/error/error.html"));
        };
    }
}
```

- `/custom-404.html`, `/custom-500.html` 등은 위에서 설명한 대로 `static/` 또는 `templates/` 에 두시면 됩니다.
- `ErrorPage` 생성자에는 `HttpStatus` 외에 `Exception.class` 도 지정할 수 있습니다.

---

## 3. @ControllerAdvice + @ExceptionHandler 활용

특정 예외를 잡아서 JSON 혹은 뷰로 응답하고 싶다면, `@ControllerAdvice` 와 `@ExceptionHandler` 로도 처리 가능합니다.

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NoHandlerFoundException.class)
    public String handle404() {
        return "error/404";  // templates/error/404.html 로 포워드
    }

    @ExceptionHandler(Exception.class)
    public String handle500(Exception ex, Model model) {
        model.addAttribute("message", ex.getMessage());
        return "error/500";  // templates/error/500.html 로 포워드
    }
}
```

---

### 요약

- **정적 HTML** 만으로 충분하다면 `src/main/resources/{static,public,resources,META-INF/resources}` 등 위치에 파일을 두고, 프로퍼티로 화이트라벨을 끄세요.
- **세밀한 코드 매핑** 이 필요하면 `WebServerFactoryCustomizer` 를 이용해 `ErrorPage` 를 등록하세요.
- **예외별 로직** 이 복잡하면 `@ControllerAdvice` + `@ExceptionHandler` 조합을 사용하시면 됩니다.

## configure error-page settings in web.xml (war only)

path: /WEB-INF/web.xml

```xml
...
<!--error page setting-->
<error-page>
    <error-code>400</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>401</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>402</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>403</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>404</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>405</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>406</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>408</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>409</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>410</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>411</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>412</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>413</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>414</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>415</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>416</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>421</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>429</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>497</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>500</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>501</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>502</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>503</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>504</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>505</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>
<error-page>
    <error-code>507</error-code>
    <location>/resources/public/error/error.html</location>
</error-page>

<error-page>
    <exception-type>java.lang.Throwable</exception-type>
    <location>/resources/public/error/error.html</location>
</error-page>

<!--default error page-->
<error-page>
    <location>/resources/public/error/error.html</location>
</error-page>
...
```
