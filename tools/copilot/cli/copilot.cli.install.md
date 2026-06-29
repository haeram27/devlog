# install copilot cli

copilot cli를 설치하려면 npm이 필요하다.  
npm은 node package manger 로써 node 배포 패키지를 통해 단독 설치할 수 도 있지만 nvm(node version manager)을 통해서 설치하는 것을 권장한다.

## GitHub Copilot CLI 설치

GitHub Copilot CLI 패키지를 설치하기 전에 NVM(nvm)과 Node.js(node, npm) 설치를 완료한다.

### online 설치

인터넷이 연결된 환경에서는 npm을 통해 직접 설치합니다.

#### Step 1. GitHub Copilot CLI 패키지 설치

NVM으로 설치한 npm은 사용자 권한이므로 `sudo` 없이 설치합니다:

```bash
npm install -g @github/copilot
```

> **주의:** `sudo npm install -g`를 사용하면 root 권한으로 설치되어 권한 문제가 발생할 수 있습니다.
> NVM 환경에서는 `sudo` 없이 사용하세요.

#### Step 2. 설치 확인 및 인증

설치가 완료되었는지 확인합니다:

```bash
copilot --version
```

처음 실행 시 대화형 세션을 열고 로그인합니다:

```bash
copilot
```

> 로그인되어 있지 않으면 CLI 내부에서 `/login` 명령을 사용하라는 안내가 표시됩니다.
> 안내에 따라 `/login` 을 입력하고 브라우저 또는 디바이스 인증 절차를 완료하면 됩니다.

#### Step 3. 사용 예시

Copilot CLI를 통해 코드 작성을 도움받을 수 있습니다:

```bash
# 대화형 모드로 시작
copilot

# 특정 파일에 대해 물어보기
copilot /path/to/file.py

# 명령어 설명 요청
copilot explain "npm install"
```

## GitHub Copilot CLI 인증 및 로그인

현재 설치된 GitHub Copilot CLI 1.0.51 기준으로는 `copilot auth ...` 형식의 하위 명령이 보이지 않습니다.
인증은 CLI를 실행한 뒤 세션 내부에서 진행합니다.

### 온라인 환경 - 대화형 세션에서 로그인

#### 1. CLI 실행

```bash
copilot
```

#### 2. 세션 내부에서 로그인

로그인되어 있지 않으면 `/login` 명령을 사용하라는 안내가 표시됩니다.

```text
/login
```

> CLI가 브라우저 인증 또는 디바이스 인증 절차를 화면에 안내합니다.
> 안내된 절차를 완료하면 현재 세션부터 바로 사용할 수 있습니다.

### GHE 포함 브라우저 불가 환경 - Device Flow

GitHub Enterprise 환경이거나 브라우저를 직접 열 수 없는 환경에서는 `/login` 실행 후 디바이스 인증 절차가 안내될 수 있습니다.

일반적으로 다음 순서로 진행됩니다.

1. `copilot` 실행
2. 세션 내부에서 `/login` 입력
3. 터미널에 표시되는 인증 URL 접속
4. 디바이스 코드 입력 후 승인

> GHE 계정의 경우 조직 환경에 따라 `https://<enterprise-host>/login/device` 형태의 URL이 표시될 수 있습니다.
> 이 URL은 CLI가 실제 환경에 맞춰 출력하는 값을 그대로 사용하면 됩니다.

### 토큰 기반 인증

현재 CLI는 Personal Access Token을 환경변수로 제공하는 방식도 지원합니다.

#### 1. 토큰 생성

GitHub에서 fine-grained PAT를 생성하고 **Copilot Requests** 권한을 추가합니다.

#### 2. 환경변수 설정

```bash
export GH_TOKEN=YOUR_TOKEN
```

또는:

```bash
export GITHUB_TOKEN=YOUR_TOKEN
```

> 우선순위는 `GH_TOKEN` 이 `GITHUB_TOKEN` 보다 높습니다.

#### 3. CLI 실행

```bash
copilot
```

### 인증 관련 명령어 및 방식

| 방식 | 설명 |
|--------|------|
| `copilot` | 대화형 세션 시작 |
| `/login` | 세션 내부에서 로그인 시작 |
| `GH_TOKEN=... copilot` | PAT로 인증된 상태로 실행 |
| `GITHUB_TOKEN=... copilot` | 대체 환경변수로 PAT 제공 |

### 인증 트러블슈팅

| 문제 | 해결 방법 |
|------|---------|
| `Invalid command format` | `copilot auth login` 형식은 현재 설치된 1.0.51 기준으로 맞지 않음. `copilot` 실행 후 `/login` 사용 |
| 로그인 URL이 GHE 주소로 표시됨 | 정상일 수 있음. 조직 환경에 맞는 `https://<enterprise-host>/login/device` URL을 그대로 사용 |
| 브라우저를 열 수 없음 | `/login` 이후 표시되는 디바이스 인증 URL과 코드로 진행 |
| 인증이 계속 실패함 | `GH_TOKEN` 또는 `GITHUB_TOKEN` 으로 PAT 설정 후 새 세션에서 `copilot` 재실행 |

---

## 문제 해결

| 문제 | 해결 방법 |
|------|---------|
| `copilot: command not found` | `nvm use v24.16.0` 실행 후 재시도 또는 `~/.zshrc`에 NVM 설정 확인 |
| 인증 실패 | 외부망에서 생성한 GitHub 토큰 사용 또는 `logout` 후 재인증 |
| 패키지 설치 실패 | `npm cache clean --force` 실행 후 재설치 |
| 권한 오류 | NVM 사용 시 `sudo` 없이 설치하기. `sudo` 사용 시 root 권한으로 설치되어 충돌 발생 |

---

설치 완료 후 `copilot --help`로 모든 사용 가능한 명령어를 확인할 수 있습니다.


## BYOK

## shell complition

`copilot completion` 명령은 **shell completion** 파일을 생성합니다. 이는 터미널에서 Tab 키를 눌렀을 때 자동완성 기능을 제공합니다.

예를 들어:
```bash
copilot <Tab>          # 사용 가능한 하위 명령어 표시 (explain, auth 등)
copilot --<Tab>        # 사용 가능한 옵션 표시
copilot explain <Tab>  # 관련 옵션 표시
```

zsh와 bash 통합: `copilot completion --help`
```bash
if command -v copilot &>/dev/null; then
    if [[ "$SHELL" == *"zsh" && ! -s ~/.zsh/completions/_copilot ]]; then
        mkdir -p ~/.zsh/completions &>/dev/null
        fpath=(~/.zsh/completions $fpath)

        if command -v copilot &>/dev/null && [[ ! -s ~/.zsh/completions/_copilot ]]; then
            copilot completion zsh > ~/.zsh/completions/_copilot
        fi

        autoload -Uz compinit && compinit
    elif [[ "$SHELL" == *"bash" ]]; then
        source <(copilot completion bash)
    fi
fi
```
