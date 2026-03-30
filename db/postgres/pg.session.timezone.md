# Postgreql Session Timezone

PostgreSQL에서 timestamptz 값 생성을 생성하는 경우 (`now()` 등) 사용되는 timezone은 세션 타임존(Session TimeZone)의 설정이 사용된다.
세션 timezone은 **클라이언트와 서버 간의 여러 설정 경로**를 통해 지정할 수 있으며, **우선순위**에 따라 최종 적용되는 타임존이 결정됩니다.

**별도의 설정을 하지 않으면 `postgresql.conf`의 `timezone` 설정 값이 세션의 timezone 값으로 사용되며, 기본 값은 `UTC` 입니다.**
postgresql.conf의 timezone 설정은 initdb 실행 파일이 최초 생성하며(TZ 환경변수, /etc/localtime 참조) 이후 사용자가 직접 수정 할 수 있습니다.

## 우선순위 요약 (가장 높은 것이 우선 적용됨)

| 우선순위 | 설정 방법                       | 적용 시점   |
| ---- | ----------------------------------- | ----------- |
| 1    | `SET TIME ZONE ...`                 | postgres console 세션 내에서 실행 |
| 2    | JDBC URL: `options=-c TimeZone=...` | JDBC의 접속 시점에 연결 url에 명시적으로 지정 |
| 3    | 환경 변수 `PGTZ`                    | `클라이언트` 시작 시점에 참조되는 환경 변수 |
| 4    | `postgresql.conf`의 `timezone` 설정 | `서버` 시작 시 기본값 |
| 5    | OS의 `TZ` 환경 변수 설정 | `서버` 시작 시 기본값, initdb가 읽어서 `postgresql.conf`에 설정 |
| 6    | OS의 /etc/localtime | `서버` 시작 시 기본값, initdb가 읽어서 `postgresql.conf`에 설정 |

---

## 세션 타임존을 설정하는 모든 방법

- **세션 안에서 직접 설정**

  - `SET TIME ZONE 'Asia/Seoul';` / `SET timezone TO 'UTC';`
    같은 세션 내에서 즉시 효력. 이후에 또 `SET`하면 그 값이 덮어씁니다. ([PostgreSQL][1])
  - 트랜잭션 한정: `SET LOCAL TIME ZONE ...` (트랜잭션 종료 시 원복). ([PostgreSQL][1])

- **클라이언트가 접속 시 세션 기본값으로 주입**

  - `PGOPTIONS="-c TimeZone=UTC"`(libpq) 또는 JDBC URL에 `options=-c%20TimeZone=UTC` 같은 방식으로 전달 → 접속 직후 그 세션의 기본으로 설정. ([PostgreSQL][1])
  - `PGTZ=Asia/Seoul`(libpq 환경변수) → 접속 시 자동으로 `SET TIME ZONE`을 실행. ([PostgreSQL][2])
  - 애플리케이션/드라이버가 “접속 훅”에서 `SET TIME ZONE`을 실행(ORM 등) → 결국 세션 레벨 `SET`과 동일. ([PostgreSQL][1])

- **역할/데이터베이스 기본값으로 지정(새 세션 시작 시 적용)**

  - `ALTER ROLE ... IN DATABASE db SET timezone=...` (특정 역할+DB 조합) ([PostgreSQL][3])
  - `ALTER ROLE role SET timezone=...` (역할 전반) ([PostgreSQL][3])
  - `ALTER DATABASE db SET timezone=...` (DB 전반) ([PostgreSQL][4])

- **서버 전역 기본값**

  - 서버 시작 옵션: `postgres -c timezone=UTC` → 설정파일보다 우선. ([PostgreSQL][1])
  - `ALTER SYSTEM SET timezone=...` → `postgresql.auto.conf`에 기록, `postgresql.conf`보다 우선. ([PostgreSQL][5])
  - `postgresql.conf` 및 include된 설정들. ([PostgreSQL][1])

### 우선순위(높음 → 낮음)

1. **세션 내 최종 `SET` 값**(직접 실행, `PGOPTIONS`/드라이버가 보낸 `SET`, `PGTZ` 등 포함) — *가장 강력*. ([PostgreSQL][1])
2. **역할+DB 특정 기본값** `ALTER ROLE ... IN DATABASE` → **역할 기본**을 이기고, **DB 기본**도 이깁니다. ([PostgreSQL][3])
3. **역할 기본값** `ALTER ROLE ... SET` ([PostgreSQL][3])
4. **DB 기본값** `ALTER DATABASE ... SET` ([PostgreSQL][4])
5. **서버 시작 옵션** `postgres -c timezone=...` ([PostgreSQL][1])
6. **ALTER SYSTEM** (`postgresql.auto.conf`) ([PostgreSQL][1])
7. **postgresql.conf** (및 include 파일들) ([PostgreSQL][1])
8. **컴파일/초기 기본값**(아무것도 지정 안 했을 때의 기본) ([PostgreSQL][1])

> 참고: 문서에 **명시적으로** “충돌 시 어떤 것이 어떤 것을 이긴다”라고 써 있는 부분
> — *“DB-역할 특정 설정 > 역할 설정 > DB 설정”* 이 순서로 **우선**하며, 이들 설정은 **설정파일/서버 커맨드라인보다 우선**입니다. ([PostgreSQL][3])

### 확인용 쿼리

```sql
SHOW TIME ZONE;  -- 현재 세션에 적용 중인 TimeZone 값 확인
```

요약: 세션 타임존은 **세션에서 가장 마지막에 적용된 설정**이 이기며, 기본값 계층은 **(ROLE inDB) > 역할 > DB > 서버 전역** 순으로 내려갑니다. 설정파일보다 **서버 시작 옵션**이, `postgresql.conf`보다 **ALTER SYSTEM**이 우선합니다. ([PostgreSQL][3])

[1]: https://www.postgresql.org/docs/current/config-setting.html "PostgreSQL: Documentation: 18: 19.1. Setting Parameters"
[2]: https://www.postgresql.org/docs/current/libpq-envars.html "Documentation: 18: 32.15. Environment Variables"
[3]: https://www.postgresql.org/docs/current/sql-alterrole.html "PostgreSQL: Documentation: 18: ALTER ROLE"
[4]: https://www.postgresql.org/docs/current/sql-alterdatabase.html "Documentation: 18: ALTER DATABASE"
[5]: https://www.postgresql.org/docs/current/sql-altersystem.html "Documentation: 18: ALTER SYSTEM"

---

## PostgreSQL 세션 타임존 설정 예 (클라이언트 중심)

### 1. **SQL 명령어를 통해 명시적으로 설정** (가장 우선)

```sql
SET TIME ZONE 'Asia/Seoul';
-- 또는
SET TIME ZONE INTERVAL '+09:00' HOUR TO MINUTE;
```

- 세션 시작 후 명시적으로 설정하는 방식
- **이 설정이 존재하면, 가장 높은 우선순위로 적용**

---

### 2. **JDBC URL 파라미터로 전달**

```java
String url = "jdbc:postgresql://host:port/db?options=-c%20TimeZone=Asia/Seoul";
```

- `options=-c TimeZone=...`는 **PostgreSQL 세션 시작 시 타임존을 설정**
- SQL에서 `SHOW TimeZone;`으로 확인 가능

---

### 3. **환경 변수 `PGTZ` 설정 (클라이언트 환경)**

```bash
export PGTZ="Asia/Seoul"
```

```bash
PGTZ=Asia/Seoul psql -Upostgres -Atc "show time zone; select now();"
```

```bash
docker exec -it -e PGTZ=Asia/Seoul -e PGPASSWORD=pass my-postgres psql -p1111 -dmydb -Uuser -Atc "show time zone; select now();"
```

- PostgreSQL `클라이언트`(`psql`, JDBC 등) 실행 시 shell에서 환경변수로 설정
- `PGTZ`는 `클라이언트` 라이브러리(libpq)에 의해 사용됨

---

### 4. **PostgreSQL 서버 설정 (기본값)**

`postgresql.conf`에 설정:

```conf
timezone = 'UTC'
```

- `서버`의 기본 세션 타임존
- 위의 설정들이 없을 경우 사용됨

---

## 확인 방법

```sql
SHOW TimeZone;
-- 또는
SELECT current_setting('TimeZone');
```

---

## 팁

- `SET TIME ZONE`은 JDBC에서도 `Statement.execute("SET TIME ZONE 'Asia/Seoul'")`처럼 명시적으로 적용 가능
- Java의 `TimeZone.setDefault()`는 **JVM 내부 시간 처리용**이지 PostgreSQL 세션에는 영향 없음

---

## initdb 란

`initdb`는 **새 PostgreSQL 데이터베이스 클러스터(데이터 디렉터리)**를 만드는 초기화 도구입니다. 한 번 실행해서 저장소의 “뼈대”를 만들면, 그 위에서 서버를 시작하고 실제 DB를 생성·사용하게 됩니다.

### 무엇을 하냐?

- **데이터 디렉터리 생성** 및 권한 설정 (`-D`로 경로 지정)
- **시스템 카탈로그**와 **템플릿 DB**(`template0`, `template1`) 초기화
- `postgres` **슈퍼유저 계정** 생성(이름은 `-U`로 변경 가능)
- **로케일/인코딩** 설정 (`--locale`, `-E`)
- **기본 타임존**을 환경/OS에서 감지해 `postgresql.conf`에 기록
- `postgresql.conf`, `pg_hba.conf`, `pg_ident.conf` 등 **기본 설정 파일** 생성
- **WAL** 디렉터리/구조 준비(옵션에 따라 `--waldir`, 체크섬 `--data-checksums` 등)

### 무엇을 하지 않나?

- **서버를 시작하진 않습니다.** 초기화 후에는 `pg_ctl start` 또는 `postgres -D ...`로 서버를 띄워야 합니다.
- 이미 존재하는 데이터 디렉터리를 덮어쓰지 않습니다(빈 디렉터리 필요).

### 자주 쓰는 예시

```bash
# 새 클러스터 초기화
initdb -D /var/lib/postgresql/data \
       -U myadmin \
       -E UTF8 \
       --locale=en_US.UTF-8 \
       --data-checksums

# 서버 시작
pg_ctl -D /var/lib/postgresql/data -l logfile start

# (확인) 현재 세션 타임존
psql -c "SHOW TIME ZONE;"
```

### initdb에 타임존 정보를 지정하지 않으면?

initdb에 타임존을 따로 지정하지 않으면, postgresql.conf의 TimeZone은 “initdb가 실행된 시스템 환경의 타임존”으로 기록됩니다.
즉, 환경변수 TZ가 있으면 그 값, 없으면 OS의 로컬 타임존을 반영합니다. 이는 “기본(내장) 값은 GMT지만, 보통 postgresql.conf에서 덮어쓴다. initdb가 시스템 환경에 맞는 값을 거기에 써 넣는다”라고 문서에 명시되어 있습니다.

### TIP

- **컨테이너/클라우드**: 최초 부팅 스크립트에서 `initdb`가 자동 실행되는 이미지가 많습니다(빈 볼륨일 때만).
- **타임존/로케일**을 초기값으로 깔끔히 고정하고 싶다면, `initdb` 실행 시점에 환경변수(`TZ`, `LANG` 등)나 옵션을 명시하세요.
- 한 머신(인스턴스)에서 **여러 클러스터**도 가능하지만, 각 클러스터는 **별도 데이터 디렉터리 + 포트**를 써야 합니다.

---

## 주의: JVM의 타임존 설정은 postgresql과의 세션 타임존에 아무런 영향을 끼치지 않는다

JVM의 타임존 설정을 읽고 JDBC가 Postgres와의 세션 연결에 사용하지 않을까하는 의문이 들 수 있다.
짧게 말하면 JDBC가 타임존 값 필요시(timestamptz 변환 등) JVM의 설정을 사용하는 것은 맞지만 JDBC가 Postgres 세션 생성에 JVM의 타임존 정보는 사용하지 않으며, `JDBC는 세션 연결에 어떤 타임존 정보도 자동으로 세션 연결에 명시하지 않는다.`
JDBC의 세션 연결에 타임존을 별도로 지정하고 싶다면 `사용자가 반드시 연결 url에 옵션으로 명시`하여야 한다.

- **PostgreSQL “세션 타임존”은 서버가 정합니다.**
  접속할 때 아무 조치를 하지 않으면, 세션의 `TimeZone` 파라미터는 **서버/DB/사용자 기본값**(예: `postgresql.conf`의 `timezone`)을 따릅니다. 이 값은 `SHOW TIME ZONE;`으로 확인할 수 있어요. ([PostgreSQL][11])

- **클라이언트(JDBC)의 JVM 타임존은 자동으로 서버 세션 타임존을 바꾸지 않습니다.**
  세션 타임존은 클라이언트가 명시적으로 바꿀 때만 변합니다. 바꾸는 방법은 예를 들면:

  - SQL로: `SET TIME ZONE 'Asia/Seoul';` ([PostgreSQL][12])
  - 접속 옵션으로: JDBC URL에 `options=-c%20TimeZone=Asia/Seoul` 등 **서버 GUC 설정 전달** ([PostgreSQL][12])
  - (libpq 계열 전용) 환경변수 `PGTZ`를 통해 접속 시 자동 `SET TIME ZONE` 실행 — **pgJDBC는 libpq가 아니므로 보통 해당 없음**. ([PostgreSQL][11])

- **따라서**

  > JVM 타임존 = `Asia/Seoul`, `postgresql.conf` = `timezone = 'Etc/UTC'`
  > → 애플리케이션/드라이버가 별도로 `SET TIME ZONE`(또는 옵션 전달)을 **하지 않았다면**, 각 세션의 `TimeZone`은 **UTC**입니다. `SELECT now();` 같은 `timestamptz`의 “표시”는 세션 타임존(=UTC)을 따릅니다. ([PostgreSQL][11])

- **혼동 포인트 (왜 서울시간처럼 보일 때가 있나?)**
  JDBC가 `timestamptz`를 가져와 **자바 타입**(예: `Timestamp`, `OffsetDateTime`)으로 변환한 뒤 **문자열로 포맷**할 때는 JVM 기본 타임존(여기선 `Asia/Seoul`)이 적용되어 **출력만 서울시간으로 보일 수 있습니다.** 값 자체(instant)는 동일합니다. 이 동작 때문에 “세션은 UTC인데 앱에서 보면 +09로 보임” 같은 현상이 생깁니다. 세션 타임존은 그대로고, **표현 층**에서 바뀌는 거예요. ([GitHub][13])

**결론:** 기본적으로 **JVM 타임존 ≠ DB 세션 타임존**입니다. 별도 지시가 없다면, 질문의 구성에선 **세션 타임존은 UTC**이고, 자바 쪽 출력만 서울시간처럼 보일 수 있어요.

### 실전 권장 세팅

1. **DB 세션 타임존을 고정(보통 UTC)**

- DB/사용자/DB별 기본값:

  ```sql
  ALTER DATABASE mydb SET TIME ZONE 'UTC';
  -- 또는 ALTER ROLE myuser SET TIME ZONE 'UTC';
  ```

- 접속 시 강제: JDBC URL 예)
  `jdbc:postgresql://.../mydb?options=-c%20TimeZone=UTC` ([PostgreSQL][12])

1. **애플리케이션에서 일관성 유지**

- 자바 8+라면 `Instant`/`OffsetDateTime` 사용, 포맷 시 `ZoneOffset.UTC`를 명시.
- ORM을 쓴다면 Hibernate의 `hibernate.jdbc.time_zone=UTC` 같은 설정으로 JDBC 변환 타임존을 고정. ([In Relation To][14])

1. **현재 세션 상태 확인**

```sql
SHOW TIME ZONE;      -- 세션 타임존 확인
SELECT now();        -- 세션 타임존 기준 표시되는 timestamptz
```

[11]: https://www.postgresql.org/docs/current/datatype-datetime.html "Documentation: 17: 8.5. Date/Time Types - PostgreSQL"
[12]: https://www.postgresql.org/docs/current/sql-set.html "Documentation: 17: SET - PostgreSQL"
[13]: https://github.com/pgjdbc/pgjdbc/discussions/3281 "Time and timezone issues in the driver ..."
[14]: https://in.relation.to/2016/09/12/jdbc-time-zone-configuration-property "How to store timestamps in UTC using the new hibernate.jdbc ..."
