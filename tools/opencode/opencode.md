# opencode

## 정리

- Qwen(ailbaba ai), Kimi(moonshot ai) 무료 ai 모델 기본 사용가능 (단, 둘 모두 china 국적)
- 그외 `open-ai`와 `github-copilot` 사용 가능.
- **antropic claude나 google gemini를 opencode를 통해서 사용하면 계정이 블럭 될 수 있음**
- `github copiot`만 사용할 거라면 `copilot cli` 사용이 나음

## install

- 공식 스크립트 사용

```bash
## install
curl -fsSL https://opencode.ai/install | bash

## uninstall
opencode uninstall
```

- Node.js (npm) 사용

```bash
npm install -g opencode-ai@latest
```

## omo(oh-my-opencode) install

설치:

```bash
npx oh-my-opencode install
```

환경 설정: ~/.zshrc

```bash
#####################
## OH MY OPENCODE
#####################
# disable send telemery
export OMO_SEND_ANONYMOUS_TELEMETRY=0
```
