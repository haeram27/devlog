
# timestamp with timezone 저장 및 전송 방법

- **DB 저장**: PostgreSQL `timestamptz` 컬럼 ↔ Java `Instant` 매핑이 이상적
- **REST/JSON 전송**: Java `Instant` → Json `ISO-8601 UTC 문자열` (`2025-09-04T01:15:30Z`) 권장
- **비즈니스 로직에서 지역 시간 표시**: `ZonedDateTime`이나 `OffsetDateTime`을 사용 후, 저장/전송 시 `Instant`로 변환

| Postgresql (DB, 저장)  | Java   | Json (통신, 전송) |
| ----------- | ------------- | ------------------- |
| timestamp   | LocalDateTime | ISO-8601 UTC 문자열 |
| timestamptz | Instant       | ISO-8601 UTC 문자열 |

---

## Java 시간 타입 비교

| Java 타입 | 특징 | 사용처 |
| --- | --- | --- |
| `Instant`        | UTC 기준 타임스탬프 | **전송/저장용 표준, 절대 시각 표현**, nanosec 단위, epoch time을 0으로 하는 timepoint 값 |
| `OffsetDateTime` | UTC 오프셋 포함 (예: `+09:00`) | 사용자 출력 포맷, LocalDateTime + UTC 기준 offset 시간 표현, DST 규칙 미반영 |
| `ZonedDateTime`  | 지역 타임존 포함 (예: `Asia/Seoul`) | 사용자 출력 포맷, LocalDateTime + timezone ID 표현, DST(Daylight Saving Time) 규칙 반영 |
| `LocalDateTime`  | timezone 정보 없음 | 로컬 비즈니스 로직 (예: 매장 오픈시간 "09:00") |

---

## DB 저장 시 (특히 PostgreSQL)

- PostgreSQL `timestamptz` 컬럼 ←→ Java `Instant` 매핑이 가장 안전
  → JDBC 42.2+부터 `ResultSet.getObject(..., Instant.class)` 지원
- 이유:

  - `timestamptz`는 내부적으로 UTC로 저장 → `Instant`와 딱 맞음
  - `LocalDateTime`은 타임존을 잃어버려서 혼란 발생 가능
  - `OffsetDateTime`도 괜찮지만, 결국 서버/클라이언트 타임존 이슈가 섞일 수 있음

---

## 네트워크 전송 시 (예: REST/JSON)

- JSON/REST API 교환 포맷에서 **ISO-8601 UTC 문자열** (`2025-09-04T01:15:30Z`)을 쓰는 게 표준적
- Jackson 같은 라이브러리는 `Instant`를 자동으로 **UTC ISO-8601 문자열**로 직렬화/역직렬화
- 따라서 API/네트워크 전송에도 `Instant`가 가장 호환성이 좋음

---

## JAVA Instant -> ISO-8601 UTC 문자열 변환

```java
DateTimeFormatter.ISO_INSTANT.format(Instant.now());
```
