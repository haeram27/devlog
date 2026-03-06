# redis-cli

## 접속

접속 명령:

```bash
redis-cli -h <ip> -p <port> -a <password> -n <dbnum>
# redis-cli -h 1.1.1.1 -p 6379 -a password -n 1
redis-cli -u redis://<user>:<password>@<host>:<port>/<dbnum>
# redis-cli redis://default:password@1.1.1.1:6379/1
```

- 별도로 user를 설정 하지 않았다면 user의 기본값은 `default` 임

Usage:

```text
Usage: redis-cli [OPTIONS] [cmd [arg [arg ...]]]
  -h <hostname>      Server hostname (default: 127.0.0.1).
  -p <port>          Server port (default: 6379).
  -t <timeout>       Server connection timeout in seconds (decimals allowed).
                     Default timeout is 0, meaning no limit, depending on the OS.
  -s <socket>        Server socket (overrides hostname and port).
  -a <password>      Password to use when connecting to the server.
                     You can also use the REDISCLI_AUTH environment
                     variable to pass this password more safely
                     (if both are used, this argument takes precedence).
  --user <username>  Used to send ACL style 'AUTH username pass'. Needs -a.
  --pass <password>  Alias of -a for consistency with the new --user option.
  --askpass          Force user to input password with mask from STDIN.
                     If this argument is used, '-a' and REDISCLI_AUTH
                     environment variable will be ignored.
  -u <uri>           Server URI on format redis://user:password@host:port/dbnum
                     User, password and dbnum are optional. For authentication
                     without a username, use username 'default'. For TLS, use
                     the scheme 'rediss'.
  -r <repeat>        Execute specified command N times.
  -i <interval>      When -r is used, waits <interval> seconds per command.
                     It is possible to specify sub-second times like -i 0.1.
                     This interval is also used in --scan and --stat per cycle.
                     and in --bigkeys, --memkeys, --keystats, and --hotkeys per 100 cycles.
  -n <db>            Database number. 0~15.
```

- db는 0~15 총 16개의 논리적 DB 공간이 있음

## 자주 사용하는 명령어

명령어는 대소문자를 구분하지 않음

- 시스템 및 관리 (상태 확인)
  - PING: 연결 상태 확인 (정상이면 PONG 응답)
  - INFO: 서버의 상세 정보 및 통계(메모리, CPU, 클라이언트 수 등) 출력
  - DBSIZE: 현재 선택된 데이터베이스의 키 총개수 확인
  - SELECT `<db_index>`: 사용할 데이터베이스(0~15) 변경
  - FLUSHDB: 현재 DB의 모든 데이터 삭제
  - FLUSHALL: 모든 DB의 전체 데이터 삭제 (주의!)

- 키(Key) 조작 (공통)
  - KEYS `<pattern>`: 패턴과 일치하는 모든 키 조회 (운영 환경 사용 금지, KEYS * 등), 조회 중 redis 실행 blocking됨 (redis는 thread 1개로 동작)
  - SCAN `<cursor>`: 키 목록을 분할해서 조회 (운영 환경 권장)
  - EXISTS `<key>`: 특정 키가 존재하는지 확인
  - DEL `<key>`: 키 삭제
  - RENAME `<key>` `<newkey>`: 키 이름 변경
  - EXPIRE `<key>` `<seconds>`: 키의 만료 시간(초) 설정
  - TTL `<key>`: 키의 남은 유효 시간 확인 (초 단위)

- 데이터 타입별 조작 (Value)
  - Strings (일반 Key-Value)
    - SET `<key>` `<value>`: 저장
    - GET `<key>`: 조회
    - INCR `<key>`: 숫자 값을 1 증가
  - Hashes (객체 형태)
    - HSET `<key>` `<field>` `<value>`: 필드 저장
    - HGET `<key>` `<field>`: 특정 필드 조회
    - HGETALL `<key>`: 모든 필드와 값 조회
  - Lists (순서 있는 목록)
    - LPUSH `<key>` `<value>`: 왼쪽(앞)에 추가
    - RPUSH `<key>` `<value>`: 오른쪽(뒤)에 추가
    - LRANGE `<key>` `<start>` `<stop>`: 범위 조회 (예: 0 -1은 전체)
  - Sets (중복 없는 집합)
    - SADD `<key>` `<member>`: 멤버 추가
    - SMEMBERS `<key>`: 모든 멤버 조회

- Pub/Sub (발행/구독 - Spring 연동 시 중요)
  - PUBLISH `<channel>` `<message>`: 특정 채널에 메시지 발송
  - SUBSCRIBE `<channel>`: 특정 채널의 메시지 대기(구독)

- 모니터링 및 종료
  - MONITOR: 서버에 들어오는 모든 명령을 실시간으로 확인 (디버깅용)
  - QUIT 또는 EXIT: CLI 종료

## 명령 사용예

```bash
$ redis-cli -h 1.1.1.1 -p 6379 -a passw@rd -n 1

1.1.1.1:6379[1]> select 2
1.1.1.1:6379[2]> select 0
1.1.1.1:6379> keys *
1.1.1.1:6379> get key:of:my:value
1.1.1.1:6379> exit
```
