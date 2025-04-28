
# ps (report a snapshot of the current processes)

## options

| 항목 | 설명 |
|:---:|:---|
| -f | Do full-format listing. |
| -F | Extra full format. |
| -l | Long format.  The -y option is often useful with this. |
| -y | Do not show flags; show rss in place of addr. This option can only be used with -l. |
| -w | Wide output.  Use this option twice for unlimited width. |

To see every process on the system using standard syntax:

```bash
ps -ef
```

```bash
ps -efww
```

```bash
ps -efly
```

To print a process tree:

```bash
ps -ejH
```

To get info about threads:

```bash
ps -eLf
```

To see favorite items only or use `top` command

```bash
ps -eo pid,ppid,user,group,%cpu,%mem,etime,stat,args
```

To get security info:

```bash
ps -eo euser,ruser,suser,fuser,f,comm,label
```

To see every process with a user-defined format:

```bash
ps -eo pid,tid,class,rtprio,ni,pri,psr,pcpu,stat,wchan:14,comm
```

---

## 예시

```bash
ps -ef | grep bash
```

출력:

```
UID        PID  PPID  C STIME TTY          TIME CMD
user      1234  1220  0 10:23 pts/0    00:00:00 bash
```

- `pts/0`: 이 프로세스는 **pseudo-terminal 0**, 즉 **터미널 에뮬레이터 창**에서 실행 중
- `tty1`: 물리적인 콘솔 (Ctrl+Alt+F1 같은)
- `?`: **터미널이 없는 백그라운드 프로세스** (예: 데몬)


### 추가 설명

- `TTY`가 `?`인 경우 → 터미널이 없는 데몬, 서비스 등
- `STIME`은 프로세스가 오래된 경우 날짜(`Apr10`), 최근일 경우 시간(`14:32`) 형식으로 나옴
- `CMD`는 `ps`의 출력 너비에 따라 일부 잘릴 수도 있음 → 전체 확인은 `ps -efww` 사용 가능

### `ps -ef` 출력 항목

| 열 이름 | 의미 (영문) | 의미 (한글) |
|:---|:---|:---|
| `UID`   | User ID                          | 프로세스를 실행한 사용자 이름   |
| `PID`   | Process ID                       | 프로세스의 고유 식별자   |
| `PPID`  | Parent Process ID                | 부모 프로세스의 식별자   |
| `C`     | CPU usage (recent %)             | CPU 사용률 (최근 값, 정수형)   |
| `STIME` | Start time/date of the process   | 프로세스가 시작된 시각 또는 날짜   |
| `TTY`   | Controlling terminal             | 프로세스가 연결된 터미널 또는 장치   |
| `TIME`  | Total accumulated CPU time       | 누적 CPU 사용 시간 (user + system)   |
| `CMD`   | Command with all arguments       | 실행된 전체 명령줄 (명령 + 인자 포함) |

---

## `TTY`

**해당 프로세스가 연결된 터미널 장치**를 나타냅니다.

### 상세 설명

| 항목 | 설명 |
|:---|:---|
| `TTY` | **프로세스가 입력/출력을 수행하는 터미널 장치 이름** |
| 예시 값 | `pts/0`, `tty1`, `?` 등 |
| 전체 경로 | 실제로는 `/dev/tty1`, `/dev/pts/0` 와 같은 디바이스 파일 |
| 의미 | 로그인 세션, 터미널, 또는 가상 콘솔과의 연결 상태를 보여줌 |

### 자주 보는 `TTY` 값

| 필드 | 의미 |
|------|------|
| `TTY` | **프로세스가 연결된 터미널 장치 이름** |
| 값이 `?` | 터미널 없이 실행된 프로세스 (데몬 등) |
| 값이 `pts/N` | 가상 터미널 (터미널 앱, SSH 등), N == numeric |
| 값이 `ttyN` | 물리 콘솔 (Ctrl+Alt+F1 같은 콘솔 창) N == numeric |


### 관련 명령어

- 현재 내 터미널 확인:

```bash
tty
```

출력 예: `/dev/pts/0`

- `ls -l /proc/$$/fd/0` → 현재 쉘의 표준 입력(0)의 터미널 연결 확인

---

## -o 옵션 사용시 가용 필드 목록

| 필드 이름 | 설명 |
|-----------|------|
| `pid` | 프로세스 ID |
| `ppid` | 부모 프로세스 ID |
| `uid` | 사용자 ID (숫자) |
| `user` | 사용자 이름 |
| `gid` | 그룹 ID (숫자) |
| `group` | 그룹 이름 |
| `euid` | 유효 사용자 ID |
| `egid` | 유효 그룹 ID |
| `ruser` | 실제 사용자 이름 |
| `rgroup` | 실제 그룹 이름 |
| `comm` | 명령어 이름 (경로 제외) |
| `args` | 명령어 전체 인자 포함 |
| `cmd` | 명령어 전체 인자 포함 (args와 동일) |
| `etime` | 프로세스 실행 시간 (elapsed time) |
| `time` | 누적 CPU 시간 |
| `c` | CPU 사용률 |
| `pcpu` | CPU 사용률 (%) |
| `pmem` | 메모리 사용률 (%) |
| `rss` | 실제 메모리 사용량 (KB) |
| `vsz` | 가상 메모리 사용량 (KB) |
| `tty` | 연결된 터미널 |
| `stat` | 프로세스 상태 코드 |
| `start` | 프로세스 시작 시간 |
| `lstart` | 프로세스 시작 시간 (긴 형식) |
| `nice` | 우선순위 값 (NI) |
| `pri` | 우선순위 값 (PRI) |
| `nlwp` | 스레드 수 |
| `sess` | 세션 ID |
| `sid` | 세션 리더의 PID |
| `pgid` | 프로세스 그룹 ID |
| `class` | 스케줄링 클래스 |
| `flags` | 프로세스 플래그 |
| `wchan` | 대기 중인 커널 함수 |
| `fname` | 실행 파일 이름 |
| `f` | 플래그 |
| `uid` | 사용자 ID |
| `gid` | 그룹 ID |
| `euid` | 유효 사용자 ID |
| `egid` | 유효 그룹 ID |
| `ruser` | 실제 사용자 이름 |
| `rgroup` | 실제 그룹 이름 |
