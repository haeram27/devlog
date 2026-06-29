# vscode: Local의 github.copilot extension을 Remote SSH 세션에서 사용

## 설정 방식

vscode remote에서 github.copilot extension을 설치하거나 구동하지 않고,
remote에서 사용하더라도 local에만 github.copilot extension을 설치하고 로그인

## remote에 github.copilot 관련 extension 제거(uninstall)

이미 remote vscode server에 github.copilot extension이 깔려 있으면 제거

## remote 환경에서 Local의 github.copilot extension을 사용하도록 설정

<Ctrl+Shift+P> (Command Palete) > Preferences: Open User Settings (JSON) 열기
아래를 그대로 추가(또는 병합)하고 저장

```yaml
{
  "remote.extensionKind": {
    "github.copilot": ["ui"],
    "github.copilot-chat": ["ui"]
  }
}
```

- [remote.extensionKind values](https://code.visualstudio.com/api/advanced-topics/extension-host#preferred-extension-location)
  - ["workspace"]: 기본값. Remote-SSH를 쓰면 원격(서버)에서 실행
  - ["ui"]: 항상 로컬(UI 프로세스)에서 실행
  - ["ui", "workspace"] or ["workspace", "ui"]: 선호 순서로 시도

## remote extension 자동 설치 방지 (User Settings)

```yaml
"remote.defaultExtensionsIfInstalledLocally": [
    "GitHub.vscode-pull-request-github"
],
```

remote.defaultExtensionsIfInstalledLocally 설정의 default 값은
"GitHub.copilot", "GitHub.copilot-chat"을 포함하므로 이를 제외

## login

github.copilot 로그인은 반드시 local에서 수행