
# JAVA DB 데이터 매핑

## 데이터 흐름 구조

```java
Java Application
      |
   MyBatis
      |
 JDBC API (java.sql.*)
      |
PostgreSQL JDBC Driver (postgresql-<ver>.jar)
      |
 PostgreSQL 서버
```

##

| PostgreSQL | JDBC 4.2.x+ (`jdbcType`) | MyBatis (Java)  (권장 / 대안) |
| --- | --- | --- |
| `boolean` | `BOOLEAN` | `Boolean` / `boolean`|
| `smallint` | `SMALLINT` | `Short` / `short` |
| `integer` | `INTEGER` | `Integer` / `int` |
| `bigint` | `BIGINT` | `Long` / `long` |
| `serial` | `INTEGER` | `Integer` / `int` |
| `bigserial` | `BIGINT` | `Long` / `long` |
| `numeric(p,s)` / `decimal(p,s)` | `DECIMAL` / `NUMERIC` | `java.math.BigDecimal` |
| `real` (float4) | `REAL` | `Float` / `float` |
| `double precision` (float8) | `DOUBLE` | `Double` / `double` |
| `money` | `DECIMAL` | `BigDecimal` / `String` |
| `char(n)` | `CHAR` | `String` |
| `varchar(n)` | `VARCHAR` | `String` |
| `text` | `LONGVARCHAR` (또는 `VARCHAR`) | `String` |
| `bytea` | `BINARY` / `VARBINARY` | `byte[]` |
| `uuid` | `OTHER` | `java.util.UUID` |
| `date` | `DATE` | `java.time.LocalDate` *(대안: `java.sql.Date`)* |
| `time without time zone` | `TIME` | `java.time.LocalTime` *(대안: `java.sql.Time`)* |
| `time with time zone` (`timetz`) | `TIME_WITH_TIMEZONE` | `java.time.OffsetTime` *(드라이버/버전별 주의)* |
| `timestamp without time zone` (`timestamp`)    | `TIMESTAMP` | `java.time.LocalDateTime` *(대안: `java.sql.Timestamp`)* |
| **`timestamp with time zone` (`timestamptz`)** | **`TIMESTAMP_WITH_TIMEZONE`** | **`java.time.Instant`**(추천/가장안전)/`java.time.OffsetDateTime` <br> *(대안: `java.sql.Timestamp`)* |
| `interval` | `OTHER` | `java.time.Duration`/`Period` *(커스텀 TypeHandler 권장; PG JDBC는 `PGInterval`)* |
| `json` | `VARCHAR` / `LONGVARCHAR` / `OTHER` | `String` *(또는 DTO, `JsonNode` with TypeHandler)* |
| `jsonb` | `OTHER` | `String` / DTO *(TypeHandler)* |
| `xml` | `LONGVARCHAR` / `SQLXML` | `String` / `javax.sql.rowset.serial.SerialBlob` 등 |
| `inet` / `cidr` / `macaddr` | `OTHER` | `String` *(또는 커스텀 타입 + TypeHandler)* |
| 배열 예: `int[]`, `text[]` | `ARRAY` | `java.sql.Array` *(또는 `List<T>` with TypeHandler)* |
| `enum` (domain/enum type) | `VARCHAR` / `OTHER` | `String` / Java `enum` *(TypeHandler로 매핑)* |
