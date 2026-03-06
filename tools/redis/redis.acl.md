# redis access control

## 계정 관리 명령

과거(Redis 6.0 이전)에는 별도의 계정 개념 없이 단일 비밀번호(requirepass)만 사용했으나, Redis 6.0부터 ACL(Access Control List) 기능이 도입되면서 상세한 계정 생성이 가능해졌습니다.

## requirepass (before 6.0)

### redis-cli를 통한 설정

```bash
127.0.0.1:6379> CONFIG SET requirepass newpassword123
127.0.0.1:6379> CONFIG REWRITE
```

### redis.conf에 requirepass 직접 설정 예

#### redis.conf

```conf
# redis.conf 파일 내용 예시
requirepass mypassword
```

## ACL (since 6.0)

### 계정 생성 및 권한 부여 (ACL SETUSER)

redis-cli에 접속한 후 ACL SETUSER 명령어를 사용합니다.

```bash
# 형식: ACL SETUSER [계정명] [활성화] [비밀번호] [권한]
ACL SETUSER myuser on >mypassword123 ~user:* +get +set
```

- `on`: 계정을 활성화합니다.
- `>mypassword123`: > 기호 뒤에 비밀번호를 설정합니다.
- `~user:*`: 접근 가능한 키 패턴을 지정합니다. (여기서는 user:로 시작하는 키만 가능)
- `+get +set`: 허용할 명령어를 지정합니다. (전체 권한은 +@all, 읽기 전용은 +@read)

### 생성된 계정 확인

``` bash
# 전체 계정 목록 보기
127.0.0.1:6379> ACL LIST

# 특정 계정의 상세 권한 보기
127.0.0.1:6379> ACL GETUSER myuser
```

### 생성한 계정으로 로그인 테스트

```bash
# CLI 내부에서 전환
127.0.0.1:6379> AUTH myuser mypassword123

# 또는 CLI 접속 시 바로 로그인
$ redis-cli -u redis://myuser:mypassword123@127.0.0.1:6379
```

### 계정 삭제

```bash
127.0.0.1:6379> ACL DELUSER myuser
```

### 계정 영구저장

```bash
127.0.0.1:6379> ACL SAVE
```

주의사항: ACL SETUSER로 생성한 계정은 서버를 재시작하면 사라질 수 있습니다. 설정을 유지하려면 `redis.conf`에 `aclfile` 경로를 반드시 지정히고 계정 생성 후 `ACL SAVE` 명령어를 실행하여 파일에 저장해야 합니다.

#### redis.conf 

```conf
# redis.conf 파일 내용 예시
aclfile /etc/redis/users.acl
```
