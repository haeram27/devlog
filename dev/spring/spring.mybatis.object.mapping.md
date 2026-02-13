
# Mybatis: RDB table - JAVA Object 매핑

- Mybatis는 column의 이름을 그대로 java object의 member 이름과 매칭한다.
  - RDB에서는 전통적으로 column의 이름에 lower_snake_case 네이밍 컨벤션을 사용한다.
  - Java에서는 Object 필드의 이름에 lowewrCamelCase 네이밍 컨벤션을 사용한다.
  - 두 네이밍 컨벤션을 자동으로 맵핑하려면 Mybatis의 `map-underscore-to-camel-case` 설정을 사용해야한다.
  - Mybatis의 `map-underscore-to-camel-case=true` 설정을 하면, RDB의 lower_snake_case 이름을 자동으로 JAVA Object의 lowewrCamelCase와 맵핑해준다.
- Mybatis용 Java Object에는 별도의 Jackson annotation(JsonNaming, JsonProperty)이 필요하지 않다.
  - Mybatis는 자체적으로 RDB table 데이터와 Java Object를 맵핑하는 로직을 갖으며, 이 과정에 Jackson을 사용하지 않는다.
  - JsonNaming, JsonProperty 어노테이션은 Jackson 라이브러리에서 Java Object를 Json 문서로 직렬화 또는 역직렬화 하는 과정에서 네이밍 컨벤션을 매칭하기 위하여 사용된다.

## Column Naming Convention 맵핑

### mapUnderscoreToCamel

Mybatis에서 RDB column 이름의 lower_snake_case 네이밍 컨벤션을 Java class field의 lowewrCamelCase 네이밍 컨벤션으로 자동 매칭해 주는 설정

mybatis-config.xml

```xml
<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

application.properties(spring boot)

```text
mybatis.configuration.map-underscore-to-camel-case=true
```

application.yml(spring boot)

```yml
mybatis:
  configuration:
    map-underscore-to-camel-case: true
    use-generated-keys: true
  mapper-locations: classpath*:/mapper/**/*.xml
```

@Configuration Bean Java 파일에서 설정 (spring boot)

```java
    @Bean(name = "oltpSqlSessionFactory")
    @Primary
    public SqlSessionFactory oltpSqlSessionFactory(
            @Qualifier("oltpDataSource") DataSource oltpDataSource,
            ApplicationContext applicationContext) {
        try {
            SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
            sqlSessionFactoryBean.setDataSource(oltpDataSource);
            org.apache.ibatis.session.Configuration configuration = new org.apache.ibatis.session.Configuration();
            configuration.setUseGeneratedKeys(true);
            configuration.setMapUnderscoreToCamelCase(true);  // map-underscore-to-camel-case  설정
            sqlSessionFactoryBean.setConfiguration(configuration);
            sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources("classpath*:/mapper/**/*.xml"));
            return sqlSessionFactoryBean.getObject();
        } catch (Exception e) {
            throw new BeanCreationException("cannot create bean.", e);
        }
    }
```

#### `map-underscore-to-camel-case=true` 사용시 Java Object 구현 예제

```java
import lombok.Data;

import java.io.Serializable;
import java.time.LocalDateTime;

@Data
public class SelectResultObject implements Serializable {

    private static final long serialVersionUID = 1L;

    // table column name: snake
    // id, name, description, is_student, modification_time

    private Long id;                     // column: id, type: bigsearial, PRIMARY KEY
    private String name;                 // column: name, type: varchar
    private String description;          // column: description, type: varchar
    private Boolean isStudent;           // column: is_student, type: boolean
    private LocalDateTime modifiedTime;  // column: modified_time, type: timestamp 
}
```

## Type Mapping

### 데이터 흐름 구조

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

### 데이터 전달 간 데이터 타입 호환

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
| **`timestamp with time zone` (`timestamptz`)** | **`TIMESTAMP_WITH_TIMEZONE`** | **`java.time.Instant`**(추천/가장 안전) / `java.time.OffsetDateTime` <br> *(대안: `java.sql.Timestamp`)* |
| `interval` | `OTHER` | `java.time.Duration`/`Period` *(커스텀 TypeHandler 권장; PG JDBC는 `PGInterval`)* |
| `json` | `VARCHAR` / `LONGVARCHAR` / `OTHER` | `String` *(또는 DTO, `JsonNode` with TypeHandler)* |
| `jsonb` | `OTHER` | `String` / DTO *(TypeHandler)* |
| `xml` | `LONGVARCHAR` / `SQLXML` | `String` / `javax.sql.rowset.serial.SerialBlob` 등 |
| `inet` / `cidr` / `macaddr` | `OTHER` | `String` *(또는 커스텀 타입 + TypeHandler)* |
| 배열 예: `int[]`, `text[]` | `ARRAY` | `java.sql.Array` *(또는 `List<T>` with TypeHandler)* |
| `enum` (domain/enum type) | `VARCHAR` / `OTHER` | `String` / Java `enum` *(TypeHandler로 매핑)* |

