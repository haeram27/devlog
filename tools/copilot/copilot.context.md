
# github-copilot을 사용하는 project 환경 설정

- 참고: [github-copilot-customization](https://docs.github.com/copilot/tutorials/customization-library)
- github copilot은 커스텀 context를 지원함
- copilot이 동작하는데 참조해야할 context를 문서(markdown) 형식으로 미리 정의하여 copilot이 동작시 이 context를 참조하도록 설정 해 줌
- 대표적인 커스텀 context는 아래 3가지
  - Custom Instructions
  - Prompt Files
  - Custom agents

## .github 디렉토리 표준 파일 위치

* **{repo-root}/AGENTS.md**
  * 역할: Copilot 에이전트(코드 분석, 빌드, 테스트 등)에서 참조 되는 공용 지침 파일입니다.
  * 명령 쿼리에 참조 방식: 항상 자동 포함
* **{repo-root}/CLAUDE.md**
  * 역할: 에이전트 지침 파일
  * 명령 쿼리에 참조 방식: 조건부 자동 포함, Anthropic's Claude model 사용시
* **{repo-root}/GEMINI.md**
  * 역할: 에이전트 지침 파일
  * 명령 쿼리에 참조 방식: 조건부 자동 포함, Google Gemini model 사용시
* **{repo-root}/.github/copilot-instructions.md**
  * 역할: 리포지토리 전체에 적용되는 전역 지침입니다. 모든 채팅 및 인라인 요청에 자동으로 포함됩니다.
  * 참조 방식: 항상 자동 포함
* **{repo-root}/.github/agents/*.agent.md**
  * 역할: 특정 페르소나와 도구(Tools) 권한을 가진 사용자 정의 에이전트를 정의합니다.
  * 명령 쿼리에 참조 방식: 수동(사용자 호출), 채팅에서 `@에이전트명`으로 명시적으로 호출하여 사용
* **{repo-root}/.github/instructions/*.instructions.md**
  * 역할: 경로별(Path-specific) 지침 파일입니다. 파일 상단에 applyTo 패턴을 작성하여 특정 폴더나 파일 형식 작업 시에만 컨텍스트에 포함되도록 설정합니다.
  * 명령 쿼리에 참조 방식: 조건부 자동(file context의 경로 패턴), 채팅창에서 `current file context`나 `#`으로 추가한 `file context`의 경로나 파일 이름이 `instructions.md`의 `applyTo` 항목과 매칭 되면 참조됨
    ```md
    # java.instructions.md
    ---
    name: Java Coding Standards
    description: Java 프로젝트를 위한 코딩 규칙
    applyTo:
    - "src/main/java/**/*.java"  # 특정 경로의 Java 파일에만 적용
    - "**/*.gradle"              # Gradle 설정 파일에도 적용 가능
    ---
    # 여기에 Java 관련 지침 작성
    ```

* **{repo-root}/.github/prompts/*.prompt.md**
  * 역할: 자주 쓰는 복잡한 프롬프트를 템플릿화한 파일입니다.
  * 작성 내용: 사용자가 자주 사용하는 혹은 복잡한 작업 요청을 템플릿으로 작성
  * 명령 쿼리에 참조 방식: 수동(사용자 호출), 채팅 창에서 `/프롬프트명`으로 명시적으로 호출하여 사용
* **{repo-root}/.github/skills/skill-name/SKILL.md**
  * 역할: 에이전트가 수행할 수 있는 특정 기술(Skill)이나 워크플로 지침을 정의합니다.
  * 작성 내용: 에이전트가 수행 가능한 기능을 정의함. 기능의 목표 및 수행하기 위한 작업 순서 내용 및 커맨드 라인 명령어 등을 상세 작성
  * 명령 쿼리에 참조 방식: agent.md 에서 참조, agent.md의 파일 상단에 메타 정보로서 `skills:`에 경로를 명시
    ```md
    # code-reviewer.agent.md
    ---
    name: code-reviewer
    description: 코드 리뷰 전문 에이전트
    # 해당 에이전트가 참조할 스킬들을 명시
    skills:
    - .github/skills/refactoring.skill.md
    - .github/skills/security-check.skill.md
    ---
    ```

## .github 디렉토리 예제

```text
project/
└── .github/
    ├── README.md                               # .github 디렉토리 설명
    ├── copilot-instructions.md                 # repository 전체에 적용되는 전역 지침 파일
    ├── code-style-guide.md                     # 전체 코드 스타일 가이드 (AI 도구 통합 가이드 포함)
    ├── commit-message.style.md                 # 커밋 메시지 스타일 가이드
    ├── security-patterns.md                    # 보안 규칙
    ├── agents/                                 # Custom Agent 설정 파일들 (역할별 페르소나 및 특수기능 활성화)
    │   ├── janitor.agent.md                    # 코드 정리 모드 (기술 부채 제거)
    │   ├── plan.agent.md                       # 계획 수립 모드
    │   └── prd.agent.md                        # PRD 문서 생성 모드
    ├── instructions/                           # Cumstom Instrunction 설정 파일들, 모든 프롬프트에 자동 적용되는 context
    │   ├── security-patterns.instructions.md   # 보안 코딩 패턴 가이드
    │   ├── error-handling.instructions.md      # 에러 처리 표준 패턴
    │   └── markdown.instructions.md            # 마크다운 문서 작성 표준
    └── prompts/                                # 사전 정의된 prompt, 채팅 창에서 '/' 단축키로 호출
        ├── function-generation.prompt.md       # 함수 생성용 프롬프트
        ├── code_review.prompt.md               # 코드 리뷰용 프롬프트
        ├── plan.prompt.md                      # 기능 개발 계획 수립용 프롬프트
        ├── analyze.prompt.md                   # 클래스 분석 및 문서 생성용 프롬프트
        └── make_gtest.prompt.md                # GTest 생성용 프롬프트
```

## copilot이 참조하는 설정 디렉토리

### agents (custom agent)

- AI의 페르소나 설정 (너는 어떤 역할 이다)
- 사용자와의 대화 방식 및 답변 스타일과 내용 등을 설정 한다.
- 사용자의 질의 목표(분석, 자문, 계획, 구현, 정리 등)에 따라 AI 에게 맡기고 싶은 역할을 지정하는데 사용함
- Copilot Chat이 동작하는 환경/컨텍스트 모드를 말합니다.
  - Editor Mode: 코드 편집기 안에서 Copilot Chat을 사용해 코드 관련 질문을 할 때.
  - Terminal Mode: 터미널에서 명령어 관련 도움을 받을 때.
  - Docs Mode: 문서나 주석을 작성할 때.
- 모드에 따라 Copilot이 제공하는 답변 스타일과 기능이 달라지며, Editor Mode는 코드 제안에 집중하고, Docs Mode는 설명과 문서화에 집중합니다.

### instructions

- 행동 지침(방식)을 정의, How to do
- 항상 모든 프롬프트에서 참조됨
- Copilot에게 행동 지침을 주는 설정 또는 명령입니다.
  - “코드를 TypeScript로 작성해줘”
  - “함수에 JSDoc 주석을 추가해”
- Instructions는 Copilot이 답변을 생성할 때 전반적인 톤, 언어, 형식을 결정하는 데 사용됩니다.

### prompts

- 무엇을 할지를 정의, What to do
- 자주 쓰는 프롬프트 모음
- Copilot에게 구체적인 요청을 전달하는 입력 텍스트입니다.
  - “이 함수의 성능을 개선해줘”
  - “Python으로 파일 읽는 예제 코드 작성”
- Prompt는 Copilot이 실제로 무엇을 해야 하는지를 가이드하는 핵심 context 입니다.

### skills

- 프로젝트 스킬 (팀 공유): `.github/skills/`
- 개인 스킬 (로컬 전용): `~/.copilot/skills/`

## context 문서의 종류

- instruction
- prompt
- skill

### Instruction(행동지침)

- copilot이나 AI에게 "항상 따라야 하는 규칙" 또는 "행동 원칙"을 제공합니다.
- 예시: 코딩 스타일, 폴더 구조, 테스트 방식, 커밋 메시지 규칙 등 프로젝트 전반에 적용되는 가이드라인입니다.
- 항상 모든 프롬프트에서 참조됨

- 장점
  - 별도 호출 없이 항상 자동 적용되므로 팀 전체 코딩 규칙을 일관성 있게 강제 가능
  - applyTo 속성으로 파일 유형별 규칙을 세분화 적용 가능
  - 가장 오래된 기능으로 레퍼런스와 예시가 풍부함

- 단점
  - 매 대화마다 Context에 포함되므로 내용이 길어지면 토큰 소모 증가
  - 스크립트나 리소스 파일 포함 불가. 지시사항 텍스트만 작성 가능
  - VS Code, GitHub.com 외 다른 AI 도구에서는 동작하지 않음

### Prompt(프롬프트)

- copilot에게 "지금 이 순간, 특정 작업을 요청"하는 질문이나 명령입니다.
- 예시: "이 함수에 대한 테스트 코드를 작성해줘", "이 코드의 버그를 찾아줘" 등 구체적인 요청입니다.
- 자주 사용하는 prompt를 미리 정의하여 `.github/prompts/` 경로에 작성해 두면 채팅창에서 `/`를 이용하여 간단히 호출 할 수 있습니다.
- 각 prompot 별로 markdown 문서를 별도 작성하여 `.github/prompts/`에 넣어 둡니다.

- 장점
  - 슬래시 명령어로 간단히 호출할 수 있어 반복 작업 자동화에 적합
  - agent, model, tools 필드로 실행 환경을 세밀하게 제어 가능
  - 기존 대화 세션을 그대로 재사용 가능한 프롬프트로 변환 가능

- 단점
  - 수동 호출이 필수. 자동 적용이 안 되므로 팀 규칙 강제 용도로는 부적합
  - VS Code 전용으로 다른 AI 도구에서는 동작하지 않음
  - Agent Skills처럼 외부 스크립트나 리소스 파일을 직접 묶어서 관리하기 어려움

### Skill (스킬)

Copilot에게 "이런 상황에서는 이렇게 해"라고 미리 정해둔 작업 레시피다.
한 번 만들어두면 VS Code, Copilot CLI, Coding Agent 등에서 자동으로 재사용된다.

- 장점
  - 스크립트, 예시 파일 등 리소스를 함께 묶어 복잡한 워크플로우 구성 가능
  - VS Code, CLI, Coding Agent 등 여러 도구에서 재사용 가능한 오픈 표준
  - 필요할 때만 Context에 로드되므로 토큰 낭비 없음

- 단점
  - SKILL.md 외 폴더 구조까지 구성해야 하므로 초기 세팅 비용이 있음
  - Custom Instructions나 Prompt Files에 비해 상대적으로 신규 기능이라 레퍼런스가 적음

### 정리

- Instruction은 copilot의 "행동 기준"이고, prompt는 "즉각적인 작업 요청"입니다.
- copilot은 항상 instruction을 참고하며, prompt에 따라 구체적인 작업을 수행합니다.

| 항목 | skill | instruction | prompt |
|-------|--------------------------------|-------------------------------------------------------------------------|-----------------------------|
| 파일 위치 | .github/skills/<name>/SKILL.md | .github/copilot-instructions.md,<br>.github/instructions/*.instructions.md | .github/prompts/*.prompt.md |
| 목적    | 특수 작업·워크플로우 부여 | 코딩 규칙·가이드라인 정의 | 반복 사용 가능한 단일 작업 프롬프트        |
| 사용 범위 | VS Code, CLI, Coding Agent     | VS Code, GitHub.com | VS Code |
| 포함 내용 | 지시사항 + 스크립트 + 예시 파일 | 지시사항만 | 지시사항 + 참조 파일                |
| 로딩 방식 | 필요할 때만 (on-demand) | 항상 적용 | 호출 시에만 적용 |
| 재사용성  | 높음. 여러 AI 도구에서 공통 사용 가능 | 중간. VS Code·GitHub.com 한정 | 중간. VS Code 한정              |

## context 문서별 작성 방법

### root instruction 정의

- copilot 규칙 선언
- always/never 구조 활용
- 위치: `.github/copilot-instruction.md`

### 개별 instruction 정의

개별 파일 또는 영역에 대한 별도의 instruction (규칙) 작성

- add context > instructions

### prompt

자주 사용하는 작업을 명시
prompt에 작성된 내용은 챗봇 창에서 '/' 단축 prompt를 사용하여 호출 가능

## 참고 사이트

### 프롬프트 엔지니어링 가이드

- [github - copilot tutorial](https://docs.github.com/ko/copilot)
- [github - prompt-engineering](https://docs.github.com/en/copilot/concepts/prompting/prompt-engineering)

### MS 개발자처럼 설정하기

- [vscode 코드 프롬프트](https://github.com/microsoft/vscode/blob/main/.github/copilot-instructions.md)

- [vscode docs 프롬프트](https://github.com/microsoft/vscode-docs/blob/main/.github/copilot-instructions.md)

### context 문서 커뮤니티

- [awesome github copilot - copilot context 문서 공유 커뮤니티](https://github.com/github/awesome-copilot)

### 기타

- [copoilot 강의] (https://scopelabs.notion.site/Copilot-Github-264acc5ec8ff8082a79cd6217d025890)