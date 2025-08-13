# Postgreql Session Timezone

PostgreSQL에서 세션 타임존(Session TimeZone)은 **클라이언트와 서버 간의 여러 설정 경로**를 통해 지정할 수 있으며, **우선순위**에 따라 최종 적용되는 타임존이 결정됩니다.

**별도의 설정을 하지 않으면 `postgresql.conf`의 `timezone` 설정 값이 사용되며, 기본 값은 `UTC` 입니다.**

## 우선순위 요약 (가장 높은 것이 우선 적용됨)

| 우선순위 | 설정 방법                       | 적용 시점   |
| ---- | ----------------------------------- | ----------- |
| 1    | `SET TIME ZONE ...`                 | 세션 내에서 실행      |
| 2    | JDBC URL: `options=-c TimeZone=...` | 접속 시점             |
| 3    | 환경 변수 `PGTZ`                    | `클라이언트` 시작 시  |
| 4    | `postgresql.conf`의 `timezone` 설정 | `서버` 시작 시 기본값 |

---

## PostgreSQL 세션 타임존 설정 방법 (클라이언트 중심)

### 1. **SQL 명령어를 통해 명시적으로 설정** (가장 우선)

```sql
SET TIME ZONE 'Asia/Seoul';
-- 또는
SET TIME ZONE INTERVAL '+09:00' HOUR TO MINUTE;
```

* 세션 시작 후 명시적으로 설정하는 방식
* **이 설정이 존재하면, 가장 높은 우선순위로 적용**

---

### 2. **JDBC URL 파라미터로 전달**

```java
String url = "jdbc:postgresql://host:port/db?options=-c%20TimeZone=Asia/Seoul";
```

* `options=-c TimeZone=...`는 **PostgreSQL 세션 시작 시 타임존을 설정**
* SQL에서 `SHOW TimeZone;`으로 확인 가능

---

### 3. **환경 변수 `PGTZ` 설정 (클라이언트 환경)**

```bash
export PGTZ="Asia/Seoul"
```

* PostgreSQL `클라이언트`(`psql`, JDBC 등) 실행 시 shell에서 환경변수로 설정
* `PGTZ`는 `클라이언트` 라이브러리(libpq)에 의해 사용됨

---

### 4. **PostgreSQL 서버 설정 (기본값)**

`postgresql.conf`에 설정:

```conf
timezone = 'UTC'
```

* `서버`의 기본 세션 타임존
* 위의 설정들이 없을 경우 사용됨

---

## 확인 방법

```sql
SHOW TimeZone;
-- 또는
SELECT current_setting('TimeZone');
```

---

## 팁

* `SET TIME ZONE`은 JDBC에서도 `Statement.execute("SET TIME ZONE 'Asia/Seoul'")`처럼 명시적으로 적용 가능
* Java의 `TimeZone.setDefault()`는 **JVM 내부 시간 처리용**이지 PostgreSQL 세션에는 영향 없음

---
