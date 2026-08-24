# PSFzf 모듈 설치 방법

PowerShell에서도 Linux나 macOS의 셸처럼 Ctrl + T (파일/폴더 검색), Ctrl + R (히스토리 검색), Alt + C (디렉터리 이동) 단축키를 그대로 사용할 수 있습니다.

다만 PowerShell은 기본 셸 통합(fzf --bash 등)을 지원하지 않기 때문에, PowerShell 모듈인 [PSFzf](https://github.com/kelleyma49/PSFzf)를 설치하여 단축키를 연결해야 합니다.

## Nuget 이용한 설치

PowerShell을 열고 아래 순서대로 명령어를 입력하세요.

### 1단계: PSFzf 모듈 설치

먼저 PowerShell용 fzf 래퍼 모듈을 설치합니다.

```ps1
Install-Module -Name PSFzf -Scope CurrentUser -Force
```

### 2단계: PowerShell 프로필 파일 열기

터미널이 켜질 때마다 자동으로 단축키가 로드되도록 프로필 설정 파일($PROFILE)을 메모장으로 엽니다.

```ps1
notepad $PROFILE
```

### 3단계: 단축키 스크립트 붙여넣기

메모장이 열리면 아래 코드를 그대로 붙여넣고 저장(Ctrl + S)한 뒤 닫습니다.

#### fzf 모듈 가져오기 및 단축키 매핑 설정

```text
Import-Module PSFzf
Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+t' `
                -PSReadlineChordReverseHistory 'Ctrl+r' `
                -PSReadlineChordChangeDirectory 'Alt+c'
. $PROFILE
```

### 4단계: 설정 적용하기

터미널을 재시작하거나 아래 명령어로 설정을 즉시 적용합니다.

```ps1
. $PROFILE
```

### 매핑된 단축키 사용법

* Ctrl + T: 현재 디렉터리 하위의 파일/폴더를 fzf 창으로 검색합니다. 선택 후 엔터를 누르면 현재 입력 중이던 명령행 자리에 해당 경로가 자동 삽입됩니다.
* Ctrl + R: 이전에 입력했던 명령어 히스토리를 퍼지 매칭으로 검색하여 불러옵니다.
* Alt + C: 하위 디렉터리를 검색한 뒤, 선택한 디렉터리로 바로 이동(cd)합니다.

⚠️ 주의: fzf 프로그램 본체(fzf.exe)가 시스템 환경 변수(Path)에 등록되어 있어야 정상 작동합니다. (설치되어 있지 않다면 winget install junegunn.fzf 또는 scoop install fzf로 먼저 설치해 주세요.)

## PSFzf 모듈 수동 설치 방법

### 1단계: 모듈 파일 직접 다운로드

Powershell Gallery PsFzf > Manual Download

- https://www.powershellgallery.com/packages/PSFzf

### 2단계: 폴더 구조 만들고 파일 옮기기

   1. 다운로드한 파일의 확장자를 .nupkg에서 **.zip**으로 변경합니다. (예: psfzf.zip)
   2. 압축을 해제하면 여러 파일이 나옵니다.
   3. 압축 해제된 폴더의 이름을 **PSFzf**로 변경합니다.
   4. 키보드의 Win + R을 누르고 아래 경로를 그대로 복사·붙여넣기하여 엔터를 누릅니다.

Powershell 버전 5- 경로:

```ps1
   %USERPROFILE%\Documents\WindowsPowerShell\Modules
```

Powershell 버전 7+ 경로:

```ps1
   %USERPROFILE%\Documents\PowerShell\Modules
```

   (만약 Modules 폴더가 없다면, WindowsPowerShell 폴더 안에서 마우스 우클릭으로 새 폴더를 만들고 이름을 Modules로 지정하세요.)
   5. 이 Modules 폴더 안에 아까 이름을 바꾼 PSFzf 폴더를 통째로 집어넣습니다.

올바른 최종 파일 경로 상태:

> C:\Users\<사용자이름>\Documents\PowerShell\Modules\PSFzf\PSFzf.psd1 (이 파일이 보이면 성공)

### 3단계: 단축키 프로필에 즉시 등록

이제 수동 설치가 끝났으니 PowerShell이 모듈을 바로 인식합니다. 마지막으로 단축키 설정을 등록합니다.

  1. 열려 있는 PowerShell 창을 완전히 닫고 새로 엽니다.
  2. 아래 명령어를 입력해 설정 파일을 메모장으로 켭니다.

```ps1
notepad $PROFILE
```

  3. 메모장 맨 밑에 아래 내용을 복사해서 붙여넣고 저장(Ctrl + S)한 뒤 닫습니다.

```text
# 
Import-Module PSFzf
Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+t' `
                -PSReadlineChordReverseHistory 'Ctrl+r' `
                -PSReadlineChordChangeDirectory 'Alt+c'
```
 
  4. PowerShell을 새로 고침합니다.

```powershell

# 보안 정책(디지털 서명 없음) 우회 - 인터넷에서 받은 파일은 윈도우가 내부적으로 '차단(Zone.Identifier)' 마크를 붙이고 실행 차단함
# 현재 로그인한 사용자 영역(CurrentUser)에만 스크립트 실행을 허용 및 차단 해제 (Unblock)
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
Get-ChildItem -Path "$HOME\Documents\PowerShell\Modules\PSFzf" -Recurse | Unblock-File

# 프로필 로드
. $PROFILE
```

터미널창에서 Ctrl + T 나 **Ctrl + R**을 누르면 기다리시던 fzf 퍼지 매칭 창이 정상적으로 실행
