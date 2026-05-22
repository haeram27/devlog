# nvm 설치 하기 (node, npm 버전 호환 관리)

nvm은 현재 환경의 node 버전을 관리해 주는 프로그램이다.
nvm을 사용하여 여러 버전의 node를 설치할 수 있고 또 현재 환경에 맞는 node 버전을 선택하여, 여러 버전을 변경해 가며 사용할 수 있다.

nvm을 설치하면 호환 버전의 node.js와 npm이 함꼐 설치되므로 시스템에 기존에 standalone node나 npm이 설치되어 있다면 충돌 방지를 위하여 삭제 하도록 한다.

## nvm 설치

### online 설치

#### Step 1. 필수 도구 설치 및 스크립트 실행

터미널을 열고, 스크립트 다운로드를 위한 curl 설치 후 [NVM 공식 GitHub 저장소](https://github.com/nvm-sh/nvm)의 최신 버전 설치 스크립트를 다운로드하여 실행합니다.  

1. 패키지 리스트 업데이트 및 curl 설치
sudo apt update && sudo apt install -y curl

2. NVM 최신 설치 스크립트 실행

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash
```

#### Step 2. 터미널에 환경 변수 반영하기  

설치 스크립트는 사용자의 셸 설정 파일(~/.bashrc 또는 ~/.zshrc)에 환경 변수를 자동으로 추가합니다. 변경 내용을 현재 터미널 창에 즉시 반영해야 합니다.  

1. 설정 반영 (사용 중인 셸에 맞게 선택)

```bash
source ~/.bashrc
```

2. NVM 설치 정상 완료 확인 (버전 숫자가 출력되면 성공)

```bash
nvm --version
```

(※ 만약 command not found: nvm 에러가 발생한다면 터미널 창을 완전히 닫았다가 다시 실행해 보세요.)  

#### Step 3. NVM으로 Node.js 설치 및 설정  

NVM 설치가 끝났으므로, 앞서 활용하려 했던 GitHub Copilot CLI 요구 사양(Node.js 22 이상)에 맞춰 최신 안정 버전을 다운로드합니다.  

1. 최신 LTS(장기 지원 안정 버젼) Node.js 설치 (권장)

```bash
nvm install --lts  # node lts 버전 설치
# 또는
nvm install 24  # 특정 메이저 버전(예: Node.js 24) 설치
```

설치 확인:

```bash
node -v
npm -v
```

만약 `node` 명령이 실행되지 않으면 현재 세션에서 Node.js를 활성화 후 다시 실행

```bash
nvm use node  # 또는 nvm use 24 특정 버전
```

향후 모든 새 터미널에서 자동 활성화되도록 기본값을 지정합니다:

```bash
nvm alias default node
```

> **참고:** `nvm alias default node`는 항상 최신 LTS 버전을 가리킵니다. 
> 특정 버전 고정이 필요하면 `nvm alias default 24` 형태로 사용할 수 있습니다.

#### NVM 주요 활용 명령어 모음

- `nvm ls`: 현재 내 시스템에 설치된 Node.js 버전 목록을 확인합니다.
- `nvm ls-remote`: 설치 가능한 모든 Node.js 버전 목록을 원격 조회합니다.
- `nvm use <버전>`: 설치된 다른 Node.js 버전으로 즉시 스위칭합니다.  


#### 셸 환경설정 파일에 NVM 환경 변수 수동 수록

일반적으로 설치 스크립트를 사용하여 nvm을 설치한 경우 환경변수가 자동으로 `~/.bashrc`나 `~/.zshrc`에 추가된다. 별도의 스크립트를 통해서 nvm 환경 변수를 등록하고자 하는 경우 아래와 같이 작성한다.

적용된 파일의 내용 (~/.env.cust):

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" # This loads nvm bash_completion
```

설정 즉시 반영 및 설치 확인

```bash
source ~/.env.cust
nvm --version
```