# JDBC의 timestamptz 읽기

JDBC가 RDB 로 부터 /timstamptz(tstz)를 읽을 때, 세션의 timezone 설정을 기준으로 시간 값이 자동 변환 된다.
JDBC 연결시 별도의 timezone 설정이 없으면, 세션 timezone은 RDB의 기본 설정(설정 파일)을 따르게 되고 기본값은 `UTC`이다.

## RDB -> JDBC 과정에서 timestamp와 timestamptz의 세션 timezone 기반 자동변환 여부

timestamptz는 세션 tiemzone 기준으로 시간 값이 자동 변환되고 <br>
timestamp는 자동 변환 되지 않지 않고 값 그대로 전달(시간 숫자 기준) 된다.
변환 주체는 JDBC이다.

**Table.** JDBC(4.2+) + java(8+) 조합 환경에서 timestamp, timestamptz 시간 값의 세션 timezone 기준 자동 변환

|RDB 타입|JAVA 맵핑 타입|시간값 자동변환 여부|
|---|---|---|
|timestamp|java.time.LocalDateTime|`X`|
|timestamptz|java.time.OffsetDateTime<br>java.time.ZonedDateTime|`O`|

**Table.** JDBC(4.1-) + java(7-) 조합 환경에서 timestamp 시간 값의 세션 timezone 기준 자동 변환

|RDB 타입|JAVA 맵핑 타입|시간값 자동변환 여부|
|---|---|---|
|timestamp|java.sql.Timestamp|`O`|

## JDBC가 RDB의 timestamptz를 timestamp로 변환 맵핑하는 과정 (Java 8+, JDBC 4.2+)

JDBC 4.2 이상에서는 timestamptz 컬럼을 OffsetDateTime 또는 ZonedDateTime으로 가져올 경우,
JDBC는 더 이상 java.sql.Timestamp로 변환하지 않고, 직접적으로 `java.time.OffsetDateTime 또는 ZonedDateTime 객체`로 변환하여 반환합니다.

즉, 명시적으로 getObject(..., OffsetDateTime.class) 또는 getObject(..., ZonedDateTime.class)를 사용할 경우, JDBC는 java.sql.Timestamp를 거치지 않습니다.

* Java 8+ 사용 중이라면, `java.sql.Timestamp`는 사용을 **지양**하고 `OffsetDateTime` 사용을 추천드립니다.
* **ORM (예: JPA)** 에서도 `@Column`에 `OffsetDateTime`을 매핑하면 `timestamptz`와 자동 대응됩니다.
* 단 JDBC는 RDB의 timestamptz 값을 현재 세션의 timezone으로 변환하여 `OffsetDateTime`나 `ZonedDateTime` 형식에 맵핑하므로, JDBC가 읽어오는 timestamptz의 값이 임의로 변경되길 원하지 않는다면, JDBC 세션 설정시 timezone 값은 되도록 `UTC`로 사용하도록 설정한다. 별도로 설정하지 않으면 JDBC와 RDB 사이의의 세션 timezone은 RDB의 기본값을 사용하며 이는 `UTC`이다.

## 예제: JAVA에서 JDBC를 통해 `OffsetDateTime`으로 RDB의의 `timestamptz` 받기

### PostgreSQL 테이블

```sql
CREATE TABLE example (
    id SERIAL,
    ts timestamptz
);
INSERT INTO example (ts) VALUES ('2025-05-01 12:00:00+09');
```

---

### Java 코드 (JDBC 4.2+)

```java
ResultSet rs = stmt.executeQuery("SELECT ts FROM example");
if (rs.next()) {
    OffsetDateTime odt = rs.getObject("ts", OffsetDateTime.class);
    System.out.println(odt);  // 2025-05-01T12:00+09:00
}
```

* 출력 결과는 정확한 **오프셋(+09:00)** 정보 포함
* `java.sql.Timestamp`가 아닌 java.time.OffsetDateTime 이나 ZonedDateTime 사용

## JDBC 동작 설명

| 호출 방식                                      | 결과 타입                      | 포함 정보      |
| ------------------------------------------ | -------------------------- | ---------- |
| `rs.getTimestamp("ts")`                    | `java.sql.Timestamp`       | **타임존 없음** |
| `rs.getObject("tstz", OffsetDateTime.class)` | `java.time.OffsetDateTime` | **오프셋 포함** |
| `rs.getObject("tstz", ZonedDateTime.class)`  | `java.time.ZonedDateTime`  | **시간대 포함** |

* PostgreSQL JDBC(4.2+) 드라이버가 `timestamptz` → `OffsetDateTime` 매핑을 직접 수행
* 단 JDBC는 RDB의 timestamptz 값을 현재 세션의 timezone으로 변환하여 `OffsetDateTime`나 `ZonedDateTime` 형식에 담아준다. 다시말해 `별도로 JDBC 세션 timezone을 설정`한다면 `RDB로 부터 읽어온 timestamptz는 세션의 timezone으로 변환된 값`이 된다.

---

## JDBC/RDB간 올바른 timestamptz 사용 전략

* RDB에서는 항상 UTC 기준으로 timestamp 또는 timestamptz 값을 저장한다.
* RDB의 timezone 설정은 `UTC`로 사용한다.
* JDBC의 세션 설정시 timezone은 `UTC`를 사용한다. (별도의 설정을 하지 않는다)
* JAVA 8+ 이상에서 RDB의 timestamptz를 읽는 경우 항상 `java.time.OffsetDateTime 또는 ZonedDateTime 객체`를 이용하여 읽어온다.

---

## 참고: `JDBC 4.2 미만 + JAVA 8 미만` 환경경에서 RDB의 timestamptz를 timestamp로 변환 맵핑하는 과정

* JDBC 4.2 미만 버전에서서는 RDB의 timestamptz(timestamp with timezone) 타입을 맵핑할 데이터 타입이 없다.
* RDB의 timestamptz는 항상 UTC 기준 timezonme + 시간 값으로 저장된다.
* JDBC는 RDB의 timestamptz 형식을 java.sql.Timestamp 타입으로 변환 맵핑하여 가져온다.
* java.sql.Timestamp 는 timezone 정보가 없는 timestamp 저장 형식이다.
* JDBC가 timestamptz 를 Timestamp로 변환할 때는 `JDBC와 RDB 사이에 맺어진 현재 세션의 timezone 설정`을 기준으로 timestamp를 계산하여 가져온다. 결과적으로 JDBC를 통해 가져온 timestamptz는 세션의 timezone으로 변환 연산된 timestamp 값이다.
* 현재 세션의 timezone은 별도의 사용자 설정이 없는 경우 RDB 설정 파일의 기본 값을 사용하게 되는데, 보통 `UTC`이다.
* RDB의 기본 timezone은 각 RDB 어플리케이션의 공식 문서 또는 설정을 확인한다.
* JDBC를 통해서 읽어온 timestamp 값은 클라이언트 JVM 어플리케이션에서 필요한 timezone으로 변화하여 사용하여야 한다.

---

## 참고: JDB(클라이언트 드라이버) <-> RDB(서버) 사이의 세션 타임존 설정 우선순위

JDBC 연결시 별도의 명시적적 설정이 없으면, 서버 기본 설정이 적용됨

| 우선순위  | 설정 위치                        | 설명                                       |
| --------- | -------------------------------- | ------------------------------------------ |
| ① (가장 우선) | **세션 내 명시적 설정**                  | `SET TIME ZONE 'Asia/Seoul';` |
| ②         | **JDBC URL 파라미터**                        | `?options=-c%20TimeZone=Asia/Seoul` |
| ③ (가장 낮음) | **서버 기본 설정** (`postgresql.conf`)   | `timezone = 'UTC'` 등, 명시 설정이 없을 때만 사용됨 |

---

## 참고: JDBC와 RDB의 세션 timezone 확인하기

Postgresql 기준:

```java
import java.sql.*;

public class JdbcTimezoneCheck {
    public static void main(String[] args) {
        String url = "jdbc:postgresql://localhost:5432/mydb?options=-c%20TimeZone=Asia/Seoul";
        String user = "myuser";
        String password = "mypassword";

        try (Connection conn = DriverManager.getConnection(url, user, password);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SHOW TimeZone")) {

            if (rs.next()) {
                String sessionTimeZone = rs.getString(1);
                System.out.println("JDBC 세션의 PostgreSQL TimeZone: " + sessionTimeZone);
            }

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}

```

```yaml
JDBC 세션의 PostgreSQL TimeZone: Asia/Seoul
```

---

## 참고: JVM 타임존 확인 (RDB 서버가 아닌 Java JVM 측)

```java
System.out.println("JVM default TimeZone: " + java.util.TimeZone.getDefault().getID());
```
