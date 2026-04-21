# GitHub Copilot instructions.md 작성법

GitHub Copilot instructions.md 파일은 에이전트에게 특정한 지침이나 규칙을 전달하기 위한 문서입니다. 이 파일은 Markdown 형식으로 작성되며, YAML 프론트매터(Frontmatter)를 사용하여 메타데이터를 정의할 수 있습니다. instructions.md 파일은 에이전트가 어떻게 행동해야 하는지, 어떤 규칙을 따라야 하는지 등을 명확하게 전달하는 데 사용됩니다.

- links
  - [VS Code Custom Instructions](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
  - [GitHub Copilot Configure Custom Instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/)


## YAML 프론트매터(Frontmatter)
instructions.md 파일의 상단에는 YAML 형식의 프론트매터가 위치합니다. 이 부분에서는 다음과 같은 메타데이터를 정의할 수 있습니다.

|Field |Required |Description|
|---|---|---|
|name |No |Display name shown in the UI. Defaults to the file name.
|description |No |Short description shown on hover in the Chat view.
|applyTo |No |Glob pattern that defines which files the instructions apply to automatically, relative to the workspace root. Use ** to apply to all files. If not specified, the instructions are not applied automatically, but you can still add them manually to a chat request.|
|tools |No |List of tools that the agent can use.|


### tools
agent.md 파일의 tools 항목은 에이전트가 단순히 텍스트 답변을 생성하는 것을 넘어, 실제로 특정 동작을 수행하거나 외부 정보를 가져올 수 있도록 허용된 권한(함수 호출) 목록을 정의하는 곳입니다.
쉽게 말해, 에이전트에게 "너는 이 도구들을 손에 들고 직접 사용할 수 있어"라고 명령하는 것과 같습니다.

#### 1. 주요 역할

* 실행 권한 부여: 에이전트가 파일을 읽거나, 터미널 명령어를 실행하거나, 웹 검색을 할 수 있는 권한을 명시합니다.
* 워크플로우 자동화: 에이전트가 스스로 "이 문제를 해결하려면 먼저 이 도구를 써야겠군"이라고 판단하여 단계를 수행합니다.

#### 2. 메타정보 헤더 작성 예시 (YAML 형식)
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


#### 3. 자주 사용되는 주요 도구(Tools) 종류
GitHub Copilot 및 관련 에이전트 시스템에서 지원하는 대표적인 도구들입니다.

* shell_execute (또는 terminal): npm test, ls, git status 같은 명령어를 터미널에서 직접 실행합니다.
* read_file / write_file: 특정 파일의 내용을 읽거나 수정 사항을 저장합니다.
* list_dir: 디렉토리 구조를 확인합니다.
* fetch_url: 특정 웹 페이지의 내용을 긁어옵니다.
* mcp (Model Context Protocol): 외부 서비스(Slack, Jira, Database 등)와 연결된 사용자 정의 도구를 호출할 때 사용합니다.

#### 4. 작동 방식

    1. 사용자 질문: "프로젝트의 모든 테스트를 돌려줘."
    2. 에이전트 판단: "내 tools 목록에 shell_execute가 있네? 이걸로 테스트 명령어를 실행하자."
    3. 실행: 에이전트가 내부적으로 도구를 호출하여 결과를 가져옵니다.
    4. 결과 보고: "테스트 결과 5개가 성공하고 1개가 실패했습니다."라고 답변합니다.

주의사항: 보안을 위해 에이전트가 도구(특히 shell_execute)를 사용할 때 사용자의 승인을 거치도록 설정되어 있는 경우가 많습니다.

