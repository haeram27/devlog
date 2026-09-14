# Atlassian Skills

- [github repository](https://github.com/eunsanMountain/atlassian-skills)

**atlassian-skills vs mcp-atlassian 비교**

두 프로젝트는 토큰 소모와 문서 정밀도 문제에서 명확한 차이점을 보입니다.

| 비교 항목 | mcp-atlassian (sooperset) | atlassian-skills |
|---|---|---|
| 주요 타겟 | Atlassian Cloud 위주 (온프레미스도 지원) | Atlassian 온프레미스(Server/DC) 최적화 |
| 토큰 소모량 | 매우 많음 (복잡한 장황한 JSON 응답 구조) | 50% 이상 절감 (에이전트용 경량 컴팩트 출력) |
| Confluence 편집 정밀도 | 에디터 수정 시 마크업이 깨지거나 변형될 위험 있음 | 손실 없는 마크다운 무결성 유지 (cfxmark 증명 기술 적용) |
| 동작 방식 | 표준 MCP 통신 규격 서버 | Claude Code / Copilot에 최적화된 경량 CLI/MCP 에이전트 |

- "내가 개발자/PM이고, AI 에이전트(Claude Code 등)를 시켜 티켓 정리하고 문서를 대신 쓰게 하고 싶다" > mcp-atlassian 없이 atlassian-skills만으로 200% 만족하며 업무 가능합니다.
- "나는 전사 Atlassian 인프라를 총괄하며 권한을 통제하고 자동화 파이프라인을 설계하는 관리자다" > atlassian-skills만으로는 부족하며, 공식 MCP나 Atlassian 웹 GUI를 병행하셔야 합니다.

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

## atlassian-skills 설치

`uv`가 공식 릴리즈된 바이너리를 자동으로 다운로드 하여 설치하므로 인터넷이 연결된 환경이 라면 별도로 소스나 바이너리를 직접 다운로드 할 필요가 없다.

```pwsh
# install atlassian-skills, path: ~\AppData\Roaming\uv\tools\atlassian-skills
uv tool install atlassian-skills
uv tool list
atls --version
```

만약 인터넷이 연결되지 않는 환경이라면, 별도로 다운로드 받아서 설치하는 방법을 고려해야 한다.

## 사용자 환경 변수에 ATLASSIAN 액세스 정보 등록

windows/linux에서 다음의 사용자 환경 변수를 등록하면 mcp-atlassian이 자동으로 접속 정보를 사용하게 된다.

Authentication 관련 환경변수는 atlassian cloud냐 Server 형이냐에 따라서 약간의 차이가 있으므로 공식문서를 참조한다.

- [Cloud](https://mcp-atlassian.soomiles.com/docs/authentication#api-token-cloud-recommended)
- [Server/DataCenter](https://mcp-atlassian.soomiles.com/docs/authentication#personal-access-token-server/data-center)

jira 또는 confluence에 로그인하여 프로필 메뉴에서 PAT(Personal Access Tokens)를 생성할 수 있다.

Jira와 Confluence의 token 생성 메늎:

- `프로파일 > 환경설정 > 개인용 액세스 토큰`

Bitbucket의 token 생성 메늎:

- `View Profile > Manage account > HTTP access tokens`

Bitbucket의 토큰 생성 시 permission 설정

```plain
Project permissions = Project read
Repository permissions = Repository write
```

다음은 `atlassian-skills` 용 windows 사용자 환경 변수 설정 예이다.

```pwsh
[System.Environment]::SetEnvironmentVariable('ATLS_DEFAULT_JIRA_URL', 'https://atlassian.net', 'User')
[System.Environment]::SetEnvironmentVariable('JIRA_PERSONAL_TOKEN', 'YOUR_ACTUAL_PERSONAL_TOKEN', 'User')
[System.Environment]::SetEnvironmentVariable('ATLS_DEFAULT_CONFLUENCE_URL', 'https://atlassian.net', 'User')
[System.Environment]::SetEnvironmentVariable('CONFLUENCE_PERSONAL_TOKEN', 'YOUR_ACTUAL_PERSONAL_TOKEN', 'User')
[System.Environment]::SetEnvironmentVariable('ATLS_DEFAULT_BITBUCKET_URL', 'https://atlassian.net', 'User')
[System.Environment]::SetEnvironmentVariable('BITBUCKET_TOKEN', 'YOUR_ACTUAL_PERSONAL_TOKEN', 'User')
```

사용자 환경 변수가 등록되면, 이를 사용하는 프로그램(터미널 등)은 새로 시작하여야 적용된다.

## 프로그램 설정

### 설정 상태 확인

```pwsh
atls doctor
```

모든 환경 설정과 skill 설치가 완료 되었다면 다음과 같은 출력이 나온다.

```pwsh
atls 0.4.3  (couldn't reach PyPI — update check skipped)

Platform: windows (shell: powershell)
  Attachment writer: native

Paths:
  Claude config dir   : C:\Users\user\.claude
  Claude skill target : C:\Users\user\.claude\skills\atls\SKILL.md
  CLAUDE.md path      : C:\Users\user\.claude\CLAUDE.md
  Codex config dir    : C:\Users\user\.codex
  Codex AGENTS.md path: C:\Users\user\.codex\AGENTS.md
  Codex skill target  : C:\Users\user\.codex\skills\atls\SKILL.md  (canonical)
  Copilot config dir  : C:\Users\user\.copilot
  Copilot skill target: C:\Users\user\.copilot\skills\atls\SKILL.md
  Copilot instructions: C:\Users\user\.copilot\copilot-instructions.md

Skill installation status:
  Claude skill: installed (v0.4.3)
  Codex skill: installed (v0.4.3)
  Copilot skill: installed (v0.4.3)
  Codex AGENTS.md: ATLS block v0.4.3
  CLAUDE.md: ATLS block v0.4.3
  Copilot instructions: ATLS block v0.4.3

Auth:
Profile: default
  Jira URL:         https://jira.atlassian.com  (env (ATLS_DEFAULT_JIRA_URL))
  Confluence URL:   https://confluence.atlassian.com  (env (ATLS_DEFAULT_CONFLUENCE_URL))
  Bitbucket URL:    https://repos.ahnlab.com  (env (ATLS_DEFAULT_BITBUCKET_URL))
  Auth method:    pat (jira) / pat (confluence) / pat (bitbucket)
  Storage:        env
  [jira] token: set (length=44, source=env)
  [confluence] token: set (length=44, source=env)
  [bitbucket] token: set (length=49, source=env)
  TLS verify:     system trust store (OS certificates, via truststore)
```

### 설정 프롬프트 실행

atlassian 접속 정보(URL/PAT), Skill 설치를 위한 명령 프롬프트를 제공

atlassian 접속 정보들은 환경 변수로 미리 설정한 경우 환경 변수를 사용할 것이라고 표시되므로, 주로 skill.md 설치에 사용한다.

```pwsh
atls setup
```

### atlassian 연동 확인

```pwsh
atls jira user me
atls confluence user me
atls bitbucket project list
```

### AI Client 명령

이후 ai 도구(claude 또는 copilot)에서 다음과 같이 명령하면 atlassian 에서 정보를 읽어와 응답을 한다.

- `나에게 할당된 미해결 Jira 목록을 알려줘`
- `Confluence 페이지 123456789 내용을 읽고 5줄로 요약해줘`
- `https://.../wiki/spaces/TEAM/pages/123456789/... 이 페이지 읽어서 액션 아이템만 뽑아줘`
- `Confluence space_key=TEAM, title="배포 가이드" 페이지 찾아서 핵심 절차 정리해줘`
