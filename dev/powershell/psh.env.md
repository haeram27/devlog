# powershell 환경 설정


## psh 버전확인

```ps1
$PSVersionTable.PSVersion
# 또는 version 7 인 경우
pwsh -v
```

## psh 7 설치 및 업그레이드

```ps1
# 관리자 계정으로 powershell을 열고
winget install --id Microsoft.PowerShell --source winget
# upgrade
winget upgrade --id Microsoft.PowerShell --source winget
```

## git windows 설치
- https://git-scm.com/install/windows

## OH-MY-POSH 설치

linux의 zsh의 ***[powerlevel10k](https://github.com/romkatv/powerlevel10k)*** 와 같이 shell 프롬프트 Theme를 제공해주는 프로젝트

Github 주소:
- https://github.com/jandedobbeleer/oh-my-posh
- https://github.com/JanDeDobbeleer/oh-my-posh/tree/main/themes

Linux의 zsh + oh-my-zsh 느낌과 가장 비슷한 조합은 PowerShell 7 + Oh My Posh + Meslo Nerd Font + Windows Terminal 입니다.

### Oh My Posh 설치

관리자 권한 PowerShell에서:

```ps1
winget install JanDeDobbeleer.OhMyPosh -s winget
```

설치 확인:

```ps1
oh-my-posh version
```

### Nerd Font 설치

Oh My Posh 테마 아이콘이 깨지지 않게 하려면 Nerd Font가 필요

Nerd Font는 특정 유니코드에 아이콘 이미지를 표시하는 폰트이며, 주로 프롬프트 꾸미기에 사용됨 

일반적으로 폰트 이름에 표기된 `NF`는 `Nerd Font`의 약자

예를 들어 Meslo 폰트 설치:

```ps1
oh-my-posh font install meslo
```

설치 후 `Windows Terminal 재시작 → 설정 → 프로필 → 글꼴(Font Face)` 에서 다음으로 변경:

```plain
MesloLGM Nerd Font Mono
```


### PowerShell 프로필 설정

프로필이란 PowerShell 실행시 사용되는 초기화 스크립트이며, bash의 `.bashrc` 파일에 대응된다.

일반적으로 경로는 다음과 같다.
- `C:\Users\<username>\Documents\PowerShell\Microsoft.PowerShell_profile.ps1`

프로필 파일 생성: (바로 편집으로 건너뛰기 가능)

```ps1
New-Item -Path $PROFILE -Type File -Force
```

프로필 편집:

```ps1
notepad $PROFILE
```

파일의 제일 아래에 내용 추가:

```ps1
oh-my-posh init pwsh | Invoke-Expression
```

저장 후 PowerShell 재시작

### 테마 적용

테마 json 다운로드:

```ps1
mkdir $env:USERPROFILE\.env\posh
cd $env:USERPROFILE\.env\posh
git clone https://github.com/JanDeDobbeleer/oh-my-posh.git
robocopy oh-my-posh/themes posh/themes
rm oh-my-posh
```

$env:USERPROFILE\.env\posh\themes

예를 들어 pure 테마 사용:

```ps1
oh-my-posh init pwsh --config "$env:USERPROFILE\.env\posh\themes\pure.omp.json" | Invoke-Expression
```

프로필($PROFILE)의 기존 줄을 위 내용으로 교체합니다.

### 추천 설정

PowerShell 프로필에 아래도 함께 넣으면 사용성이 좋아집니다.

```ps1
Import-Module PSReadLine

Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle InlineView
```

설치 후 터미널을 다시 열면 Git 브랜치, 상태, 경로 등이 컬러 프롬프트로 표시됩니다. Linux에서 zsh + oh-my-zsh 느낌과 가장 비슷한 조합은 PowerShell 7 + Oh My Posh + Meslo Nerd Font + Windows Terminal 입니다.


## gvim 설치

gVim은 winget으로 설치하는 것이 가장 간단합니다. Vim 패키지에는 GUI 버전인 gVim이 포함되어 있습니다.

### 설치

```ps1
winget install -e --id vim.vim
```

### 설치 확인

일반적으로 설치 후 Path 환경변수에 등록되지 않으면 즉시 명령 이름으로 실행할 수 없다.

1. CLI 버전:
```ps1
vim --version
```

2. gVim 실행:
```ps1
gvim
```

3. 시작 메뉴에서 gVim 검색하여 파일 위치 확인

### PATH 확인 및 환경 변수 등록

보통 다음과 비슷한 경로에 설치됩니다.

`C:\Users\user\AppData\Local\Programs\Vim\gvim.exe`

설치 위치 확인, 결과가 True이면 파일이 존재하는 것

```ps1
Test-Path "$env:LOCALAPPDATA\Programs\Vim\vim.exe"
```

환경 변수 등록
```ps1
[Environment]::SetEnvironmentVariable("Path",[Environment]::GetEnvironmentVariable("Path", "User") + ";$env:LOCALAPPDATA\Programs\Vim","User")
```

### Oh My Posh와 같이 쓰는 경우

PowerShell 프로필에 vim 스타일 별칭을 추가할 수 있습니다.

`$PROFILE`에 넣어두면 매번 사용할 수 있습니다.

```ps1
notepad $PROFILE
```

추가:

```ps1
Set-Alias vi vim
Set-Alias gvi gvim
```

설정 반영:

```ps1
. $PROFILE
```

## 최종 profile 예

```ps1
# Path: ~\Documents\PowerShell\Microsoft.PowerShell_profile.ps1
# Edit: vi $PROFILE
# UserHome: ~ or $env:USERPROFILE or %USERPROFILE%

########################################
# List all commands
########################################
# Get-Command
# Get-Command -CommandType Cmdlet
# Get-Command -CommandType Alias
# Get-Command -CommandType Function
# Get-Command -CommandType Application

########################################
# PSReadLine: key bindings and edit mode
########################################
# list of key bindings: Get-PSReadLineKeyHandler
# list of unbound keys: Get-PSReadLineKeyHandler -Unbound
Set-PSReadLineKeyHandler -Chord Ctrl+j -Function AcceptSuggestion
Set-PSReadLineKeyHandler -Chord Alt+f -Function AcceptNextSuggestionWord
Set-PSReadLineKeyHandler -Chord Ctrl+u -Function DeleteLine
Set-PSReadLineKeyHandler -Chord Ctrl+k -Function KillLine
#Set-PSReadLineKeyHandler -Chord Ctrl+a -Function BeginningOfLine

# Set-PSReadLineOption -EditMode Vi ## vi edit mode - ESC enables command mode, i enables insert mode
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle InlineView

########################################
# functions
########################################
# bat
function bats {
    bat -p --paging=never @args
}

# copilot
function Copilot-Auto {
    copilot --autopilot --yolo
}

# eza
function l  { eza --icons }
function ll { eza -l --git --icons }
function la { eza -la --git --icons }
function lt { eza --tree --level=2 --icons }


########################################
# alias
########################################
Set-Alias ls eza
Set-Alias vi vim
Set-Alias ff fzf
Set-Alias gvi gvim
Set-Alias cop.auto Copilot-Auto

########################################
# utils - init
########################################
# zoxide
if (Get-Command zoxide -ErrorAction SilentlyContinue) {
    Invoke-Expression (& { (zoxide init powershell | Out-String) })
}


########################################
# oh-my-posh
########################################
#oh-my-posh init pwsh | Invoke-Expression
oh-my-posh init pwsh --config "$env:USERPROFILE\.env\posh\themes\pure.omp.json" | Invoke-Expression
```