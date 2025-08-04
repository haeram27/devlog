# 고급 리디렉션

## 리다이렉션 기본 이해

- `리다이렉션 표현은 쉘이 명령어 프로세스 실행할 때 명령어 프로세스의 FD를 변경 한다.`

- `쉘은 항상 명령 라인의 모든 리다이렉션 명령을 먼저 처리한 후 명령 프로세스를 실행한다.` 여기서 명령 라인은 프로세스 실행 단위를 나타내며, ';', '|', '&&', '||' 등으로 구분 될 수 있다.
그 결과로 명령어 프로세스의 실행 결과는 모든 리다이렉션 처리가 완료된 상태의 FD로 출력되게 된다.

- 리다이렉션 표현은 명령 라인에서 명령어의 앞/중간/뒤의 어디에든 위치할 수 있으나, 리다이렉션 표현과 명령 프로세스의 상대적 위치가 최종 FD 설정에 영향을 끼치진 않는다.

- 명령 라인 내에서 `리다이렉션 표현이 여러개`인 경우 `좌에서 우로 순서대로` 처리된다.
  1. 리다이렉션 표현 `좌에서 우로 순서대로` 적용 (명령어 프로세스에 반영)
  2. 명령어 프로세스 실행

## FD.A `>&` FD.B : 복사

`>&` 표현식의 좌변과 우변은 모두 FD(file descriptor) 이다.  
FD.A를 FD.B에 맵핑되어 있는 파일(값)로 리다이렉션 한다.  
FD.B에 맵핑되어 있는 file을 FD.A의 값으로 맵핑(복사)하라는 의미이다.

## FD `>` file : 할당

좌변 FD를 우변 file(path)과 맵핑(할당)한다.
`>` 표현식의 좌변은 FD 우변은 항상 file(device 포함)이어야 한다.

## 요약 표

| 표현 | 의미 |
| ------------ | ----------------------- |
| `exec >file` | 셸 자체의 stdout 변경 |
| `> file`     | stdout → file           |
| `2> file`    | stderr → file           |
| `2>&1`       | stderr → stdout이 가리키는 곳 |
| `&>/dev/null`| stdout + stderr → 지정 파일, <br> `>/dev/null 2>&1`과 같음 |
| `\|`         | stdout → 다음 명령 stdin |
| `\|&`        | stdout + stderr → 다음 명령 stdin (Bash 4+), <br> `2>&1 \|`과 같음 |

### 1. `exec` – 셸의 기본 FD를 바꾸기

#### 예제

```bash
exec > out.txt
echo "this goes to out.txt"
```

#### 의미

- `exec > out.txt`: **셸 자체의 stdout을 out.txt로 바꿈**
- 이후 **셸에서 실행하는 모든 명령어의 stdout은 out.txt로 리디렉션됨**

#### 유용할 때

- 스크립트 전체의 출력을 특정 로그 파일로 보내고 싶을 때
- 반복되는 리디렉션을 피하고 싶을 때

---

### 2. `|` (파이프) – 명령어 간 stdout 전달

#### 예제

```bash
cat file.txt | grep "keyword" | wc -l
```

#### 의미

- `cat`의 stdout → `grep`의 stdin
- `grep`의 stdout → `wc -l`의 stdin
- 각 명령어는 **표준 출력을 다음 명령어의 표준 입력으로 연결**

#### 특징

- **stdout만 전달됨**
- **stderr는 파이프에 포함되지 않음**

---

### 3. `2>&1` – stderr을 stdout으로 합치기

#### 예제

```bash
some_command >out.log 2>&1
```

#### 의미

1. `>out.log` → **stdout → out.log**
2. `2>&1` → **stderr → "현재의 stdout이 가리키는 대상"** → `out.log`

#### 중요: `2>&1 >out.log`는 결과가 다름

- 그건 stderr → 기존 stdout (터미널)
- 그 후 stdout → out.log로 바뀜
- 그래서 stderr는 터미널로, stdout은 파일로 가게 됨

**리디렉션 순서가 매우 중요**!

---

### 4. stderr 파이프: `2>&1 |` 과 `|&`

Bash 4 이상에서는 `|&` 문법이 추가됨.

#### 예제 1

```bash
make 2>&1 | tee build.log
```

- `make`의 stdout과 stderr 모두 파이프를 통해 `tee`에 전달

#### 예제 2

```bash
make |& tee build.log
```

- 위와 동일. `|&`는 `2>&1 |`의 축약형

---

### 5. 파일 디스크립터 다루기

쉘에서는 **임의의 파일 디스크립터 번호**를 열거나 복사할 수 있음.

```bash
exec 3> debug.log   # FD 3을 열어 debug.log로 연결
echo "debug" >&3    # FD 3으로 출력
exec 3>&-           # FD 3 닫기
```

- FD 3은 사용자가 만든 임시 출력 채널
- 리디렉션을 더 세밀하게 제어 가능

---

### 복합 예제

```bash
{
  echo "stdout"
  echo "stderr" >&2
} > out.log 2>&1
```

- 명령 블록 `{}` 내부의 stdout과 stderr 모두 `out.log`로 기록
