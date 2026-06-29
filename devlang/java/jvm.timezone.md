# JVM 기본 타임존은?

- **기본값:** JVM 시작 시점의 **운영체제(OS) 타임존**을 읽어 **프로세스 전체의 기본 타임존**으로 설정합니다.

  - 확인: `ZoneId.systemDefault()` 또는 `TimeZone.getDefault()`.

## 주의

JVM의 타임존은 java 어플리케이션내에서 timezone이 필요할 때 사용되나 DB 세션등에 영향을 끼치지 않는다.
DB 세션 timezone은 DB 연결시(JDBC의 DB 연결 명령에 옵션으로 추가 하거나 세션 실행 후 SQL 명령 실행) 별도로 설정하지 않으면, 기본적으로 DB의 설정에 명시된 timezone 값이 사용된다.

## 무엇이 기본값을 좌우하나?

우선순위(높음 → 낮음)

1. **JVM 옵션 `-Duser.timezone=...`** (가장 우선)
2. **환경변수 `TZ`** (Unix/Linux·컨테이너 환경에서 자주 사용)
3. **OS 설정**
   - linux의 경우 /etc/localtime (내용은 timedatectl 또는 readlink 명령으로 확인 또는 설정)
   - windows의 경우 지역 설정

> 참고: 한 번 **JVM이 뜬 뒤 OS 타임존을 바꿔도** 대부분의 플랫폼에서 **자동 반영되지 않습니다.** (JVM 시작 시 읽어 캐시)

## 변경 방법 (상황별)

- **애플리케이션/프로세스 단위(권장):**

  - JVM 옵션: `java -Duser.timezone=Asia/Seoul -jar app.jar`
  - 컨테이너: `docker run -e TZ=Asia/Seoul ...` (또는 /etc/localtime 마운트)
- **런타임에서(전역, 주의 필요):**

  ```java
  TimeZone.setDefault(TimeZone.getTimeZone("Asia/Seoul")); // 프로세스 전체에 영향
  ```

  - 서버/멀티테넌트 환경에서는 전역 변경이라 **권장되지 않음**.
- **OS 수준:** OS의 시간대 설정을 변경(새로 뜨는 JVM 프로세스에 적용).

## 사용 시 유의사항

- `Instant`나 `System.currentTimeMillis()` 같은 **절대시간**은 타임존과 무관.
- `LocalDateTime.now()`나 `ZonedDateTime.now()`처럼 **“now”를 구할 때** 기본 타임존이 적용됨.
- **스레드별 기본 타임존은 없다.** 필요한 곳에서 항상 `ZoneId`를 명시하는 것이 안전.
- JVM timezone에 영향 받는 클래스와 메소드
  - LocalDateTime.now()
  - OffsetDateTime.now()
  - ZonedDateTime.now()
  - SimpleDateForamt.format()

## 빠른 체크 코드

```java
ZoneId jdkZone = ZoneId.systemDefault();
TimeZone tz = TimeZone.getDefault();
System.out.println("ZoneId.systemDefault() = " + jdkZone);
System.out.println("TimeZone.getDefault()  = " + tz.getID());
```

요약: JVM의 기본 타임존은 **프로세스 시작 시 OS(또는 TZ/env, -Duser.timezone)에서 결정**되고, 이후에는 **전역 변경**(`TimeZone.setDefault`)만 반영됩니다. 안전하게 하려면 **필요한 곳에서 명시적으로 `ZoneId`를 사용**하세요.
