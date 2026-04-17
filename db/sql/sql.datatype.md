# sql datatype


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

