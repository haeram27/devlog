# GitHub Copilot CLI 명령 목록

기준: GitHub Copilot CLI **v1.0.51** 단독 설치형(`@github/copilot`)의 공식 도움말 출력.

## `copilot` 명령 전체 옵션

아래는 `copilot --help` 기준으로 확인되는 **top-level 옵션 전체 목록**

| 옵션 | 설명 |
| --- | --- |
| `--effort`, `--reasoning-effort <level>` | 추론 강도를 `none`, `low`, `medium`, `high`, `xhigh`, `max` 중에서 지정한다. |
| `--acp` | Agent Client Protocol 서버 모드로 시작한다. |
| `--add-dir <directory>` | 파일 접근 허용 디렉터리를 추가한다. 여러 번 지정 가능하다. |
| `--add-github-mcp-tool <tool>` | GitHub MCP 서버에서 기본 집합 대신 특정 도구를 추가 허용한다. |
| `--add-github-mcp-toolset <toolset>` | GitHub MCP 서버에서 특정 toolset을 추가 허용한다. `all`도 가능하다. |
| `--additional-mcp-config <json>` | 추가 MCP 서버 설정을 JSON 문자열 또는 `@파일경로`로 주입한다. |
| `--agent <agent>` | 사용할 커스텀 에이전트를 지정한다. |
| **`--allow-all`** | 모든 도구, 경로, URL 권한을 한 번에 허용한다. |
| `--allow-all-paths` | 파일 경로 검증을 끄고 모든 경로 접근을 허용한다. |
| `--allow-all-tools` | 모든 도구 실행을 확인 없이 허용한다. |
| `--allow-all-urls` | 모든 URL 접근을 확인 없이 허용한다. |
| `--allow-tool[=tools...]` | 지정한 도구들을 추가 승인 없이 사용할 수 있게 한다. |
| `--allow-url[=urls...]` | 지정한 URL 또는 도메인 접근을 허용한다. |
| `--attachment <path>` | 초기 프롬프트에 파일을 첨부한다. 비대화형 모드에서만 유효하다. |
| **`--autopilot`** | 시작부터 autopilot 모드로 진입한다. |
| `--available-tools[=tools...]` | 모델이 사용할 수 있는 도구를 지정한 목록으로 제한한다. |
| `--banner` | 시작 배너를 표시한다. |
| `--bash-env[=value]` | bash 셸에서 `BASH_ENV` 지원을 켜거나 끈다. |
| `-C <directory>` | 실행 전에 작업 디렉터리를 변경한다. |
| `--connect[=sessionId]` | 원격 세션에 직접 연결한다. 세션 ID나 작업 ID를 줄 수 있다. |
| `--continue` | 가장 최근 세션을 다시 연다. |
| `--deny-tool[=tools...]` | 지정한 도구를 사용 금지한다. |
| `--deny-url[=urls...]` | 지정한 URL 또는 도메인을 차단한다. `--allow-url`보다 우선한다. |
| `--disable-builtin-mcps` | 기본 내장 MCP 서버들을 비활성화한다. |
| `--disable-mcp-server <server-name>` | 특정 MCP 서버 하나를 비활성화한다. |
| `--disallow-temp-dir` | 시스템 임시 디렉터리에 대한 자동 접근을 막는다. |
| `--enable-all-github-mcp-tools` | GitHub MCP 서버의 모든 도구를 활성화한다. |
| `--enable-reasoning-summaries` | OpenAI 모델용 추론 요약을 요청한다. |
| `--excluded-tools[=tools...]` | 모델이 절대 사용할 수 없도록 제외할 도구를 지정한다. |
| `--experimental` | 실험 기능을 활성화한다. |
| `-h`, `--help` | 도움말을 표시한다. |
| `-i`, `--interactive <prompt>` | 대화형 모드로 시작하면서 첫 프롬프트를 즉시 실행한다. |
| `--log-dir <directory>` | 로그 파일 저장 디렉터리를 지정한다. |
| `--log-level <level>` | 로그 레벨을 `none`, `error`, `warning`, `info`, `debug`, `all`, `default` 중에서 지정한다. |
| `--max-autopilot-continues <count>` | autopilot 모드의 자동 continuation 최대 횟수를 지정한다. |
| `--mode <mode>` | 초기 에이전트 모드를 `interactive`, `plan`, `autopilot` 중에서 지정한다. |
| `--model <model>` | 사용할 AI 모델을 지정한다. |
| `--mouse[=value]` | alt screen 모드에서 마우스 지원을 켜거나 끈다. |
| `-n`, `--name <name>` | 새 세션 이름을 지정한다. |
| `--no-ask-user` | `ask_user` 도구를 비활성화해 질문 없이 자율 진행하게 한다. |
| `--no-auto-update` | 자동 업데이트 다운로드를 비활성화한다. |
| `--no-bash-env` | bash 셸에서 `BASH_ENV` 지원을 끈다. |
| `--no-color` | 모든 컬러 출력을 비활성화한다. |
| `--no-custom-instructions` | `AGENTS.md` 등 커스텀 지침 파일 로딩을 끈다. |
| `--no-experimental` | 실험 기능을 비활성화한다. |
| `--no-mouse` | alt screen 모드에서 마우스 지원을 끈다. |
| `--no-remote` | GitHub 웹/모바일의 원격 제어를 비활성화한다. |
| `--output-format <format>` | 출력 형식을 `text` 또는 `json`으로 지정한다. |
| **`-p`**, **`--prompt <text>`** | 비대화형 모드에서 프롬프트를 실행하고 종료한다. |
| `--plain-diff` | 리치 diff 렌더링을 끄고 일반 diff 출력만 사용한다. |
| `--plan` | 시작부터 plan 모드로 진입한다. |
| `--plugin-dir <directory>` | 로컬 디렉터리의 플러그인을 로드한다. 여러 번 지정 가능하다. |
| `--remote` | GitHub 웹/모바일에서 현재 세션 원격 제어를 허용한다. |
| `--resume[=value]` | 이전 세션을 재개한다. 세션 ID, 작업 ID, ID prefix, 이름을 줄 수 있다. |
| `-s`, `--silent` | 통계 없이 에이전트 응답만 출력한다. |
| `--screen-reader` | 스크린 리더 최적화를 활성화한다. |
| `--secret-env-vars[=vars...]` | 출력과 실행 환경에서 숨길 민감 환경변수 이름을 지정한다. |
| `--session-id <id>` | 기존 세션/작업을 해당 ID로 재개하거나 새 세션 UUID를 지정한다. |
| `--share[=path]` | 비대화형 완료 후 세션을 Markdown 파일로 저장한다. |
| `--share-gist` | 비대화형 완료 후 세션을 secret GitHub Gist로 공유한다. |
| `--stream <mode>` | 스트리밍 출력 사용을 `on` 또는 `off`로 지정한다. |
| `-v`, `--version` | 버전 정보를 표시한다. |
| **`--yolo`** | `--allow-all`의 별칭으로 모든 권한을 허용한다. |

## 대화형 세션(interaction mode)의 명령 목록

### 특수 입력 명령

| 명령 | 설명 |
| --- | --- |
| `/` | 사용 가능한 슬래시 명령 목록을 표시한다. |
| `@` | 파일을 멘션해 현재 대화 컨텍스트에 추가한다. |
| `#` | 이슈나 Pull Request를 멘션해 대화에 참조로 추가한다. |
| `!` | 셸 명령을 실행한다. |

### Agent Environment

| 명령 | 설명 |
| --- | --- |
| `/init` | 현재 저장소용 Copilot 지침 파일 구성을 초기화한다. |
| `/agent` | 사용 가능한 에이전트를 조회하고 선택한다. |
| `/skills` | 스킬 기능을 조회하고 관리한다. |
| `/mcp` | MCP 서버 설정을 관리한다. |
| `/plugin` | 플러그인과 플러그인 마켓플레이스를 관리한다. |

### Agents / Subagents

| 명령 | 설명 |
| --- | --- |
| `/model` | 현재 세션에서 사용할 AI 모델을 선택한다. |
| `/delegate` | 현재 작업을 GitHub 쪽으로 위임해 Copilot이 PR 생성을 진행하게 한다. |
| **`/fleet`** | 병렬 서브에이전트 실행용 fleet 모드를 켜거나 끈다. |
| `/tasks` | 서브에이전트와 셸 작업 목록을 조회하고 관리한다. |

### Code

| 명령 | 설명 |
| --- | --- |
| `/ide` | IDE 작업공간과 연결한다. |
| `/diff` | 현재 디렉터리의 변경 사항을 검토한다. |
| `/pr` | 현재 브랜치와 관련된 Pull Request 작업을 수행한다. |
| `/review` | 코드 리뷰 에이전트를 실행해 변경 사항을 분석한다. |
| `/lsp` | 언어 서버 설정을 관리한다. |
| `/terminal-setup` | 멀티라인 입력 지원을 위한 터미널 설정을 구성한다. |

### Permissions

| 명령 | 설명 |
| --- | --- |
| `/allow-all` | 도구, 경로, URL 권한을 모두 허용한다. |
| `/add-dir` | 파일 접근 허용 목록에 디렉터리를 추가한다. |
| `/list-dirs` | 현재 허용된 디렉터리 목록을 보여준다. |
| `/cwd` | 현재 작업 디렉터리를 확인하거나 변경한다. |
| `/reset-allowed-tools` | 허용된 도구 목록을 초기화한다. |

### Session

| 명령 | 설명 |
| --- | --- |
| `/resume` | 다른 세션으로 전환하거나 세션 ID/작업 ID/이름으로 재개한다. |
| `/rename` | 현재 세션 이름을 바꾸거나 대화 내용으로 자동 생성한다. |
| **`/context`** | 컨텍스트 윈도 사용량과 시각화를 보여준다. |
| `/usage` | 세션 사용량 통계와 메트릭을 보여준다. |
| `/session` | 세션 목록과 세부 정보를 조회하고 관리한다. |
| `/compact` | 대화 히스토리를 요약해 컨텍스트 사용량을 줄인다. |
| `/share` | 세션이나 리서치 결과를 Markdown, HTML, GitHub Gist로 공유한다. |
| `/remote` | GitHub 웹과 모바일에서의 원격 제어 기능을 켜거나 끈다. |
| `/copy` | 마지막 응답을 클립보드로 복사한다. |
| `/rewind` | 마지막 턴을 되감고 파일 변경도 함께 되돌린다. |

### Help

| 명령 | 설명 |
| --- | --- |
| `/help` | 대화형 명령 도움말을 보여준다. |
| `/changelog` | CLI 버전별 변경 이력을 보여주며 `summarize` 옵션으로 요약도 가능하다. |
| `/feedback` | CLI에 대한 피드백을 보낸다. |
| `/theme` | 색상 테마를 조회하거나 설정한다. |
| `/statusline` | 상태 줄 표시 항목을 설정한다. |
| `/footer` | 하단 상태 표시 항목을 설정한다. |
| `/update` | CLI를 최신 버전으로 업데이트한다. |
| `/version` | 버전 정보를 보여주고 업데이트 가능 여부를 확인한다. |
| `/experimental` | 실험 기능 목록을 보여주거나 실험 모드를 켜고 끈다. |
| `/memory` | 메모리 기능 상태를 확인하거나 세션 간 메모리 사용을 켜고 끈다. |
| `/clear` | 현재 세션을 버리고 새 세션으로 시작한다. |
| `/instructions` | 커스텀 지침 파일 상태를 조회하고 토글한다. |
| `/streamer-mode` | 모델명과 쿼터 정보 숨김용 streamer mode를 켜거나 끈다. |

### Other commands

| 명령 | 설명 |
| --- | --- |
| `/after` | 일정 시간이 지난 뒤 한 번 실행할 프롬프트나 스킬을 예약한다. |
| `/ask` | 대화 히스토리에 남기지 않는 짧은 보조 질문을 한다. |
| `/autopilot` | autopilot 모드를 켜거나 끈다. |
| `/chronicle` | 세션 히스토리와 관련 인사이트 기능을 사용한다. |
| `/env` | 로드된 지침, MCP, 스킬, 에이전트, 플러그인, LSP, 확장 정보를 보여준다. |
| `/every` | 일정 주기로 반복 실행할 프롬프트나 스킬을 예약한다. |
| `/exit` | CLI를 종료하며 필요하면 세션 내용을 출력하고 끝낸다. |
| `/keep-alive` | 시스템 절전 방지용 keep-alive 모드를 관리한다. |
| `/login` | GitHub Copilot에 로그인한다. |
| `/logout` | OAuth 로그인 세션에서 로그아웃한다. |
| `/new` | 새 대화를 시작한다. |
| `/plan` | 코딩 전에 구현 계획을 생성한다. |
| `/research` | GitHub 검색과 웹 소스를 활용한 심층 조사를 실행한다. |
| `/restart` | 현재 세션을 유지한 채 CLI를 재시작한다. |
| `/search` | 대화 타임라인을 검색한다. |
| `/sidekicks` | 실행 중인 sidekick 에이전트를 조회한다. |
| `/undo` | 마지막 턴과 파일 변경을 되돌리며 사실상 `/rewind`와 같은 역할을 한다. |
| `/user` | GitHub 사용자 목록을 관리한다. |

## 자동 진행 설정 하기

실무 기준 추천:

1. 기본: --autopilot만
1. --yolo: 로컬 실험/일회성 작업에서만 제한적으로
1. --no-ask-user: 사실상 비권장 (격리된 테스트 환경에서만)


### 대화형 세션 안에서 설정하기

```text
/autopilot
/allow-all
```

### cli 옵션으로 설정하기 (세션 전체)

```bash
copilot --autopilot --allow-all
```

또는 `--allow-all`과 같은 의미의 별칭인 `--yolo`를 써서 아래처럼 실행할 수 있다.

```bash
copilot --autopilot --yolo
```

추가로, 질문 없이 더 자율적으로 진행시키려면 다음 옵션도 함께 사용할 수 있다.

```bash
copilot --autopilot --yolo --no-ask-user
```

- `--autopilot`: 시작 시 autopilot 모드로 진입한다.
- `--allow-all`: 모든 도구, 경로, URL 권한을 한 번에 허용한다.
- `--yolo`: `--allow-all`의 별칭이다.
- `--no-ask-user`: `ask_user` 도구를 비활성화해 에이전트가 질문 없이 최대한 자율적으로 진행하게 한다.

실전 예시는 다음과 같다.

```bash
copilot --autopilot --yolo -i "중간 확인 없이 가능한 범위 끝까지 진행해"
```