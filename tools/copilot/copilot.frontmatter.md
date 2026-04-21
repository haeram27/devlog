# Github Copilot FrontMatter

1. [Github Copilot FrontMatter](#github-copilot-frontmatter)
   1. [instructions.md](#instructionsmd)
   2. [applyTo](#applyto)
   3. [excludeAgent](#excludeagent)
      1. [1. excludeAgent의 동작 방식](#1-excludeagent의-동작-방식)
      2. [2. 두 에이전트 모두에서 제외하고 싶은 경우](#2-두-에이전트-모두에서-제외하고-싶은-경우)
      3. [요약 표](#요약-표)
   4. [tools](#tools)
      1. [1. 주요 역할](#1-주요-역할)
      2. [2. 메타정보 헤더 작성 예시 (YAML 형식)](#2-메타정보-헤더-작성-예시-yaml-형식)
      3. [3. 자주 사용되는 주요 도구(Tools) 종류](#3-자주-사용되는-주요-도구tools-종류)
      4. [4. 작동 방식](#4-작동-방식)

---

## instructions.md

|Field |Required |Description|
|---|---|---|
|name |No |Display name shown in the UI. Defaults to the file name.
|description |No |Short description shown on hover in the Chat view.
|applyTo |No |Glob pattern that defines which files the instructions apply to automatically, relative to the workspace root. Use *- to apply to all files. If not specified, the instructions are not applied automatically, but you can still add them manually to a chat request.|
|tools |No |List of tools that the agent can use.|

- 필요시 tools 항목 명시 가능

```yaml
---
name: 'Python Standards'
description: 'Coding conventions for Python files'
applyTo: '**/*.py'
tools: ['semantic-search', 'web-search']
excludeAgent: "code-review"
excludeAgent: "coding-agent"
---
```

## applyTo

```yaml
---
applyTo: "app/models/**/*.rb,**/*.ts,**/*.tsx"
---
```

- comma(,)로 여러 패턴 구분 지정 가능
- GLOB 패턴으로 파일 또는 경로 명시
  - `*` - 현재 디렉터리의 모든 파일과 매칭 됩니다.
  - `**` 또는 `**/*` - 모든 디렉터리에 있는 모든 파일과 매칭 됩니다.
  - `*.py` - 현재 디렉터리의 모든 `.py` 파일과 매칭 됩니다.
  - `**/*.py` - 모든 디렉터리에 있는 모든 `.py` 파일과 매칭 됩니다.
  - `src/*.py` - `.py` 디렉터리 내 모든 `src` 파일을 일치시킵니다. 예를 들어, `src/foo.py` 와 `src/bar.py` 에 매칭 되지만 `src/foo/bar.py` 에는 매칭 되지 않습니다.
  - `src/**/*.py` - 디렉터리의 모든 `.py` 파일 `src` 과 재귀적으로 일치합니다. 예: `src/foo.py`, `src/foo/bar.py`및 `src/foo/bar/baz.py` 와 매칭 됩니다.
  - `**/subdir/**/*.py` - 모든 `.py` 디렉터리의 깊은 곳에 있는 모든 `subdir` 파일을 재귀적으로 일치시킵니다. 예를 들어 `subdir/foo.py`, `subdir/nested/bar.py`, `parent/subdir/baz.py`, `deep/parent/subdir/nested/qux.py`와 같은 경우에는 사용되지만, _디렉터리_`foo.py`가 포함되지 않은 `subdir` 경로에서는 사용되지 않습니다.

## excludeAgent

이 `instructions.md` 파일이 특정 agent의해 참조되는 것을 막으려면 `excludeAgent`에 에이전트의 이름을 지정합니다.

```yaml
---
excludeAgent: "code-review"
excludeAgent: "coding-agent"
---
```

excludeAgent 속성은 현재 한 번에 하나의 값만 지정할 수 있도록 설계되어 있습니다. 따라서 "code-review"와 "cloud-agent" 둘 모두를 동시에 명시하여 제외하는 단일 설정은 지원되지 않습니다.

### 1. excludeAgent의 동작 방식

- 단일 값 할당: 프론트매터(frontmatter)에서 `excludeAgent`에는 단일 agent만 지정 가능합니다.
- 기본 동작: 지정된 이름의 agent는 이 지침을 사용하지 않습니다. 이 키워드를 명시하지 않으면 모든 agent에서 이 지침 사용이 가능 합니다.

### 2. 두 에이전트 모두에서 제외하고 싶은 경우

만약 특정 지침 파일을 두 에이전트 모두에게 숨기고 싶다면, 현재의 excludeAgent 옵션으로는 직접적인 해결이 어렵습니다. 대신 다음과 같은 대안을 고려할 수 있습니다:

- 파일 분리: 특정 에이전트별로 전용 지침 파일을 각각 생성하여 관리합니다.
  - 파일 A: excludeAgent: "code-review" (클라우드 에이전트만 사용)
  - 파일 B: excludeAgent: "coding-agent" (코드 리뷰만 사용)
- 설정 페이지 활용: [GitHub 리포지토리 설정 페이지](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/configuring-agent-settings)에서 각 에이전트의 활성화 여부나 세부 기능을 직접 끄는 방법이 더 확실할 수 있습니다.

### 요약 표

| 설정 값 | 효과 |
|---|---|
| 미지정 | 모든 에이전트(리뷰, 클라우드)가 지침을 사용함 |
| "code-review" | Copilot code review가 이 지침을 무시함 |
| "coding-agent" | Copilot cloud agent가 이 지침을 무시함 |

## tools

agent.md 파일의 tools 항목은 에이전트가 단순히 텍스트 답변을 생성하는 것을 넘어, 실제로 특정 동작을 수행하거나 외부 정보를 가져올 수 있도록 허용된 권한(함수 호출) 목록을 정의하는 곳입니다.
쉽게 말해, 에이전트에게 "너는 이 도구들을 손에 들고 직접 사용할 수 있어"라고 명령하는 것과 같습니다.

### 1. 주요 역할

- 실행 권한 부여: 에이전트가 파일을 읽거나, 터미널 명령어를 실행하거나, 웹 검색을 할 수 있는 권한을 명시합니다.
- 워크플로우 자동화: 에이전트가 스스로 "이 문제를 해결하려면 먼저 이 도구를 써야겠군"이라고 판단하여 단계를 수행합니다.

### 2. 메타정보 헤더 작성 예시 (YAML 형식)

.agent.md 파일 상단의 프론트매터(Frontmatter)에 다음과 같이 정의합니다.

```md
---
name: dev-assistant
description: 개발 및 인프라 관리 지원 에이전트
tools:
  - shell_execute          # 터미널 명령어 실행 권한
  - file_read             # 프로젝트 파일 읽기 권한
  - file_write            # 파일 생성 및 수정 권한
  - google_search         # (지원 시) 외부 정보 검색 권한
  - codebase_search       # 전체 코드베이스 검색 권한
---

# 에이전트 지침
너는 코드 수정 후 반드시 `shell_execute`를 통해 테스트를 실행해야 해.
```

### 3. 자주 사용되는 주요 도구(Tools) 종류

GitHub Copilot 및 관련 에이전트 시스템에서 지원하는 대표적인 도구들입니다.

- shell_execute (또는 terminal): npm test, ls, git status 같은 명령어를 터미널에서 직접 실행합니다.
- read_file / write_file: 특정 파일의 내용을 읽거나 수정 사항을 저장합니다.
- list_dir: 디렉토리 구조를 확인합니다.
- fetch_url: 특정 웹 페이지의 내용을 긁어옵니다.
- mcp (Model Context Protocol): 외부 서비스(Slack, Jira, Database 등)와 연결된 사용자 정의 도구를 호출할 때 사용합니다.

### 4. 작동 방식

    1. 사용자 질문: "프로젝트의 모든 테스트를 돌려줘."
    2. 에이전트 판단: "내 tools 목록에 shell_execute가 있네? 이걸로 테스트 명령어를 실행하자."
    3. 실행: 에이전트가 내부적으로 도구를 호출하여 결과를 가져옵니다.
    4. 결과 보고: "테스트 결과 5개가 성공하고 1개가 실패했습니다."라고 답변합니다.

주의사항: 보안을 위해 에이전트가 도구(특히 shell_execute)를 사용할 때 사용자의 승인을 거치도록 설정되어 있는 경우가 많습니다.
