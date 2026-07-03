# omo agent별 model 변경 가이드

Oh My OpenCode(OMO)에서 각 AI 에이전트별로 LLM 모델을 변경하는 방법은 ① 실시간 명령어 구성과 ② 설정 파일(oh-my-opencode.json) 수정 두 가지가 있습니다.

프로젝트의 규모와 비용(토큰 사용량)에 따라 적절한 모델을 매칭하는 것이 핵심입니다.

## 방법 1. TUI 실시간 명령어로 변경

OpenCode 실행 중 특정 에이전트의 흐름이 막히거나 비용을 아끼고 싶을 때, 채팅창에 직관적인 슬래시(/) 명령어로 즉시 변경할 수 있습니다.

- 기본 사용법: /model [에이전트명] [프로바이더]/[모델명]
- 입력 예시:
  - 메인 오케스트레이터(sisyphus)를 최신 Claude 모델로 변경

    ```text
    /model sisyphus anthropic/claude-3-5-sonnet-20241022
    ```

  - 탐색 에이전트(explore)를 가성비 좋은 Gemini Flash 모델로 변경

    ```text
    /model explore google/gemini-2.5-flash
    ```

## 방법 2. 설정 파일(oh-my-opencode.json)로 영구 변경 (추천)

에이전트별 역할을 고정하여 항상 특정 모델로 구동되도록 설정하는 가장 확실한 방법입니다.

### 1. 설정 파일 열기

터미널 환경에 따라 아래 경로의 파일을 텍스트 에디터(VS Code, Vim 등)로 엽니다. 

- 글로벌 경로: ~/.config/opencode/oh-my-opencode.json (또는 프로젝트 루트 폴더 내) 

### 2. 에이전트별 agents 블록 수정

아래 예시처럼 agents 오브젝트 안에 각 에이전트 이름과 매핑할 model을 명시해 줍니다. 

```json
{
  "agents": {
    "sisyphus": { "model": "anthropic/claude-3-5-sonnet-20241022" },
    "oracle": { "model": "openai/gpt-4o" },
    "librarian": { "model": "google/gemini-2.5-flash" },
    "explore": { "model": "google/gemini-2.5-flash" }
  }
}
```

(※ 주의: JSON 파일 내에 모델을 작성하더라도, 해당 프로바이더의 API 키가 오픈코드 내에 /connect로 인증되어 있어야 정상 작동합니다.) 

## 오케스트레이션 모델 가치 효율적 배치 팁

OMO는 여러 에이전트가 동시에 협업하므로 똑똑한 모델(Claude, GPT-4o 등)만 도배하면 토큰 비용 폭탄을 맞을 수 있습니다.

- Sisyphus(전체 통제) / oracle(설계 및 디버깅): 복잡한 컨텍스트 해석과 정교한 판단 능력이 필요하므로 Claude 3.5 Sonnet 혹은 GPT-4o 계열의 플래그십 모델을 권장합니다.
- librarian(리서치) / explore(코드 탐색): 긴 레거시 코드를 빠르게 훑거나 외부 문서를 탐색하는 단순 반복 작업이 많으므로 context window가 길고 가격이 저렴한 Gemini Flash 라인업을 배치하는 것이 비용 절감에 절대적으로 유리합니다.

현재 OpenAI 계정의 GPT-4o/5 계열 위주로 세팅하고 싶으신가요, 아니면 Anthropic의 Claude 계열 위주로 팀을 구성하고 싶으신가요? 보유하신 API 키에 맞춤형 JSON 설정 템플릿을 바로 만들어 드릴 수 있습니다.

## 가성비 극대화 설정

제시해주신 가용 모델 목록을 바탕으로, 에이전트별 성능 효율을 극대화하면서 토큰 비용을 최소화(가성비 세팅)할 수 있도록 매칭한 oh-my-opencode.json 설정 파일입니다. [1, 2] 
최상위 플래그십 모델(Opus 4.8, GPT-5.4 등)은 가격이 매우 비싸므로, 가격 대비 코딩 지표가 우수한 최신 메인스트림 모델과 경량화(mini/Haiku) 모델을 조합하는 라우터 하이브리드 전략을 적용했습니다. [3, 4] 

### 1. 가성비 극대화 설정 파일 (oh-my-opencode.json)

가용 모델:

```
Claude Opus 4.8
Claude Opus 4.7
Claude Opus 4.6
Claude Opus 4.5
Claude Sonnet 5
Claude Sonnet 4.6
Claude Sonnet 4.5
Claude Haiku 4.5
GPT-5.4
GPT-5.4 mini
GPT-5.3-Codex
GPT-5 mini
Gemini 3.1 Pro
```

```json
{
  "agents": {
    "sisyphus": {
      "model": "anthropic/claude-sonnet-4.6"
    },
    "oracle": {
      "model": "openai/gpt-5.3-codex"
    },
    "librarian": {
      "model": "anthropic/claude-haiku-4.5"
    },
    "explore": {
      "model": "openai/gpt-5.4-mini"
    }
  }
}
```

### 2. 에이전트별 모델 선정 이유 (비용 절감 핵심)

- Sisyphus (메인 오케스트레이터) ➔ Claude Sonnet 4.6
- 이유: 에이전트 팀 전체를 지휘하는 핵심 역할입니다. 최상위 모델인 Opus 4.8은 토큰 단가가 너무 높습니다. 최신 Sonnet 4.6은 Opus급의 코딩 성능을 발휘하면서도 비용은 약 60% 절감할 수 있어 메인 제어용으로 가장 효율적입니다.
- oracle (코드 설계 및 디버깅) ➔ GPT-5.3-Codex
- 이유: 작성된 코드의 오류를 잡고 구조를 짜는 역할입니다. 일반 범용 모델보다 코드 특화 백본을 가진 Codex 계열(GPT-5.3-Codex)을 배치하여 불필요한 토큰 낭비(반복 프롬프트 및 오답으로 인한 재실행)를 줄이고 정밀한 코드 작성을 유도합니다.
- librarian (문서 및 리서치) ➔ Claude Haiku 4.5
- 이유: 외부 API 문서나 라이브러리를 읽어오는 작업은 읽기(Input) 토큰이 대량으로 발생합니다. 가용 모델 중 Haiku 4.5는 플래그십 모델 대비 최대 15배 이상 저렴하여, 대용량 문서를 읽어오는 리서치 역할에 비용 면에서 절대적으로 유리합니다.
- explore (코드 탐색 및 진입점 분석) ➔ GPT-5.4 mini
- 이유: 기존 레거시 코드 구조를 파악하고 파일들을 훑는 역할입니다. 컨텍스트를 넓게 쓰면서도 처리 속도가 빠르고 단가가 낮은 GPT-5.4 mini 혹은 Gemini 3.1 Pro가 적합합니다. 응답 속도와 경량화 비용 효율을 고려해 gpt-5.4-mini를 매칭했습니다.

## 3. 설정 적용 방법

   1. 프로젝트 루트 폴더 혹은 전역 설정 경로(~/.config/opencode/oh-my-opencode.json)의 파일을 엽니다.
   2. 위의 JSON 내용을 복사하여 붙여넣고 저장합니다.
   3. 만약 특정 세션에서 비용을 더 아끼거나 임시로 모델을 바꾸고 싶다면 OMO 채팅창에서 아래 명령어로 실시간 전환도 가능합니다.
   - 예시: /model librarian anthropic/claude-haiku-4.5

제시해 드린 모델 외에 혹시 특정 AI 프로바이더(예: Anthropic 혹은 OpenAI)의 크레딧만 집중적으로 소진하셔야 하는 상황인가요? 원하시는 특정 플랫폼 중심의 가성비 세팅으로 변경해 드릴 수도 있습니다.

## 가성비 극대화 설정 예제 (가용 모델 기준)

위에서 나열한 가용 모델만 사용하여 비용을 최소화하는 완전한 설정 파일입니다.

**핵심 전략**:
- **Heavy 작업** (oracle, ultrabrain, deep): 코드 특화 또는 추론 특화 모델로 재실행 횟수 최소화
- **Orchestration** (sisyphus, prometheus, metis, momus): Sonnet 4.6으로 Opus 호출 없이 품질 확보
- **반복/경량 작업** (explore, librarian, quick): Haiku 4.5 / GPT-5 mini로 최저 단가 배치

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/dev/assets/oh-my-opencode.schema.json",
  "agents": {
    "sisyphus": {
      "model": "github-copilot/claude-sonnet-4.6",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.4-mini" }
      ]
    },
    "oracle": {
      "model": "github-copilot/gpt-5.3-codex",
      "fallback_models": [
        { "model": "github-copilot/claude-sonnet-4.6" }
      ]
    },
    "librarian": {
      "model": "github-copilot/claude-haiku-4.5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5-mini" }
      ]
    },
    "explore": {
      "model": "github-copilot/gpt-5-mini",
      "fallback_models": [
        { "model": "github-copilot/claude-haiku-4.5" }
      ]
    },
    "multimodal-looker": {
      "model": "github-copilot/gpt-5.4-mini"
    },
    "prometheus": {
      "model": "github-copilot/claude-sonnet-4.6",
      "fallback_models": [
        { "model": "github-copilot/gemini-3.1-pro-preview" }
      ]
    },
    "metis": {
      "model": "github-copilot/claude-sonnet-4.6",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.4" }
      ]
    },
    "momus": {
      "model": "github-copilot/claude-sonnet-4.6",
      "fallback_models": [
        { "model": "github-copilot/gemini-3.1-pro-preview" }
      ]
    },
    "atlas": {
      "model": "github-copilot/claude-haiku-4.5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5-mini" }
      ]
    },
    "sisyphus-junior": {
      "model": "github-copilot/claude-sonnet-4.5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.4-mini" }
      ]
    }
  },
  "categories": {
    "visual-engineering": {
      "model": "github-copilot/gemini-3.1-pro-preview",
      "fallback_models": [
        { "model": "github-copilot/claude-sonnet-4.6" }
      ]
    },
    "ultrabrain": {
      "model": "github-copilot/gemini-3.1-pro-preview",
      "fallback_models": [
        { "model": "github-copilot/claude-opus-4.7" }
      ]
    },
    "deep": {
      "model": "github-copilot/gpt-5.3-codex",
      "fallback_models": [
        { "model": "github-copilot/claude-sonnet-4.6" }
      ]
    },
    "artistry": {
      "model": "github-copilot/gemini-3.1-pro-preview",
      "fallback_models": [
        { "model": "github-copilot/claude-sonnet-4.6" }
      ]
    },
    "quick": {
      "model": "github-copilot/claude-haiku-4.5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5-mini" }
      ]
    },
    "unspecified-low": {
      "model": "github-copilot/gpt-5-mini",
      "fallback_models": [
        { "model": "github-copilot/claude-haiku-4.5" }
      ]
    },
    "unspecified-high": {
      "model": "github-copilot/claude-sonnet-4.6",
      "fallback_models": [
        { "model": "github-copilot/gemini-3.1-pro-preview" }
      ]
    },
    "writing": {
      "model": "github-copilot/claude-haiku-4.5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5-mini" }
      ]
    }
  }
}
```

### 모델 등급별 배치 전략

| 등급 | 모델 | 배치 역할 |
|---|---|---|
| 허리 (균형) | `claude-sonnet-4.6` | sisyphus, prometheus, metis, momus, unspecified-high |
| 코드 특화 | `gpt-5.3-codex` | oracle, deep |
| 멀티모달·긴 추론 | `gemini-3.1-pro-preview` | visual-engineering, ultrabrain, artistry |
| 경량 (최저 단가) | `claude-haiku-4.5` / `gpt-5-mini` | librarian, explore, quick, writing, unspecified-low |

### 에이전트·카테고리별 선정 이유

| 역할 | 모델 | 선정 이유 |
|---|---|---|
| **sisyphus** | claude-sonnet-4.6 | 팀 전체 지휘 — Opus 대비 비용 ~60% 절감, 코딩 성능 동급 |
| **oracle** | gpt-5.3-codex | 코드 특화 백본 — 정밀도 높아 재실행 횟수 감소로 총 비용 절감 |
| **librarian** | claude-haiku-4.5 | 대용량 문서 읽기 — 플래그십 대비 최대 15× 저렴 |
| **explore** | gpt-5-mini | 반복 코드 탐색 — 최저 단가 + 빠른 응답 |
| **multimodal-looker** | gpt-5.4-mini | 단순 이미지 분석 — 경량으로 충분 |
| **prometheus / metis / momus** | claude-sonnet-4.6 | 플래닝·검토 추론 — Sonnet으로 품질 확보, Opus 미사용 |
| **sisyphus-junior** | claude-sonnet-4.5 | 하위 작업 실행 — 4.6보다 소폭 저렴 |
| **atlas** | claude-haiku-4.5 | 보조 에이전트 — 경량으로 충분 |
| **visual-engineering / artistry** | gemini-3.1-pro-preview | 긴 컨텍스트 + 멀티모달 강점, 비용 효율 우수 |
| **ultrabrain** | gemini-3.1-pro-preview | 긴 추론 컨텍스트 필요 — Opus 호출 최소화 |
| **deep** | gpt-5.3-codex | 코드 중심 자율 탐색 — Codex 특화 |
| **quick** | claude-haiku-4.5 | 단순 단일 파일 수정 — 최소 비용 |
| **unspecified-low** | gpt-5-mini | 저난도 작업 — 최저 단가 |
| **unspecified-high** | claude-sonnet-4.6 | 중간 난도 — Sonnet으로 품질·비용 균형 |
| **writing** | claude-haiku-4.5 | 문서 작성 — 경량으로 충분 |

## 우수 성능 타협 설정 (최상위 제외, 우수 등급만 사용)

최상위(Flagship) 등급인 `Claude Opus 4.8`, `GPT-5.4`, `Gemini 3.1 Pro`는 완전히 제외하고, **우수(Excellent) 등급인 `Claude Sonnet 5`와 `GPT-5.3-Codex` 두 모델만으로** 모든 에이전트를 구성한 절충안입니다.

- 최상위 모델 대비 토큰 단가와 지연 시간을 낮추면서도, 앞서 다룬 "가성비 극대화 설정"의 mini/Haiku 경량 모델보다는 한 단계 높은 성능을 유지하고 싶을 때 사용합니다.
- 우수 등급에 멀티모달 전용 모델(Gemini 계열)이 빠지므로, 이미지·비주얼 관련 역할도 자체적으로 멀티모달을 지원하는 `Claude Sonnet 5`로 대체했습니다.
- 코드 작성·디버깅처럼 코드 특화 백본이 유리한 역할에는 `GPT-5.3-Codex`를, 그 외 범용/추론/리서치 역할에는 `Claude Sonnet 5`를 배치했습니다.

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/dev/assets/oh-my-opencode.schema.json",
  "agents": {
    "sisyphus": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "oracle": {
      "model": "github-copilot/gpt-5.3-codex",
      "fallback_models": [
        { "model": "github-copilot/claude-sonnet-5" }
      ]
    },
    "librarian": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "explore": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "multimodal-looker": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "prometheus": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "metis": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "momus": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "atlas": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "sisyphus-junior": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    }
  },
  "categories": {
    "visual-engineering": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "ultrabrain": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "deep": {
      "model": "github-copilot/gpt-5.3-codex",
      "fallback_models": [
        { "model": "github-copilot/claude-sonnet-5" }
      ]
    },
    "artistry": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "quick": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "unspecified-low": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "unspecified-high": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    },
    "writing": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.3-codex" }
      ]
    }
  }
}
```

### 우수 성능 타협 배치 전략

| 등급 | 모델 | 배치 역할 |
|---|---|---|
| 우수 (코드 특화) | `gpt-5.3-codex` | oracle, deep |
| 우수 (범용·추론·멀티모달) | `claude-sonnet-5` | 그 외 모든 agents/categories |

### 선정 이유

| 역할 | 모델 | 선정 이유 |
|---|---|---|
| **oracle / deep** | gpt-5.3-codex | 코드 설계·디버깅·자율 코드 탐색은 코드 특화 백본이 여전히 유리하므로 우수 등급 중 코드 전문 모델 배치 |
| **그 외 전체 역할** | claude-sonnet-5 | 최상위 모델 없이도 준수한 추론·멀티모달·문서 이해 능력을 갖춘 범용 우수 모델로 통일하여 관리 단순화 |

> 참고: 이 설정은 모델 종류를 2개로 단순화해 관리 편의성을 높였지만, 역할별 특화(멀티모달 전용, 초장문 컨텍스트 등)가 필요한 경우 "성능 최우선 설정"의 Gemini 3.1 Pro / Claude Opus 4.8 배치를 부분적으로 참고해 개별 에이전트만 상향 조정하는 것도 방법입니다.

## 성능 최우선 설정 (비용 무시, 최고 성능 우선)

가용 모델 목록 중 **최상위(Flagship)** 등급과 **우수(Excellent)** 등급만 사용해 모든 에이전트가 최고 품질의 응답을 내도록 구성한 설정입니다. mini/Haiku 등 경량 모델은 fallback에서도 배제했습니다.

**등급 정의**:
- **최상위 (Flagship)**: `Claude Opus 4.8`, `GPT-5.4`, `Gemini 3.1 Pro` — 가장 깊은 추론·최장 컨텍스트가 필요한 핵심 역할에 배치
- **우수 (Excellent)**: `Claude Sonnet 5`, `GPT-5.3-Codex` — 최상위 대비 소폭 가볍지만 여전히 최고 수준의 코딩/추론 성능을 내는 역할에 배치

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/dev/assets/oh-my-opencode.schema.json",
  "agents": {
    "sisyphus": {
      "model": "github-copilot/claude-opus-4.8",
      "fallback_models": [
        { "model": "github-copilot/claude-sonnet-5" }
      ]
    },
    "oracle": {
      "model": "github-copilot/gpt-5.3-codex",
      "fallback_models": [
        { "model": "github-copilot/claude-opus-4.8" }
      ]
    },
    "librarian": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gemini-3.1-pro-preview" }
      ]
    },
    "explore": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gemini-3.1-pro-preview" }
      ]
    },
    "multimodal-looker": {
      "model": "github-copilot/gemini-3.1-pro-preview",
      "fallback_models": [
        { "model": "github-copilot/claude-opus-4.8" }
      ]
    },
    "prometheus": {
      "model": "github-copilot/claude-opus-4.8",
      "fallback_models": [
        { "model": "github-copilot/claude-sonnet-5" }
      ]
    },
    "metis": {
      "model": "github-copilot/claude-opus-4.8",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.4" }
      ]
    },
    "momus": {
      "model": "github-copilot/claude-opus-4.8",
      "fallback_models": [
        { "model": "github-copilot/gemini-3.1-pro-preview" }
      ]
    },
    "atlas": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.4" }
      ]
    },
    "sisyphus-junior": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.4" }
      ]
    }
  },
  "categories": {
    "visual-engineering": {
      "model": "github-copilot/gemini-3.1-pro-preview",
      "fallback_models": [
        { "model": "github-copilot/claude-opus-4.8" }
      ]
    },
    "ultrabrain": {
      "model": "github-copilot/claude-opus-4.8",
      "fallback_models": [
        { "model": "github-copilot/gemini-3.1-pro-preview" }
      ]
    },
    "deep": {
      "model": "github-copilot/gpt-5.3-codex",
      "fallback_models": [
        { "model": "github-copilot/claude-opus-4.8" }
      ]
    },
    "artistry": {
      "model": "github-copilot/gemini-3.1-pro-preview",
      "fallback_models": [
        { "model": "github-copilot/claude-sonnet-5" }
      ]
    },
    "quick": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.4" }
      ]
    },
    "unspecified-low": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.4" }
      ]
    },
    "unspecified-high": {
      "model": "github-copilot/claude-opus-4.8",
      "fallback_models": [
        { "model": "github-copilot/gemini-3.1-pro-preview" }
      ]
    },
    "writing": {
      "model": "github-copilot/claude-sonnet-5",
      "fallback_models": [
        { "model": "github-copilot/gpt-5.4" }
      ]
    }
  }
}
```

### 성능 최우선 배치 전략

| 등급 | 모델 | 배치 역할 |
|---|---|---|
| 최상위 (Flagship, 최고 성능) | `claude-opus-4.8` | sisyphus, prometheus, metis, momus, ultrabrain, unspecified-high |
| 최상위 (Flagship, 멀티모달·장문 컨텍스트) | `gemini-3.1-pro-preview` | multimodal-looker, visual-engineering, artistry |
| 우수 (Excellent, 코드 특화) | `gpt-5.3-codex` | oracle, deep |
| 우수 (Excellent, 범용 서브 역할) | `claude-sonnet-5` | librarian, explore, atlas, sisyphus-junior, quick, unspecified-low, writing |

### 에이전트·카테고리별 선정 이유 (성능 최우선)

| 역할 | 모델 | 선정 이유 |
|---|---|---|
| **sisyphus** | claude-opus-4.8 | 전체 팀 지휘 — 가용 모델 중 가장 강력한 추론력으로 오케스트레이션 오류 최소화 |
| **oracle** | gpt-5.3-codex | 코드 설계/디버깅 특화 백본 — 코딩 정밀도 최상위 |
| **librarian** | claude-sonnet-5 | 문서 리서치 — 경량 모델 배제, 문맥 이해도 높은 최신 Sonnet으로 정확도 확보 |
| **explore** | claude-sonnet-5 | 코드 탐색 — 대규모 코드베이스에서도 정확한 분석을 위해 우수 등급 배치 |
| **multimodal-looker** | gemini-3.1-pro-preview | 이미지·멀티모달 분석 — 해당 분야 최상위 모델 |
| **prometheus / metis / momus** | claude-opus-4.8 | 계획·비평·검증 — 가장 깊은 추론이 필요한 역할이라 최상위 모델 배치 |
| **atlas / sisyphus-junior** | claude-sonnet-5 | 보조 실행 역할 — 메인보다는 가볍지만 여전히 최상급 성능 유지 |
| **visual-engineering / artistry** | gemini-3.1-pro-preview | 긴 컨텍스트·멀티모달 강점 활용 |
| **ultrabrain** | claude-opus-4.8 | 가장 깊은 추론이 필요한 카테고리 — 최상위 모델 필수 |
| **deep** | gpt-5.3-codex | 코드 중심 자율 탐색 — 코드 특화 최상위 |
| **quick / unspecified-low / unspecified-high / writing** | claude-sonnet-5 / claude-opus-4.8 | 비용보다 품질 우선 원칙에 따라 경량 모델 대신 우수~최상위 등급만 배치 |

> 참고: 이 설정은 비용 효율을 고려하지 않으므로 토큰 사용량이 급격히 늘어날 수 있습니다. 실제 운영 시에는 위 "가성비 극대화 설정"과 병행하여 필요한 순간에만 `/model` 명령어로 전환하는 것을 권장합니다.

