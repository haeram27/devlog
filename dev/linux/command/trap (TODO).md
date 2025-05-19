# trap

`trap` 명령어에서 `ERR`과 `EXIT`는 **특수한 시그널**로, 다른 시스템 신호와는 다르게 **Bash 스크립트의 오류나 종료**와 관련된 이벤트를 처리할 수 있게 해줍니다. 이들에 대해 설명하겠습니다.

### 1. **ERR (Error)**

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

### 2. **EXIT (Script Exit)**

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

---

### `ERR`과 `EXIT`의 차이점

- `ERR`는 **명령어 실패**에 반응하고, `EXIT`는 **스크립트 종료** 시 실행됩니다.
- `ERR`은 특정 명령어가 **실패**했을 때만 실행되며, `EXIT`은 스크립트가 **정상 종료**되거나 **비정상 종료**되었을 때 모두 실행됩니다.

---

### 예시: `ERR`과 `EXIT`을 함께 사용하는 경우

```bash
#!/bin/bash

trap 'echo "An error occurred. Exiting."' ERR
trap 'echo "Script is about to exit."' EXIT

echo "Starting the script"

# 예시 명령어 - 실패하는 명령어
non_existent_command

echo "This line will not be reached"
```

- 이 스크립트는 `non_existent_command`가 실패할 때 `ERR` 트랩을 실행하고, 이후 스크립트가 종료될 때 `EXIT` 트랩이 실행됩니다.

---

### 결론

- **`ERR`**: 명령어가 실패할 때 실행되는 트랩입니다.
- **`EXIT`**: 스크립트가 종료될 때 실행되는 트랩입니다.

이 두 가지를 적절하게 사용하면, 스크립트에서 오류 처리 및 종료 작업을 보다 효율적으로 처리할 수 있습니다.

`trap` 명령어는 Bash 스크립트에서 특정 **신호(signal)**가 발생할 때 실행할 동작을 지정하는 데 사용됩니다. 예를 들어, 사용자가 스크립트를 종료하거나 시스템 신호가 전달되면 특정 작업을 처리하도록 설정할 수 있습니다.

`sigspec`은 **신호의 종류**를 나타내며, 시스템에서 발생하는 다양한 신호들에 대응할 수 있습니다.

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
