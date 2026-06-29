# Spring Batch 사용시 제약 사항

## JDBC를 이용한 ItemReader

- RDB의 timestamptz 형식을 DTO의 ZonedDateTime으로 자동 매핑이 되지 않는다.
JDBC에는 timestamptz 형식을 java.sql.Timestamp로 맵핑하고 이 타입은 ZondedDateTime으로 자동 맵핑 되지 않는다.