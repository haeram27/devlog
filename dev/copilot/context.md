
# github-copilot을 사용하는 project 환경 설정

copilot이 동작하는데 참조해야할 context를 문서(markdown) 형식으로 미리 정의하여 copilot이 항상 동일한 context를 참조하도록 환경 구성

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