# How to add Filter in Spring Web
- [How to add Filter in Spring Web](#how-to-add-filter-in-spring-web)
  - [1. **`@Component`를 사용한 자동 등록 (Spring Boot)**](#1-component를-사용한-자동-등록-spring-boot)
  - [2. **`FilterRegistrationBean`을 사용한 수동 등록**](#2-filterregistrationbean을-사용한-수동-등록)
  - [3. **`@WebFilter`와 `@ServletComponentScan`을 사용한 등록 (Servlet 방식)**](#3-webfilter와-servletcomponentscan을-사용한-등록-servlet-방식)
  - [4. **`Filter`를 `DelegatingFilterProxy`로 Spring Security와 연동**](#4-filter를-delegatingfilterproxy로-spring-security와-연동)
    - [`SecurityFilter.java`](#securityfilterjava)
    - [`FilterConfig.java`](#filterconfigjava)
  - [**결론**](#결론)

---

Spring Web에서 `Filter`를 등록하는 방법에는 몇 가지가 있습니다. Spring Boot 및 기존 Spring MVC 환경에서 각각 다르게 적용할 수 있습니다.

---

## 1. **`@Component`를 사용한 자동 등록 (Spring Boot)**
Spring Boot 환경에서는 `@Component` 어노테이션을 사용하면 `Filter`가 자동으로 등록됩니다.

```java
import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.FilterConfig;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import org.springframework.stereotype.Component;
import java.io.IOException;

@Component
public class CustomFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("Custom Filter is executed.");
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {}
}
```
- `@Component` 덕분에 `Filter`가 자동으로 등록됩니다.
- `doFilter` 메서드에서 요청을 가로채서 필요한 처리를 수행한 후, `chain.doFilter(request, response);`를 호출하여 다음 필터 또는 서블릿으로 요청을 넘깁니다.

---

## 2. **`FilterRegistrationBean`을 사용한 수동 등록**
Spring Boot 환경에서 `FilterRegistrationBean`을 사용하면 `Filter`의 순서 및 URL 패턴을 설정할 수 있습니다.

```java
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<CustomFilter> loggingFilter() {
        FilterRegistrationBean<CustomFilter> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new CustomFilter());
        registrationBean.addUrlPatterns("/api/*"); // 특정 URL 패턴에만 적용
        registrationBean.setOrder(1); // 필터 순서 지정
        return registrationBean;
    }
}
```
- 특정 URL 패턴(`/api/*`)에만 필터를 적용할 수 있습니다.
- 여러 개의 필터가 있을 때 `setOrder(int order)`를 이용해 실행 순서를 설정할 수 있습니다.

---

## 3. **`@WebFilter`와 `@ServletComponentScan`을 사용한 등록 (Servlet 방식)**
Spring Boot가 아닌 기존 Servlet 기반 Spring MVC 환경에서는 `@WebFilter`를 사용하여 필터를 등록할 수 있습니다.

```java
import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.FilterConfig;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.annotation.WebFilter;
import java.io.IOException;

@WebFilter(urlPatterns = "/api/*") // 특정 URL 패턴 지정
public class CustomWebFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("Custom WebFilter is executed.");
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {}
}
```
- `@WebFilter`를 사용하면 특정 URL 패턴에만 필터를 적용할 수 있습니다.
- `@ServletComponentScan`을 `@SpringBootApplication` 클래스에 추가해야 합니다.

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;

@SpringBootApplication
@ServletComponentScan // @WebFilter 사용을 위해 필요
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---

## 4. **`Filter`를 `DelegatingFilterProxy`로 Spring Security와 연동**
Spring Security와 같은 Spring의 다른 기능과 통합하려면 `DelegatingFilterProxy`를 사용할 수 있습니다.

### `SecurityFilter.java`
```java
import jakarta.servlet.Filter;
import jakarta.servlet.FilterChain;
import jakarta.servlet.FilterConfig;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import java.io.IOException;

public class SecurityFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) {}

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        System.out.println("Security Filter is executed.");
        chain.doFilter(request, response);
    }

    @Override
    public void destroy() {}
}
```

### `FilterConfig.java`
```java
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.filter.DelegatingFilterProxy;

@Configuration
public class FilterConfig {

    @Bean
    public FilterRegistrationBean<DelegatingFilterProxy> securityFilter() {
        FilterRegistrationBean<DelegatingFilterProxy> registrationBean = new FilterRegistrationBean<>();
        registrationBean.setFilter(new DelegatingFilterProxy("securityFilterBean"));
        registrationBean.addUrlPatterns("/*");
        registrationBean.setOrder(1);
        return registrationBean;
    }

    @Bean(name = "securityFilterBean")
    public Filter securityFilterBean() {
        return new SecurityFilter();
    }
}
```
- `DelegatingFilterProxy`를 사용하면 Spring의 `@Bean`으로 등록한 필터를 쉽게 관리할 수 있습니다.

---

## **결론**
- 간단한 필터는 `@Component`를 사용하여 자동 등록.
- URL 패턴 및 순서를 조정하려면 `FilterRegistrationBean` 사용.
- Servlet 기반에서는 `@WebFilter` + `@ServletComponentScan` 활용.
- Spring Security 같은 프레임워크와 연동하려면 `DelegatingFilterProxy` 사용.

어떤 방식이든 `doFilter()` 메서드 내부에서 원하는 로직을 추가하여 요청을 가로채거나 변환할 수 있습니다.