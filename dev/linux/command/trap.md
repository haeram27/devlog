# trap

`trap` 명령어는 Bash 스크립트에서 특정 **신호(signal)**가 발생할 때 실행할 동작을 지정하는 데 사용됩니다. 예를 들어, 사용자가 스크립트를 종료하거나 시스템 신호가 전달되면 특정 작업을 처리하도록 설정할 수 있습니다.

`sigspec`은 **신호의 종류**를 나타내며, 시스템에서 발생하는 다양한 신호들에 대응할 수 있습니다.

Syntax:

```bash
trap <command> <sigspec>...
trap -p
trap -l 또는 'bash -c "trap -l"'
```

sigspec을 명시할 때 signal 이름은 case insesitive 이며, `SIG` prefix 사용은 optional이다.
-p 옵션은 현재 trap에 설정된 sigspec과 매치되는 명령을 출력하낟.
-l 옵션은 지정 가능한 시스템 시그널 리스트를 출력한다.

## 특수 sigspec

- DEBUG : 각 단일 명령(simple command), 함수 호출, '!'(논리부정)으로 시작 하지 않은 모든 명령 실행 직전에
- ERR : 각 단일 명령(simple command), 함수 호출이 비정상 종료값(0이 아닌 값)을 반환했을 때
- EXIT : 스크립트가 종료되기 직전, 스크립트는 전체 실행을 마치거나 exit 명령에 등에 의해서 종료될 수 있으며 EXIT 트랩 발동시 종료의 원인은 고려하지 않음

### 1. **DEBUG (Command Debug)**

- **의미**: `DEBUG`는 **명령어 또는 함수 실행 직전에** 발생되는 시그널입니다.

- **주의사항**: `DEBUG`는 `!` (논리 부정, reserved word) 예약어로 시작하는 명령에는 발생되지 않습니다. `!`가 직접 명시된 명령어/함수의 실행 시점에는 발생하지 않지만 해당 명령어/함수의 내부에서 실행되는 명령어/함수에는 적용이 됩니다.

- **주요용도**:
  - 명령 추적(logging): 어떤 명령이 어떤 순서로, 어떤 인자와 함께 호출되는지 실시간 로그 남기기
  - 프로파일링/계측(monitoring): 특정 명령 앞뒤에 타임스탬프나 성능 측정 코드 삽입
  - 커맨드 검증: 실행 전 커맨드를 검사해서, 위험한 옵션이 있으면 중단하거나 사용자에게 확인 요청
  - “마지막 명령” 기록: DEBUG 트랩 내에서 이전 $BASH_COMMAND 값을 저장해 두었다가, EXIT 트랩에서 에러 상황 보고

- **예시**: 스크립트 종료 직전 마지막 으로 실행한 명령을 로그로 남기기

```bash
# 1) Debug 옵션 설정: -e = "set -o errexit", 명령/함수가 비정상 종료코드를 반환시 스크립트 종료
set -e

# 2) DEBUG 트랩: 각 명령 실행 직전에 발생되어 실행 대상 명령을 변수에 저장
prev_cmd="none"
current_cmd="none"
trap 'prev_cmd=$current_cmd; current_cmd=$BASH_COMMAND' DEBUG

# 3) EXIT 트랩에서 기록된 명령어 출력
destructor() {
    echo "### Print last executed commands"
    echo "prev_cmd: ${prev_cmd}"
    echo "current_cmd: ${current_cmd}"
}
trap destructor EXIT
``` 


### 2. **ERR (Error)**

- **의미**: `ERR`는 **명령어 실행이 실패했을 때** 실행되는 트랩입니다. 특정 명령어가 **비정상 종료(비영향 오류)** 되면 지정된 동작을 실행할 수 있습니다.
- **조건**: `trap`에 `ERR`을 지정하면, **스크립트 내에서 명령어가 실패했을 때** 이 트랩이 발생합니다.
- **주요 용도**: 오류가 발생했을 때 에러 메시지를 출력하거나 로그를 기록하는 등의 동작을 수행할 수 있습니다.

   ```bash
   trap 'echo "Error occurred. Exiting."' ERR
   ```

   위의 예시는 명령어 실행 중 오류가 발생하면 `"Error occurred. Exiting."` 메시지를 출력합니다.

   **참고**: `ERR`은 **명령어가 실패했을 때** 실행됩니다. 예를 들어, 존재하지 않는 파일을 읽으려 할 때 오류가 발생하면 `ERR` 트랩이 호출됩니다.

   ```bash
   # 예시
   trap 'echo "Error occurred while executing a command."' ERR
   some_non_existent_command
   ```

   위 코드에서 `some_non_existent_command`가 실행되면 `ERR` 트랩이 호출되고 메시지가 출력됩니다.

---

### 3. **EXIT (Script Exit)**

- **의미**: `EXIT`는 스크립트가 종료될 때 실행됩니다. **스크립트가 정상적으로 종료되거나 종료 코드가 반환될 때** 트랩이 실행됩니다.
- **조건**: `EXIT`은 스크립트의 **종료 시점**에 호출됩니다. 오류 코드와 상관없이 스크립트가 종료될 때 실행됩니다.
- **주요 용도**: 스크립트가 종료되기 전에 정리 작업을 수행하거나 로그를 기록하는 데 유용합니다.

    ```bash
    trap 'echo "Script is exiting."' EXIT
    ```

   위의 예시는 스크립트가 종료될 때마다 `"Script is exiting."` 메시지를 출력합니다.

   ```bash
   # 예시
   trap 'echo "Script finished."' EXIT
   echo "Doing some work"
   exit 0
   ```

   이 스크립트는 실행이 끝나면 `"Script finished."`라는 메시지를 출력하고 종료됩니다.

   EXIT는 대부분 script 종료시 처리해야하는 로직을 명시할 때 많이 사용됩니다.

    ```bash
    cleanup() {
        rm -rf temporaryfiles
    }
    trap 'cleanup &>/dev/null' EXIT
   ```

---

### 주요 `sigspec` 종류와 의미

#### 1. **SIGINT (Interrupt)**

- **신호 번호**: 2
- **의미**: 사용자로부터 `Ctrl+C` 입력을 받았을 때 발생합니다. 이 신호는 실행 중인 프로세스를 중단시키고 종료하려고 할 때 사용됩니다.

   ```bash
   trap 'echo "Ctrl+C was pressed!"' SIGINT
   ```

#### 2. **SIGTERM (Terminate)**

- **신호 번호**: 15
- **의미**: 프로세스를 종료하도록 요청하는 신호입니다. 일반적으로 프로세스를 정상적으로 종료할 때 사용됩니다.

   ```bash
   trap 'echo "Process is being terminated."' SIGTERM
   ```

#### 3. **SIGKILL (Kill)**

- **신호 번호**: 9
- **의미**: 강제로 프로세스를 종료시키는 신호로, `trap` 명령으로 처리할 수 없습니다. 시스템이 프로세스를 강제로 종료할 때 사용됩니다.

   ```bash
   trap 'echo "This will not be caught!"' SIGKILL  # 무시됨
   ```

#### 4. **SIGQUIT (Quit)**

- **신호 번호**: 3
- **의미**: 프로세스가 `Ctrl+\`로 종료될 때 발생하는 신호입니다. `SIGQUIT`은 종료할 때 **코어 덤프**를 생성합니다.

   ```bash
   trap 'echo "Ctrl+\ was pressed"' SIGQUIT
   ```

#### 5. **SIGHUP (Hangup)**

- **신호 번호**: 1
- **의미**: 터미널이나 세션이 종료될 때 발생하는 신호입니다. 종종 서버 프로그램에서 "재시작"을 의미하는 신호로 사용됩니다.

   ```bash
   trap 'echo "Terminal closed or session hangup"' SIGHUP
   ```

#### 6. **SIGUSR1 (User-defined Signal 1)**

- **신호 번호**: 10
- **의미**: 사용자가 정의한 신호로, 특정 사용자 동작에 대응할 수 있는 신호입니다.

   ```bash
   trap 'echo "User-defined signal 1 received!"' SIGUSR1
   ```

#### 7. **SIGUSR2 (User-defined Signal 2)**

- **신호 번호**: 12
- **의미**: `SIGUSR1`과 유사하게 사용자가 정의한 다른 신호입니다.

   ```bash
   trap 'echo "User-defined signal 2 received!"' SIGUSR2
   ```

#### 8. **SIGSTOP (Stop)**

- **신호 번호**: 19
- **의미**: 프로세스를 일시 정지시키는 신호입니다. 이 신호는 `trap`으로 처리할 수 없습니다.

   ```bash
   trap 'echo "This cannot catch SIGSTOP"' SIGSTOP  # 무시됨
   ```

#### 9. **SIGCONT (Continue)**

- **신호 번호**: 18
- **의미**: 일시 정지된 프로세스를 다시 실행하도록 하는 신호입니다.

   ```bash
   trap 'echo "Resuming process."' SIGCONT
   ```

#### 10. **SIGSEGV (Segmentation Violation, Segfault)**

- **신호 번호**: 11
- **의미**: 메모리 접근 오류가 발생했을 때 발생하는 신호로, 프로세스가 잘못된 메모리 위치를 참조할 때 발생합니다.

   ```bash
   trap 'echo "Segmentation fault occurred!"' SIGSEGV
   ```

#### 11. **SIGALRM (Alarm)**

- **신호 번호**: 14
- **의미**: `sleep` 명령이나 타이머에 의해 지정된 시간 초과 후 발생하는 신호입니다.

   ```bash
   trap 'echo "Alarm signal received"' SIGALRM
   ```

#### 12. **SIGPIPE (Broken Pipe)**

- **신호 번호**: 13
- **의미**: 파이프를 통해 데이터를 보내는 프로세스가 데이터를 받을 수 없을 때 발생하는 신호입니다. 예를 들어, `stdout`에 데이터를 보내는데 이를 받는 프로세스가 종료된 경우 발생합니다.

   ```bash
   trap 'echo "Broken pipe detected."' SIGPIPE
   ```

#### 13. **SIGFPE (Floating Point Exception)**

- **신호 번호**: 8
- **의미**: 수학적 계산에서 오류가 발생할 때 발생하는 신호입니다(예: 0으로 나누기).

   ```bash
   trap 'echo "Floating point exception!"' SIGFPE
   ```

#### 14. **SIGCHLD (Child Process Terminated)**

- **신호 번호**: 17
- **의미**: 자식 프로세스가 종료되었을 때 부모 프로세스에게 보내는 신호입니다. 이 신호를 사용하여 자식 프로세스의 종료 상태를 처리할 수 있습니다.

   ```bash
   trap 'echo "Child process terminated"' SIGCHLD
   ```

#### 15. **SIGTSTP (Terminal Stop)**

- **신호 번호**: 20
- **의미**: `Ctrl+Z` 키를 누를 때 발생하는 신호로, 프로세스를 일시 정지시키는 신호입니다.

   ```bash
   trap 'echo "Ctrl+Z pressed"' SIGTSTP
   ```

#### 16. **SIGCONT (Continue)**

- **신호 번호**: 18
- **의미**: 일시 정지된 프로세스를 다시 실행하도록 하는 신호입니다.

   ```bash
   trap 'echo "Process resumed"' SIGCONT
   ```

### `trap` 명령 사용 예시

```bash
trap 'echo "Ctrl+C pressed! Exiting..."' SIGINT
```

위 예시는 `Ctrl+C`가 눌렸을 때 `"Ctrl+C pressed! Exiting..."` 메시지를 출력하고 스크립트를 종료하는 방식입니다.

### 전체 신호 목록

- `trap` 명령어에서 사용할 수 있는 **모든 신호**는 `kill -l` 명령을 통해 확인할 수 있습니다:

```bash
kill -l
```

위 명령을 실행하면 시스템에서 사용할 수 있는 모든 신호 목록을 볼 수 있습니다.

---

### 정리

Bash에서 `trap` 명령어를 사용하여 다양한 시스템 신호에 대해 반응할 수 있습니다. 각 신호는 프로그램 흐름에 중요한 역할을 하며, `trap`을 사용하여 오류 처리, 프로세스 종료, 타이머 등의 이벤트를 제어할 수 있습니다.
