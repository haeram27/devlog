# spring server properties

### server.tomcat

Tomcat 서버 설정에 관련된 properties입니다.

#### 주요 설정값

```yml
server:
  shutdown: graceful # 애플리케이션 종료 시 요청을 정상적으로 처리하고 종료 (기본값: immediate)
  
  tomcat:
    # === 스레드 관련 설정 ===
    threads:
      max: 200 # 최대 워커 스레드 수. 요청 처리를 위해 사용할 최대 스레드 수 (기본값: 200)
      min-spare: 10 # 최소 유휴 스레드 수. 준비 상태를 유지하는 최소 스레드 수 (기본값: 10)
      max-queue-capacity: 2147483647 # 대기 큐의 최대 용량. 모든 스레드가 사용 중일 때 대기할 수 있는 요청 수 (기본값: 2^31-1)
    
    # === 커넥션 관련 설정 ===
    max-connections: 8192 # 동시 연결의 최대 수. HTTP 커넥션에서 허용할 동시 연결 수 제한 (기본값: 8192)
    accept-count: 100 # 최대 연결 대기 큐 크기. 모든 스레드가 사용 중일 때 대기할 수 있는 연결 수 (기본값: 100)
    connection-timeout: 60000 # 서버 소켓이 연결을 대기하는 최대 시간 (밀리초, 기본값: 60000ms)
    keep-alive-timeout: 30000 # Keep-Alive 커넥션이 유지되는 최대 시간 (밀리초, 기본값: 30000ms)
    max-keep-alive-requests: 100 # 단일 Keep-Alive 연결에서 처리할 최대 요청 수 (기본값: 100)
    
    # === 헤더/바디 크기 제한 ===
    max-http-header-size: 8192 # HTTP 요청 및 응답 헤더의 최대 크기 (바이트, 기본값: 8192)
    max-http-post-size: 2097152 # POST 요청 바디의 최대 크기 (바이트, 기본값: 2MB)
    
    # === 성능 및 버퍼 관련 설정 ===
    max-swallow-size: 2097152 # 요청을 종료하기 전에 읽을 수 있는 최대 바이트 수 (기본값: 2MB)
    buffer-size: 16384 # 입출력 버퍼 크기 (바이트, 기본값: 16384)
    
    # === Processor 관련 설정 ===
    processor-cache: 200 # 재사용 가능한 Processor 객체 캐시 크기 (기본값: 200)
    background-process-delay: 10 # 백그라운드 프로세스 실행 간격 (초, 기본값: 10)
    
    # === URI 인코딩 ===
    uri-encoding: UTF-8 # URI 인코딩 방식 (기본값: UTF-8)
    
    # === 기타 설정 ===
    accesslog:
      enabled: true # Access 로그 활성화 여부 (기본값: false)
      directory: logs # Access 로그 디렉토리 (기본값: logs)
      pattern: "%h %l %u %t \"%r\" %s %b \"%{Referer}i\" \"%{User-Agent}i\"" # Access 로그 패턴
    
    relaxed-path-chars: "" # 추가로 허용할 URL 경로 문자들
    relaxed-query-chars: "" # 추가로 허용할 쿼리 문자들
```

#### 상세 설명

| 설정 | 설명 | 기본값 |
|------|------|--------|
| `threads.max` | 요청 처리용 최대 스레드 수 | 200 |
| `threads.min-spare` | 항상 대기하는 최소 스레드 수 | 10 |
| `threads.max-queue-capacity` | 작업 대기 큐 최대 크기 | 2,147,483,647 |
| `max-connections` | 동시 연결 수 제한 | 8192 |
| `accept-count` | 대기 큐 크기 | 100 |
| `connection-timeout` | 연결 타임아웃 (ms) | 60000 |
| `keep-alive-timeout` | Keep-Alive 타임아웃 (ms) | 30000 |
| `max-keep-alive-requests` | Keep-Alive 연결당 최대 요청 수 | 100 |
| `max-http-header-size` | 헤더 최대 크기 (바이트) | 8192 |
| `max-http-post-size` | POST 바디 최대 크기 (바이트) | 2,097,152 |

### spring.datasource.hikari

HikariCP 연결 풀 설정에 관련된 properties입니다.

#### 주요 설정값

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb # 데이터베이스 JDBC URL
    username: root # 데이터베이스 사용자명
    password: password # 데이터베이스 비밀번호
    driver-class-name: com.mysql.cj.jdbc.Driver # JDBC 드라이버 클래스명

    hikari:
      # === 풀 크기 설정 ===
      maximum-pool-size: 10 # 풀의 최대 커넥션 수 (기본값: 10)
      minimum-idle: 10 # 풀에 유지할 최소 유휴 커넥션 수 (기본값: -1 = maximum-pool-size와 동일)

      # === 타임아웃 설정 ===
      connection-timeout: 30000 # 풀에서 커넥션을 가져오기 위한 최대 대기 시간 (ms, 기본값: 30000)
      idle-timeout: 600000 # 유휴 커넥션을 풀에서 제거하기까지의 시간 (ms, 기본값: 600000 = 10분)
      max-lifetime: 1800000 # 커넥션이 풀에서 제거되기 전 최대 사용 가능 시간 (ms, 기본값: 1800000 = 30분)
      validation-timeout: 5000 # 커넥션 유효성 검사 타임아웃 (ms, 기본값: 5000)
      leak-detection-threshold: 0 # 커넥션 누수 감지 임계값 (ms, 기본값: 0 = 비활성화)
      initialization-fail-timeout: 1 # 초기 커넥션 생성 실패 시 재시도 대기 시간 (초, 기본값: 1)

      # === 커넥션 테스트 ===
      connection-test-query: null # 커넥션 유효성 검사 SQL 쿼리 (기본값: null = 드라이버 기본값 사용)

      # === 커넥션 설정 ===
      read-only: false # 풀에서 가져온 커넥션을 read-only 모드로 설정할지 여부 (기본값: false)
      auto-commit: true # 자동 커밋 활성화 여부 (기본값: true)
      transaction-isolation: null # 트랜잭션 격리 수준 (기본값: null = 드라이버 기본값)

      # === 연결 풀 이름 ===
      pool-name: HikariPool-1 # 연결 풀 이름 (기본값: HikariPool-1)

      # === 모니터링 ===
      register-mbeans: false # JMX MBean 등록 여부 (기본값: false)
      metrics-tracker-factory: null # 메트릭 추적 팩토리 (기본값: null)

      # === 드라이버별 설정 ===
      allow-pool-suspension: false # 풀 일시 중단 허용 여부 (기본값: false)
      suspend-timeout: -1 # 풀 일시 중단 타임아웃 (ms, 기본값: -1 = 무제한)

      # === 커넥션 로깅 ===
      connection-timeout-message: null # 타임아웃 시 사용자 지정 메시지

      # === 멀티테넌시 ===
      schema: null # 기본 스키마 (기본값: null)
      catalog: null # 카탈로그 (기본값: null)
```

#### 상세 설명

| 설정 | 설명 | 기본값 | 권장값 |
|------|------|--------|--------|
| `maximum-pool-size` | 풀의 최대 커넥션 수 | 10 | CPU 코어 수 × 2 + 유휴 커넥션 |
| `minimum-idle` | 최소 유휴 커넥션 수 | -1 | maximum-pool-size와 동일 권장 |
| `connection-timeout` | 커넥션 획득 타임아웃 (ms) | 30000 | 30000 (30초) |
| `idle-timeout` | 유휴 커넥션 제거 시간 (ms) | 600000 | 600000 (10분) |
| `max-lifetime` | 커넥션 최대 수명 (ms) | 1800000 | 1800000 (30분) |
| `validation-timeout` | 커넥션 검증 타임아웃 (ms) | 5000 | 5000 (5초) |
| `connection-test-query` | 커넥션 테스트 SQL | null | DB 드라이버 기본값 사용 |
| `read-only` | Read-only 모드 설정 | false | 필요에 따라 설정 |
| `auto-commit` | 자동 커밋 | true | true |
| `leak-detection-threshold` | 누수 감지 임계값 (ms) | 0 | 60000 (프로드에서) |
| `register-mbeans` | JMX 모니터링 | false | true (모니터링 필요 시) |

#### 주의사항

1. **max-lifetime과 DB 타임아웃**
   - DB의 연결 타임아웃(보통 8시간)보다 작게 설정하여야 함
   - 권장: DB 타임아웃 - 30분

2. **커넥션 풀 크기 결정**
   - 일반적인 계산식: (Core × 2) + Spindle
   - 예: 4 Core CPU → 8~12 커넥션 권장

3. **idle-timeout vs max-lifetime**
   - idle-timeout: 유휴 상태인 커넥션 제거 시간
   - max-lifetime: 모든 커넥션의 최대 수명

4. **validation-timeout**
   - connection-test-query가 설정되었을 때만 적용
   - 너무 짧으면 유효한 커넥션도 실패 판정 가능

---

## server (일반 서버 설정)

Spring Boot 내장 서버 관련 기본 설정입니다.

#### 주요 설정값

```yml
server:
  port: 8080 # 서버 포트 (기본값: 8080)
  
  servlet:
    context-path: /api # 애플리케이션 컨텍스트 경로 (기본값: "")
    session:
      timeout: 30m # 세션 타임아웃 (기본값: 30분)
      cookie:
        name: JSESSIONID # 세션 쿠키 이름 (기본값: JSESSIONID)
        domain: localhost # 쿠키 도메인 (기본값: null)
        path: / # 쿠키 경로 (기본값: null)
        http-only: true # HttpOnly 플래그 (기본값: true)
        secure: false # Secure 플래그 - HTTPS에서만 전송 (기본값: false)
        max-age: -1 # 쿠키 최대 유지 시간 (초, 기본값: -1)

  error:
    include-message: always # 에러 메시지 포함 여부 (NEVER, WHEN_PARAM, ALWAYS, 기본값: NEVER)
    include-stacktrace: always # 스택트레이스 포함 여부 (기본값: ON_PARAM)
    include-exception: false # 예외 클래스명 포함 여부 (기본값: false)
    include-binding-errors: always # 바인딩 에러 포함 여부 (기본값: ON_PARAM)
    path: /error # 에러 페이지 경로 (기본값: /error)

  compression:
    enabled: true # 응답 압축 활성화 (기본값: true)
    min-response-size: 1024 # 압축할 최소 응답 크기 (바이트, 기본값: 1024)
    mime-types: # 압축할 MIME 타입
      - text/html
      - text/xml
      - text/plain
      - text/css
      - text/javascript
      - application/javascript
      - application/json
    excluded-mime-types: # 제외할 MIME 타입
      - image/jpeg
      - image/png

  http2:
    enabled: true # HTTP/2 활성화 (기본값: true)
```

#### 상세 설명

| 설정 | 설명 | 기본값 |
|------|------|--------|
| `port` | 서버 포트 | 8080 |
| `servlet.context-path` | 애플리케이션 루트 경로 | "" |
| `servlet.session.timeout` | 세션 타임아웃 | 30m |
| `error.include-message` | 에러 메시지 포함 여부 | NEVER |
| `compression.enabled` | 응답 압축 | true |
| `compression.min-response-size` | 압축 최소 크기 (바이트) | 1024 |

---

## spring.jpa (JPA/Hibernate 설정)

데이터베이스 ORM 관련 설정입니다.

#### 주요 설정값

```yml
spring:
  jpa:
    hibernate:
      ddl-auto: validate # DDL 자동 실행 (create, create-drop, update, validate, none, 기본값: none)
      format-sql: true # SQL 포맷팅 활성화 (기본값: false)
      use-new-id-generator-mappings: true # 새 ID 생성 매핑 사용 (기본값: true)
      jdbc:
        batch-size: 20 # JDBC 배치 처리 크기 (기본값: 15)
        fetch-size: 50 # JDBC fetch 크기 (기본값: -1 = 드라이버 기본값)
      properties:
        hibernate:
          dialect: org.hibernate.dialect.MySQL8Dialect # Hibernate 방언 지정
          show-sql: false # SQL 출력 여부 (기본값: false)
          use-sql-comments: false # SQL 코멘트 출력 (기본값: false)
          generate-statistics: false # 통계 정보 생성 (기본값: false)
          order-inserts: true # INSERT 문 순서 유지 (기본값: false)
          order-updates: true # UPDATE 문 순서 유지 (기본값: false)
          jdbc:
            batch-versioned-data: true # 배치 처리 시 버전 정보 포함 (기본값: false)
          enable-lazy-load-no-trans: false # 트랜잭션 외부 지연로딩 (기본값: false)

    show-sql: false # SQL 로그 출력 (기본값: false, properties.show-sql 권장)
    properties:
      javax:
        persistence:
          sharedCache:
            mode: ALL # 공유 캐시 모드 (NONE, ENABLE_SELECTIVE, DISABLE_SELECTIVE, ALL, 기본값: UNSPECIFIED)

    open-in-view: true # Open Session In View 활성화 (기본값: true, 주의: N+1 문제 가능)
    database-platform: org.hibernate.dialect.MySQL8Dialect # 데이터베이스 플랫폼
```

#### 상세 설명

| 설정 | 설명 | 기본값 | 값 설명 |
|------|------|--------|---------|
| `ddl-auto` | DDL 자동 실행 정책 | none | create(항상 생성), update(변경만 반영), validate(검증만) |
| `show-sql` | SQL 출력 | false | true = 콘솔에 SQL 출력 |
| `jdbc.batch-size` | 배치 크기 | 15 | DB별로 조정 필요 (보통 10-30) |
| `open-in-view` | View 렌더링 중 세션 유지 | true | false 권장 (N+1 방지) |

#### 주의사항

1. **open-in-view는 false 권장**
   - true일 경우 View 렌더링 중 DB 쿼리 발생 가능 (N+1 문제)
   - `@Transactional(readOnly=true)` 사용 시 false 권장

2. **ddl-auto 설정**
   - 개발: update 또는 create-drop
   - 테스트: create-drop
   - 프로덕션: validate (수동으로 스키마 관리)

3. **방언(dialect) 설정**
   - MySQL: MySQL8Dialect (MySQL 8.0 이상)
   - PostgreSQL: PostgreSQL10Dialect
   - Oracle: Oracle12cDialect

---

## spring.jackson (JSON 직렬화 설정)

JSON 변환 관련 설정입니다.

#### 주요 설정값

```yml
spring:
  jackson:
    default-property-inclusion: non_null # null 값 포함 여부 (always, non_null, non_absent, non_empty, 기본값: always)
    serialization:
      write-dates-as-timestamps: false # 날짜를 타임스탬프로 변환 (기본값: true)
      indent-output: false # JSON 들여쓰기 (기본값: false)
      fail-on-empty-beans: false # 빈 객체 직렬화 실패 여부 (기본값: true)
      write-enums-using-to-string: false # Enum을 toString()으로 직렬화 (기본값: false)

    deserialization:
      fail-on-unknown-properties: false # 알 수 없는 속성 역직렬화 실패 여부 (기본값: false)
      fail-on-null-for-primitives: false # null을 원시 타입에 매핑 시 실패 여부 (기본값: false)
      accept-single-value-as-array: false # 배열이 아닌 값을 배열로 처리 (기본값: false)

    time-zone: UTC # 시간대 (기본값: UTC)
    date-format: yyyy-MM-dd'T'HH:mm:ss.SSS'Z' # 날짜 포맷 (기본값: ISO-8601)

    parser:
      allow-unquoted-control-chars: false # 인용되지 않은 제어 문자 허용 (기본값: false)
      allow-single-quotes: false # 작은따옴표 허용 (기본값: false)
      allow-comments: false # JSON 주석 허용 (기본값: false)

    generator:
      write-bigdecimal-as-plain: false # BigDecimal을 plain 형식으로 (기본값: false)
```

#### 상세 설명

| 설정 | 설명 | 기본값 | 권장값 |
|------|------|--------|--------|
| `default-property-inclusion` | null 값 포함 | always | non_null (응답 크기 감소) |
| `write-dates-as-timestamps` | 날짜 타임스탬프 | true | false (ISO 형식) |
| `fail-on-unknown-properties` | 알 수 없는 속성 | false | false (하위 호환성) |
| `time-zone` | 시간대 | UTC | Asia/Seoul |

#### 예시

```yml
# null 값 제외한 JSON 응답 예시
# default-property-inclusion: non_null

# 입력:
# { "name": "John", "email": null }

# 출력:
# { "name": "John" }
```

---

## logging (로깅 설정)

애플리케이션 로그 레벨 및 출력 설정입니다.

#### 주요 설정값

```yml
logging:
  level:
    root: INFO # 루트 로거 레벨 (기본값: INFO)
    org.springframework: INFO # Spring 프레임워크 (기본값: INFO)
    org.springframework.web: DEBUG # Spring Web (기본값: INFO)
    org.springframework.security: DEBUG # Spring Security (기본값: INFO)
    org.hibernate: INFO # Hibernate (기본값: INFO)
    org.hibernate.SQL: DEBUG # Hibernate SQL 출력 (기본값: INFO)
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE # Hibernate 바인딩 파라미터 (기본값: TRACE)
    com.zaxxer.hikari: INFO # HikariCP (기본값: INFO)
    com.zaxxer.hikari.pool.HikariPool: DEBUG # HikariCP 풀 (기본값: INFO)

    # 패키지별 로그 레벨
    com.example.app: DEBUG
    com.example.app.controller: INFO
    com.example.app.service: DEBUG
    com.example.app.repository: DEBUG

  file:
    name: logs/application.log # 로그 파일 경로 (기본값: null = 파일 출력 안 함)
    max-size: 10MB # 롤링 파일 최대 크기 (기본값: 10MB)
    max-history: 10 # 보관 기간 (일, 기본값: 7)
    total-size-cap: 1GB # 전체 로그 크기 제한 (기본값: unlimited)

  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %logger{36} - %msg%n" # 콘솔 로그 패턴
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n" # 파일 로그 패턴

  logback:
    rollingpolicy:
      max-file-size: 10MB # 롤링 파일 크기 (기본값: 10MB)
      max-history: 10 # 보관 기간 (기본값: 7)
      total-size-cap: 1GB # 전체 크기 (기본값: unlimited)
      file-name-pattern: logs/application-%d{yyyy-MM-dd}.%i.log # 파일명 패턴
```

#### 상세 설명

| 로그 레벨 | 설명 | 사용 시점 |
|-----------|------|----------|
| TRACE | 상세 정보 | 디버깅용 (프로덕션 X) |
| DEBUG | 디버그 정보 | 개발/테스트 환경 |
| INFO | 일반 정보 | 프로덕션 (기본) |
| WARN | 경고 | 예상 밖의 상황 |
| ERROR | 에러 | 오류 발생 |

#### 로그 패턴 문법

```
%d{yyyy-MM-dd HH:mm:ss} - 날짜/시간
%logger{36}                - 로거 클래스명 (최대 36자)
%thread                    - 스레드명
%-5level                   - 로그 레벨 (좌측 정렬, 5자)
%msg                       - 로그 메시지
%n                         - 줄바꿈
```

---

## spring.mvc (Spring MVC 설정)

웹 요청 처리 관련 설정입니다.

#### 주요 설정값

```yml
spring:
  mvc:
    # === 경로 처리 ===
    static-path-pattern: /static/** # 정적 리소스 경로 (기본값: /static/**)
    web-path-pattern: /static/ # 정적 웹 리소스 경로

    # === 날짜/시간 포맷 ===
    format:
      date: yyyy-MM-dd # 날짜 포맷 (기본값: yyyy-MM-dd)
      time: HH:mm:ss # 시간 포맷 (기본값: HH:mm:ss)
      date-time: yyyy-MM-dd'T'HH:mm:ss # 날짜/시간 포맷

    # === 콘텐츠 협상 ===
    content-negotiation:
      favor-parameter: false # URL 파라미터로 콘텐츠 타입 선택 (기본값: false)
      favor-path-extension: false # URL 확장자로 콘텐츠 타입 선택 (기본값: false)
      media-types:
        json: application/json
        xml: application/xml

    # === 비동기 처리 ===
    async:
      request-timeout: 30000 # 비동기 요청 타임아웃 (ms, 기본값: null = 30초)

    # === 뷰 설정 ===
    view:
      prefix: /WEB-INF/views/ # 뷰 프리픽스 (기본값: "")
      suffix: .jsp # 뷰 서픽스 (기본값: "")

    # === 기타 ===
    throw-exception-if-no-handler-found: false # 핸들러 없을 시 예외 발생 (기본값: false)
    ignore-default-model-on-redirect: true # 리다이렉트 시 기본 모델 무시 (기본값: true)
    log-request-details: false # 요청 상세 로깅 (기본값: false, 프로드X)
    locale: ko_KR # 기본 로케일 (기본값: 시스템 기본값)
    locale-resolver: fixed # 로케일 결정 방식 (fixed, cookie, header, 기본값: header)
```

#### 상세 설명

| 설정 | 설명 | 기본값 |
|------|------|--------|
| `date` 포맷 | 요청/응답 날짜 포맷 | yyyy-MM-dd |
| `time` 포맷 | 요청/응답 시간 포맷 | HH:mm:ss |
| `async.request-timeout` | 비동기 요청 타임아웃 (ms) | 30000 |
| `log-request-details` | 요청 상세 로깅 | false |

#### 주의사항

1. **정적 리소스 설정**
   - CSS, JS, 이미지 등은 별도의 경로로 설정하여 성능 최적화
   - `/static/**` 경로 사용 권장

2. **로케일 설정**
   - fixed: 고정된 로케일 사용
   - cookie: 쿠키에서 로케일 읽기
   - header: Accept-Language 헤더에서 읽기 (기본)

---

## spring.cache (캐시 설정)

데이터 캐싱 관련 설정입니다.

#### 주요 설정값

```yml
spring:
  cache:
    type: simple # 캐시 타입 (simple, redis, caffeine, jcache, couchbase, etc., 기본값: simple)
    cache-names: # 생성할 캐시 이름 목록
      - users
      - products
      - sessions

    simple:
      # Simple 캐시는 별도 설정 없음 (ConcurrentHashMap 사용)

    caffeine:
      spec: "maximumSize=500,expireAfterWrite=10m" # Caffeine 캐시 설정

    redis:
      time-to-live: 600000 # 캐시 TTL (ms, 기본값: null = 무제한)
      key-prefix: "cache:" # 캐시 키 프리픽스 (기본값: "")
      use-key-prefix: true # 키 프리픽스 사용 (기본값: true)
      cache-null-values: true # null 값 캐싱 (기본값: true)

    jcache:
      config: classpath:ehcache.xml # JCache 설정 파일
      provider: org.ehcache.jsr107.EhcacheCachingProvider # JCache 제공자
```

#### 상세 설명

| 캐시 타입 | 설명 | 용도 |
|-----------|------|------|
| simple | ConcurrentHashMap 기반 | 개발/테스트, 단일 서버 |
| redis | Redis 기반 | 프로덕션, 분산 환경 |
| caffeine | 로컬 고성능 캐시 | 고성능 로컬 캐싱 |
| jcache | 표준 JCache | 다양한 백엔드 지원 |

#### 사용 예시

```java
@Service
public class UserService {
    @Cacheable(value = "users", key = "#id")
    public User getUserById(Long id) { ... }
    
    @CachePut(value = "users", key = "#user.id")
    public User updateUser(User user) { ... }
    
    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) { ... }
}
```

---

## spring.redis (Redis 설정)

Redis 서버 연결 설정입니다.

#### 주요 설정값

```yml
spring:
  redis:
    # === 연결 설정 ===
    host: localhost # Redis 호스트 (기본값: localhost)
    port: 6379 # Redis 포트 (기본값: 6379)
    password: null # Redis 비밀번호 (기본값: null)
    username: default # Redis 사용자명 (기본값: default)
    database: 0 # 사용할 데이터베이스 인덱스 (기본값: 0)

    # === 클라이언트 설정 ===
    client-name: null # 클라이언트 이름 (기본값: null)
    client-type: lettuce # Redis 클라이언트 (lettuce, jedis, 기본값: lettuce)

    # === 타임아웃 설정 ===
    timeout: 2000ms # 연결 타임아웃 (기본값: null)
    connect-timeout: 2000ms # 연결 타임아웃 (기본값: null)

    # === Lettuce 설정 (권장) ===
    lettuce:
      pool:
        max-active: 8 # 최대 활성 연결 (기본값: 8)
        max-idle: 8 # 최대 유휴 연결 (기본값: 8)
        min-idle: 0 # 최소 유휴 연결 (기본값: 0)
        max-wait-duration: -1ms # 최대 대기 시간 (기본값: -1 = 무제한)
      shutdown-timeout: 100ms # 종료 타임아웃 (기본값: 100ms)

    # === Jedis 설정 ===
    jedis:
      pool:
        max-active: 8 # 최대 활성 연결 (기본값: 8)
        max-idle: 8 # 최대 유휴 연결 (기본값: 8)
        min-idle: 0 # 최소 유휴 연결 (기본값: 0)
        max-wait-duration: -1ms # 최대 대기 시간 (기본값: -1)

    # === SSL 설정 ===
    ssl: false # SSL 사용 여부 (기본값: false)
```

#### 상세 설명

| 설정 | 설명 | 기본값 | 권장값 |
|------|------|--------|--------|
| `host` | Redis 호스트 | localhost | 운영 환경에 맞게 설정 |
| `port` | Redis 포트 | 6379 | 6379 |
| `database` | DB 인덱스 | 0 | 환경별로 구분 (개발:0, 테스트:1) |
| `timeout` | 타임아웃 (ms) | null | 2000 |
| `client-type` | 클라이언트 | lettuce | lettuce (비동기, 성능 우수) |
| `pool.max-active` | 최대 연결 | 8 | 8-16 |

---

## management (Spring Boot Actuator 설정)

애플리케이션 모니터링 및 관리 엔드포인트 설정입니다.

#### 주요 설정값

```yml
management:
  # === 엔드포인트 활성화 ===
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus # 노출할 엔드포인트 (기본값: health,info)
        exclude: shutdown # 제외할 엔드포인트 (기본값: "")
      base-path: /actuator # 엔드포인트 기본 경로 (기본값: /actuator)
      path-mapping:
        health: health-check # 엔드포인트 경로 재매핑
    jmx:
      exposure:
        include: "*" # JMX 엔드포인트 노출

  # === Health 설정 ===
  endpoint:
    health:
      show-details: always # Health 상세 정보 (never, when-authorized, always, 기본값: when-authorized)
      show-components: always # 컴포넌트 상세 정보 표시 (기본값: when-authorized)
      probes:
        enabled: true # Readiness/Liveness 프로브 활성화 (기본값: false)

  health:
    # 각 헬스 체크 활성화
    defaults:
      enabled: true # 기본 헬스 체크 (기본값: true)
    diskspace:
      enabled: true # 디스크 공간 체크 (기본값: true)
      threshold: 10485760 # 헬스 체크 임계값 (바이트, 기본값: 10MB)
    db:
      enabled: true # DB 연결 체크 (기본값: true)
    livenessState:
      enabled: true # 실행 상태 프로브 (기본값: true)
    readinessState:
      enabled: true # 준비 상태 프로브 (기본값: true)

  # === Metrics 설정 ===
  metrics:
    export:
      prometheus:
        enabled: true # Prometheus 메트릭 내보내기 (기본값: true)
    tags:
      application: ${spring.application.name} # 메트릭 태그
      environment: ${spring.profiles.active} # 환경
    distribution:
      percentiles-histogram:
        http.server.requests: true # HTTP 히스토그램 (기본값: false)
    web:
      server:
        request:
          autotime:
            enabled: true # HTTP 요청 자동 타이밍 (기본값: true)
```

#### 주요 엔드포인트

| 엔드포인트 | 경로 | 설명 |
|-----------|------|------|
| health | `/actuator/health` | 애플리케이션 상태 |
| info | `/actuator/info` | 애플리케이션 정보 |
| metrics | `/actuator/metrics` | 메트릭 정보 |
| prometheus | `/actuator/prometheus` | Prometheus 형식 메트릭 |
| threads | `/actuator/threads` | 스레드 정보 |
| heapdump | `/actuator/heapdump` | 힙 덤프 |

#### Health 프로브 설정 (Kubernetes용)

```yaml
management:
  health:
    probes:
      enabled: true
  endpoint:
    health:
      probes:
        enabled: true

# Kubernetes 설정
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 20
  periodSeconds: 5

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

---

## spring.servlet (Servlet 설정)

서블릿 컨테이너 관련 설정입니다.

#### 주요 설정값

```yml
spring:
  servlet:
    multipart:
      enabled: true # 파일 업로드 활성화 (기본값: true)
      max-file-size: 10MB # 파일 최대 크기 (기본값: 1MB)
      max-request-size: 100MB # 요청 최대 크기 (기본값: 10MB)
      file-size-threshold: 0B # 디스크 저장 임계값 (기본값: 0B = 즉시 저장)
      location: /tmp/upload # 임시 파일 저장 위치 (기본값: 시스템 임시 디렉토리)
      resolve-lazily: false # 지연 참석 (기본값: false)
```

#### 상세 설명

| 설정 | 설명 | 기본값 | 권장값 |
|------|------|--------|--------|
| `max-file-size` | 파일 최대 크기 | 1MB | 10-100MB (용도에 따라) |
| `max-request-size` | 요청 최대 크기 | 10MB | 100-500MB (용도에 따라) |
| `location` | 임시 파일 저장 위치 | 시스템 임시 경로 | `/tmp/upload` 또는 별도 경로 |

---

## 프로파일별 권장 설정

### 개발 환경 (application-dev.yml)

```yml
logging:
  level:
    root: DEBUG
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG

spring:
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
  datasource:
    hikari:
      maximum-pool-size: 5

management:
  endpoints:
    web:
      exposure:
        include: "*"
```

### 테스트 환경 (application-test.yml)

```yml
logging:
  level:
    root: INFO

spring:
  jpa:
    hibernate:
      ddl-auto: create-drop
  datasource:
    hikari:
      maximum-pool-size: 3

management:
  endpoints:
    web:
      exposure:
        include: health,info
```

### 프로덕션 환경 (application-prod.yml)

```yml
logging:
  level:
    root: WARN
    org.springframework: WARN
  file:
    name: /var/log/app/application.log
    max-size: 100MB
    max-history: 30

spring:
  jpa:
    hibernate:
      ddl-auto: validate
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5

management:
  endpoints:
    web:
      exposure:
        include: health,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
```