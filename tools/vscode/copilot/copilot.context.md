
# github-copilot을 사용하는 project 환경 설정

copilot이 동작하는데 참조해야할 context를 문서(markdown) 형식으로 미리 정의하여 copilot이 항상 동일한 context를 참조하도록 환경 구성

## .github 디렉토리

```text
project/
└── .github/
    ├── README.md                            # .github 디렉토리 설명
    ├── copilot-instructions.md              # GitHub Copilot 코딩 가이드라인
    ├── code-style-guide.md                  # 전체 코드 스타일 가이드 (AI 도구 통합 가이드 포함)
    ├── commit-message.style.md              # 커밋 메시지 스타일 가이드
    ├── security-patterns.md                 # 보안 규칙
    ├── chatmodes/                           # AI 챗봇 모드별 설정 파일 (역할별 페르소나)
    │   ├── explainer.chatmode.md            # 기술 설명 모드 (개념/코드 설명)
    │   ├── janitor.chatmode.md              # 코드 정리 모드 (기술 부채 제거)
    │   ├── mentor.chatmode.md               # 멘토링 모드 (가이드 및 조언)
    │   ├── plan.chatmode.md                 # 계획 수립 모드
    │   ├── prd.chatmode.md                  # PRD 문서 생성 모드
    │   ├── specification.chatmode.md        # 사양 문서 생성 모드
    │   └── Ultimate-Transparent-Thinking-Beast-Mode.chatmode.md  # 고급 분석 모드
    ├── instructions/                        # AI 도구별 세부 지시사항
    │   ├── security-patterns.md             # 보안 코딩 패턴 가이드
    │   ├── error-handling.md                # 에러 처리 표준 패턴
    │   └── markdown.instruction.md          # 마크다운 문서 작성 표준
    └── prompts/                             # AI 도구용 프롬프트 템플릿
        ├── function-generation.prompt.md    # 함수 생성용 프롬프트
        ├── code_review.prompt.md            # 코드 리뷰용 프롬프트
        ├── plan.prompt.md                   # 기능 개발 계획 수립용 프롬프트
        ├── analyze.prompt.md                # 클래스 분석 및 문서 생성용 프롬프트
        └── make_gtest.prompt.md             # GTest 생성용 프롬프트
```

## Modes

### Chat Modes

- Copilot Chat이 동작하는 환경/컨텍스트 모드를 말합니다.
  - Editor Mode: 코드 편집기 안에서 Copilot Chat을 사용해 코드 관련 질문을 할 때.
  - Terminal Mode: 터미널에서 명령어 관련 도움을 받을 때.
  - Docs Mode: 문서나 주석을 작성할 때.
- 모드에 따라 Copilot이 제공하는 답변 스타일과 기능이 달라지며, Editor Mode는 코드 제안에 집중하고, - Docs Mode는 설명과 문서화에 집중합니다.

### Instructions

- Copilot에게 행동 지침을 주는 설정 또는 명령입니다.
  - “코드를 TypeScript로 작성해줘”
  - “함수에 JSDoc 주석을 추가해”
- Instructions는 Copilot이 답변을 생성할 때 전반적인 톤, 언어, 형식을 결정하는 데 사용됩니다.
- 어떻게 답변할지를 정의하는 역할을 수행합니다.

### Prompts

- Copilot에게 구체적인 요청을 전달하는 입력 텍스트입니다.
  - “이 함수의 성능을 개선해줘”
  - “Python으로 파일 읽는 예제 코드 작성”
- Prompt는 Copilot이 실제로 무엇을 해야 하는지를 이해하는 핵심 입력입니다.
- 무엇을 할지를 정의하는 역할

## context 문서의 종류

- instruction
- prompt

### Instruction(지침)

- copilot이나 AI에게 "항상 따라야 하는 규칙" 또는 "행동 원칙"을 제공합니다.
- 예시: 코딩 스타일, 폴더 구조, 테스트 방식, 커밋 메시지 규칙 등 프로젝트 전반에 적용되는 가이드라인입니다.
- 한 번 설정하면 여러 작업에 반복적으로 적용됩니다.

### Prompt(프롬프트)

- copilot에게 "지금 이 순간, 특정 작업을 요청"하는 질문이나 명령입니다.
- 예시: "이 함수에 대한 테스트 코드를 작성해줘", "이 코드의 버그를 찾아줘" 등 구체적인 요청입니다.
각 작업마다 다르게 입력됩니다.

### 정리

- Instruction은 copilot의 "행동 기준"이고, prompt는 "즉각적인 작업 요청"입니다.
- copilot은 항상 instruction을 참고하며, prompt에 따라 구체적인 작업을 수행합니다.

## 문서별 작성 방법

### root instruction 정의

- copilot 규칙 선언
- always/never 구조 활용
- 위치: `.github/copilot-instruction.md`

### 개별 instruction 정의

개별 파일 또는 영역에 대한 별도의 instruction (규칙) 작성

- add context > instructions

### prompt

자주 사용하는 작업을 명시
prompt에 작성된 내용은 챗봇 창에서 '/'를 이용하여 로드 가능

### MS 개발자처럼 설정하기

[vscode 코드 프롬프트](https://github.com/microsoft/vscode/blob/main/.github/copilot-instructions.md)

[vscode docs 프롬프트](https://github.com/microsoft/vscode-docs/blob/main/.github/copilot-instructions.md)