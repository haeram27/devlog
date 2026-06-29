# Postgresql Data Type

## SQL type vs JAVA type Mapping

| PostgreSQL Data Type       | Java Data Type                  |
|----------------------------|---------------------------------|
| BOOLEAN (bool)             | java.lang.Boolean               |
| INTEGER (int, int4)        | java.lang.Integer               |
| BIGINT (int8)              | java.lang.Long                  |
| SMALLINT (int2)            | java.lang.Short                 |
| NUMERIC (numeric, decimal) | java.math.BigDecimal            |
| SERIAL                     | java.lang.Integer               |
| SMALLSERIAL                | java.lang.Short                 |
| REAL (float4)              | java.lang.Float                 |
| DOUBLE PRECISION (float8)  | java.lang.Double                |
| VARCHAR (varchar),         | java.lang.String                |
| CHAR (bpchar),             | java.lang.String                |
| TEXT (text)                | java.lang.String                |
| DATE                       | java.time.LocalDate             |
| TIME                       | java.time.LocalTime             |
| TIME \w timezone           | java.time.OffsetTime            |
| TIMESTAMP                  | java.time.LocalDateTime         |
| TIMESTAMP \w timezone      | java.time.OffsetDateTime        |
|                            | java.time.ZonedDateTime         |
| BYTEA                      | byte[]                          |
| ARRAY                      | java.sql.Array                  |
| UUID                       | java.util.UUID                  |
| JSON                       | java.lang.String                |
| JSONB                      | or json object of Jackson/Gson  |
| HSTORE                     | java.util.Map<String, String>   |

## [character](https://www.postgresql.org/docs/15/datatype-character.html)

| Name                             | Description                |
|----------------------------------|----------------------------|
| character varying(n), varchar(n) | variable-length with limit |
| character(n), char(n)            | fixed-length, blank padded |
| text                             | variable unlimited length  |

- varchar    == text
- varchar(n) == character varying(n)

varchar의 경우 `n` 없이 명시 할 수 있으며, 이 경우 text를 사용한 것과 같다. (unlimited string, 물리적으로 10485760 Byte(약  1GB)로 제약 되긴 한다).
varchar(without n)과 text 사용에 대한 performance는 같다.

## [numeric](https://www.postgresql.org/docs/15/datatype-numeric.html)

| Name             | Storage Size | Description                     | Range |
|------------------|--------------|---------------------------------|---|
| smallint         | 2 bytes      | small-range integer             | -32768 to +32767 |
| integer          | 4 bytes      | typical choice for integer      | -2147483648 to +2147483647 |
| bigint           | 8 bytes      | large-range integer             | -9223372036854775808 to +9223372036854775807 |
| decimal          | variable     | user-specified precision, exact | up to 131072 digits before the decimal point; up to 16383 digits after the decimal point |
| numeric          | variable     | user-specified precision, exact | up to 131072 digits before the decimal point; up to 16383 digits after the decimal point |
| real             | 4 bytes      | variable-precision, inexact     | 6 decimal digits precision |
| double precision | 8 bytes      | variable-precision, inexact     | 15 decimal digits precision |
| smallserial      | 2 bytes      | small autoincrementing integer  | 1 to 32767 |
| serial           | 4 bytes      | autoincrementing integer        | 1 to 2147483647 |
| bigserial        | 8 bytes      | large autoincrementing integer  | 1 to 9223372036854775807 |

## [datetime](https://www.postgresql.org/docs/15/datatype-datetime.html)

| Name | Storage Size | Description | Low Value | High Value | Resolution |
|---|---|---|---|---|---|
| timestamp [ (p) ] [ without time zone ] | 8 bytes | both date and time (no time zone) | 4713 BC | 294276 AD | 1 microsecond |
| timestamp [ (p) ] with time zone | 8 bytes | both date and time, with time zone | 4713 BC | 294276 AD | 1 microsecond |
| date | 4 bytes      | date (no time of day) | 4713 BC | 5874897 AD | 1 day |
| time [ (p) ] [ without time zone ] | 8 bytes | time of day (no date) | 00:00:00 | 24:00:00 | 1 microsecond |
| time [ (p) ] with time zone | 12 bytes     | time of day (no date), with time zone | 00:00:00+1559 | 24:00:00-1559   | 1 microsecond |
| interval [ fields ] [ (p) ] | 16 bytes | time interval | -178000000 years | 178000000 years | 1 microsecond |
