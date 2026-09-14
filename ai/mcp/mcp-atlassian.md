# MCP Atlassian

`mcp-atlassian`은 atlassian의 Confluence나 Jira와 AI를 연동하기 위한 open-souece MCP 프로젝트이다.

Model Context Protocol (MCP) server for Atlassian products (Confluence and Jira). Supports both Cloud and Server/Data Center deployments.

atlassian 공식 mcp가 cloud 지원 전용이라면, 이 프로젝트는 사내 설치형(Server/DataCenter) 제품과도 연동이 가능하다.

- [github repository](https://github.com/sooperset/mcp-atlassian)
- [공식 문서](https://mcp-atlassian.soomiles.com/docs)

## python과 uv 설치

`mcp-atlassian`은 python으로 제작된 프로젝트 이므로 사용하려면, python 실행 환경이 필요하다.

### python 설치

다음은 winodws에서 winget을 이용하여 python을 설치하는 명령어 이다.

```pwsh

# 설치 가능한 python 패키지 목록
winget search python.python

# 설치된 파이썬 목록과 정확한 ID 확인
winget list Python

# python 버전 3.12 설치
winget install --id Python.Python.3.14 -e --accept-source-agreements
python --version

# 과거 버전의 ID를 지정하여 삭제
winget uninstall --id Python.Python.3.12

# 각 python 배포를 최신 (마이너 업그레이드) 버전으로 업그레이드
winget upgrade --id Python.Python.3.12
```

만약 인터넷 연결이 자유로운 환경이라면 uv를 이용하여 python 및 각종 패키지를 다운로드는 하는 것이 가능하다.

```pwsh
# system에 설치된 uv가 인식하는 python 배포 리스트
uv python list

# python 최신 버전 설치
uv python install latest

# python 특정 버전 설치
uv python install 3.13
```

### uv 설치

`uv`는 개발사 Astral이 개발한 Rust 기반의 통합 파이썬 프로젝트 및 패키지 관리자입니다. 기존 파이썬 생태계에서 파편화되어 있던 여러 도구들을 단 하나의 실행 파일(Binary)로 통합하여 "파이썬 생태계의 Cargo(Rust의 패키지 매니저)"를 목표로 만들어진 도구입니다

- uv.exe : 파이썬 버전 변경, 가상환경 생성, 프로젝트 라이브러리 추가 등 파이썬 개발 환경 전체를 관리하고 설치하는 메인 도구입니다.
- uvx.exe : mcp-atlassian 같은 파이썬 기반 CLI 프로그램을 컴퓨터에 영구 설치하지 않고, 격리된 임시 환경에서 1회성으로 즉시 실행해 주는 도구입니다.
- uvw.exe : 윈도우 환경에서 터미널(콘솔) 창이 보이지 않게 백그라운드 전용(Windowless)으로 파이썬 스크립트나 프로그램을 실행해 주는 특수 실행기입니다.

```pwsh
# uv 설치 
winget install --id=astral-sh.uv -e
uv --version
```

## mcp-atlassian 설치

`mcp-atlassian`은 `uv`가 공식 릴리즈된 바이너리를 자동으로 다운로드 하여 설치하므로 인터넷이 연결된 환경이 라면 별도로 소스나 바이너리를 직접 다운로드 할 필요가 없다.

```pwsh
# install mcp-atlassian.exe
uv tool uninstall mcp-atlassian
uv tool install mcp-atlassian

# update path env(register env variables)
uv tool update-shell
```

만약 인터넷이 연결되지 않는 환경이라면, 별도로 다운로드 받아서 설치하는 방법을 고려해야 한다.

## IDE에 mcp-atlassian 등록

### Claude CLI (Claude Code) 설정 방법

Claude CLI는 내장된 claude mcp 명령어를 사용해 복잡한 파일 편집 없이 터미널에서 바로 MCP 서버를 추가할 수 있어 매우 직관적입니다.
PowerShell에 아래 명령어를 복사하여 입력하세요:

```pwsh
# asdfkasdf
claude mcp add mcp-atlassian uvx mcp-atlassian
```

등록이 끝나면 Claude CLI를 다시 켰을 때 자동으로 해당 도구들을 인식합니다.

### GitHub Copilot CLI 설정 방법

GitHub Copilot CLI 또한 /mcp 명령어나 copilot mcp add 전용 명령어를 기본 제공합니다.
다만 CLI 환경 변수를 인자로 직접 밀어 넣는 것보다는 대화형 폼(Form) 창을 띄워 등록하는 것이 에러가 없고 깔끔합니다.

```pwsh
# PowerShell에서 대화형 모드로 Copilot CLI를 시작합니다.
copilot --autopilot
   
# 프롬프트 창에 다음 명령어를 입력하여 MCP 서버 추가 양식을 엽니다.
/mcp add

# 대화형 마법사가 뜨면 안내에 따라 아래 항목들을 차례로 입력합니다:
* Server Name: mcp-atlassian
    * Transport Type: STDIO
    * Command: uvx mcp-atlassian
```

## 사용자 환경 변수에 ATLASSIAN 액세스 정보 등록

windows/linux에서 다음의 사용자 환경 변수를 등록하면 mcp-atlassian이 자동으로 접속 정보를 사용하게 된다.

Authentication 관련 환경변수는 atlassian cloud냐 Server 형이냐에 따라서 약간의 차이가 있으므로 공식문서를 참조한다.

- [Cloud](https://mcp-atlassian.soomiles.com/docs/authentication#api-token-cloud-recommended)
- [Server/DataCenter](https://mcp-atlassian.soomiles.com/docs/authentication#personal-access-token-server/data-center)

jira 또는 confluence에 로그인하여 프로필 메뉴에서 PAT(Personal Access Tokens)를 생성할 수 있다.

- `프로파일 > 환경설정 > 개인용 액세스 토큰`

다음은 Server/Data Center(사내 서버 설치형 atlassian)인 경우 사용하는 PAT 환경변수를 windows 사용자 변수로 추가하는 명령라인이다.

```pwsh
[System.Environment]::SetEnvironmentVariable('JIRA_URL', 'https://atlassian.net', 'User')
[System.Environment]::SetEnvironmentVariable('JIRA_PERSONAL_TOKEN', 'YOUR_ACTUAL_PERSONAL_TOKEN', 'User')
[System.Environment]::SetEnvironmentVariable('CONFLUENCE_URL', 'https://atlassian.net', 'User')
[System.Environment]::SetEnvironmentVariable('CONFLUENCE_PERSONAL_TOKEN', 'YOUR_ACTUAL_PERSONAL_TOKEN', 'User')
```

사용자 환경 변수가 등록되면, 이를 사용하는 프로그램(터미널 등)은 새로 시작하여야 적용된다.

이후 ai 도구(claude 또는 copilot)에서 다음과 같이 명령하면 atlassian 에서 정보를 읽어와 응답을 한다.

- `나에게 할당된 미해결 Jira 목록을 알려줘`
- `Confluence 페이지 123456789 내용을 읽고 5줄로 요약해줘`
- `https://.../wiki/spaces/TEAM/pages/123456789/... 이 페이지 읽어서 액션 아이템만 뽑아줘`
- `Confluence space_key=TEAM, title="배포 가이드" 페이지 찾아서 핵심 절차 정리해줘`
