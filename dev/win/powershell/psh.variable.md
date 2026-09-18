# PowerShell 변수 스코프 정리

## 변수 선언

PowerShell 변수는 **이름 앞의 스코프 한정자(scope modifier)** 로 접근 범위가 달라집니다.


기본 스코프 트리:

```text
Global
 └─ Script
     └─ Function A
         └─ Function B
```


### 변수 형태별 접근 제약 요약

| 형태 | 현재 스코프 | 자식 스코프 | 부모/상위 스코프 탐색 | 주 용도 |
|---|---|---|---|---|
| `$var` | O | O | O (읽기 시) | 일반 변수, 기본 선택 |
| `$local:var` | O | O | X (현재 스코프 명시) | 의도 명확화, 로컬 분리 |
| `$private:var` | O | X | X | 내부 구현 숨김 |
| `$script:var` | O (스크립트 범위) | O | 스크립트 범위 기준 | 스크립트 내부 공유 |
| `$global:var` | O (세션 전역) | O | 세션 전역 기준 | 세션 전체 공유 |

#### 실무에서 빠른 기준

```powershell
$var          # 기본값: 대부분 이걸 사용
$script:var   # 같은 스크립트 안에서만 공유할 때
$global:var   # 세션 전체 공유가 꼭 필요할 때만
$private:var  # 자식 스코프에 숨겨야 할 때
```


### 1) `$var` (한정자 없음)

**의미**
- 현재 스코프에서 변수 조회
- 없으면 부모 스코프를 순차 탐색

**제약 사항**
- 읽기 시 상위 스코프 값이 보일 수 있음
- 같은 이름으로 대입하면 현재 스코프에 새 변수(shadowing)가 생길 수 있음
- "현재 스코프만 강제" 동작은 아님

**예제**
```powershell
$Name = "Global"

function Test {
    $Name
}

Test   # Global
```

### 2) `$local:var`

**의미**
- 현재 스코프를 명시해서 변수 생성/참조

**제약 사항**
- 현재 스코프 기준으로만 처리됨(의도를 명확히 할 때 유용)
- 자식 스코프에서는 일반 변수처럼 조회 가능
- 부모 스코프 동일 이름 변수와 독립적으로 동작 가능

**예제**
```powershell
$Name = "Parent"

function Test {
    $local:Name = "Local"
    $Name
}

Test   # Local
$Name  # Parent
```

### 3) `$private:var`

**의미**
- 현재 스코프 내부 전용 변수

**제약 사항**
- 현재 스코프에서만 접근 가능
- 자식 스코프에서 조회 불가
- 내부 구현 숨김(캡슐화) 용도로 적합

**예제**
```powershell
function Parent {
    $private:Secret = "Password"

    function Child {
        $Secret
    }

    Child
}

Parent   # 출력 없음
```

### 4) `$script:var`

**의미**
- 현재 `.ps1` 스크립트 파일 범위에서 공유

**제약 사항**
- 같은 스크립트 내 함수/블록에서 공용 상태로 사용 가능
- 스크립트 범위를 넘어 자동 공유되지 않음
- 과도한 전역 상태처럼 쓰면 테스트/유지보수가 어려워질 수 있음

**예제**
```powershell
$script:Count = 0

function Add-Count {
    $script:Count++
}
```

### 5) `$global:var`

**의미**
- 현재 PowerShell 세션 전체 범위

**제약 사항**
- 어디서든 접근 가능해 충돌 가능성이 큼
- 스크립트/모듈 간 부작용을 만들기 쉬움
- 꼭 필요한 공유 상태에서만 제한적으로 사용 권장

**예제**
```powershell
$global:AppMode = "Dev"
```

## 스코프 선언과 참조/대입 규칙

변수 스코프는 선언과 참조에서 항상 동일할 필요는 없습니다.  
다만 **읽기**와 **쓰기** 동작이 다르므로 구분해서 이해해야 합니다.

### 1) 읽기(참조): 스코프 탐색 가능

`$script:var`로 선언했더라도 하위 스코프에서 `$var`로 읽을 수 있습니다(동일 이름 로컬 변수가 없을 때).

```powershell
$script:var = "A"

function Get-V {
    $var
}

Get-V  # A
```

### 2) 쓰기(대입): 의도한 스코프에 명시 권장

하위 스코프에서 `$var = ...` 를 대입하면 로컬 변수가 새로 생성되어, 상위 `$script:var`가 안 바뀔 수 있습니다.

```powershell
$script:var = "A"

function Set-V {
    $var = "B"
}

Set-V
$script:var  # A
```

상위 스코프 값을 실제로 바꾸려면 대입도 명시적으로 해야 합니다.

```powershell
function Set-V2 {
    $script:var = "B"
}
```

### 3) 실무 기준

- 읽기 전용이면 `$var` 참조를 허용해도 됨
- 상태 변경(쓰기)이 있으면 `$script:`/`$global:` 등을 명시해서 의도를 고정
- **공용 상태 변수는 읽기/쓰기 모두 같은 한정자를 써서 혼동 방지**

## 변수 형태별 접근 제약 요약

| 형태 | 현재 스코프 | 자식 스코프 | 부모/상위 스코프 탐색 | 주 용도 |
|---|---|---|---|---|
| `$var` | O | O | O (읽기 시) | 일반 변수, 기본 선택 |
| `$local:var` | O | O | X (현재 스코프 명시) | 의도 명확화, 로컬 분리 |
| `$private:var` | O | X | X | 내부 구현 숨김 |
| `$script:var` | O (스크립트 범위) | O | 스크립트 범위 기준 | 스크립트 내부 공유 |
| `$global:var` | O (세션 전역) | O | 세션 전역 기준 | 세션 전체 공유 |

## 실무에서 빠른 기준

```powershell
$var          # 기본값: 대부분 이걸 사용
$script:var   # 같은 스크립트 안에서만 공유할 때
$global:var   # 세션 전체 공유가 꼭 필요할 때만
$private:var  # 자식 스코프에 숨겨야 할 때
```

## 변수의 타입

PowerShell 변수는 "항상 문자열"이 아니라 **.NET 객체 타입**을 담습니다.

### 1) 타입 확인

```powershell
$a = 10
$b = 3.14
$c = "10"

$a.GetType().Name   # Int32
$b.GetType().Name   # Double
$c.GetType().Name   # String
```

### 2) 숫자 표현과 숫자 연산

```powershell
$n1 = 10        # Int32
$n2 = 2.5       # Double
$n3 = 12.34d    # Decimal (접미사 d)

$n1 + 2         # 12
$n1 - 2         # 8
$n1 * 2         # 20
$n1 / 4         # 2.5
$n1 % 3         # 1
```

### 3) 문자열과 섞일 때 제약

```powershell
"10" + 2        # "102" (문자열 결합)
[int]"10" + 2   # 12 (명시적 캐스팅)
```

- 연산 전에 타입을 맞추지 않으면 의도와 다른 결과가 나올 수 있음
- 특히 `$env:XXX` 값은 기본적으로 문자열

### 4) 안전한 숫자 변환

```powershell
if ([int]::TryParse($env:APP_PORT, [ref]$port)) {
    $nextPort = $port + 1
} else {
    $nextPort = 8081
}
```

- 캐스팅(`[int]...`)은 실패 시 예외/오류 가능
- 입력 신뢰가 낮으면 `TryParse` 사용 권장

## 시스템/사용자 환경 변수 참조

PowerShell에서 환경 변수는 보통 `$env:` 드라이브로 읽고, 범위(프로세스/사용자/시스템)를 구분해야 할 때는 .NET API를 사용합니다.

### 1) 기본 참조: `$env:변수명`

```powershell
$env:PATH
$env:JAVA_HOME
```

- 현재 PowerShell 프로세스 기준 값
- 일반적으로 사용자/시스템 값이 합쳐진 결과를 보게 됨(특히 `PATH`)

### 2) 범위별 직접 참조: `[Environment]::GetEnvironmentVariable()`

```powershell
# 사용자(User) 환경 변수
[Environment]::GetEnvironmentVariable("JAVA_HOME", "User")

# 시스템(Machine) 환경 변수
[Environment]::GetEnvironmentVariable("JAVA_HOME", "Machine")

# 현재 프로세스(Process) 환경 변수
[Environment]::GetEnvironmentVariable("JAVA_HOME", "Process")
```

- `"User"`: 현재 로그인 사용자 레벨
- `"Machine"`: 시스템 전체(컴퓨터) 레벨
- `"Process"`: 현재 실행 중인 프로세스 레벨

### 3) 예제: PATH를 사용자/시스템으로 나눠 확인

```powershell
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
$machinePath = [Environment]::GetEnvironmentVariable("Path", "Machine")

$userPath
$machinePath
```

### 4) 제약/주의 사항

- 세션에서 `$env:NAME = "..."` 로 바꾼 값은 기본적으로 **현재 프로세스에만 즉시 반영**
- 사용자/시스템 환경 변수를 OS에 영구 반영하려면 `SetEnvironmentVariable(..., "User"|"Machine")` 또는 시스템 설정 변경이 필요
- 이미 떠 있는 다른 터미널/프로세스에는 즉시 전파되지 않을 수 있어 새 세션에서 재확인 필요

## 변수 참조 시 확장/가공 패턴

PowerShell은 Bash의 `${VAR:-default}` 같은 문법 대신, **문자열/배열 인덱싱 + .NET 메서드 + 연산자** 조합으로 확장합니다.

### 1) 변수 경계 명확히 하기 (`${}`)

```powershell
$name = "dev"
"${name}_log"   # dev_log
```

- 변수명 뒤에 문자(특히 `_`, 영숫자)가 붙는 문자열에서 경계가 모호할 때 사용

### 2) 앞/뒤 특정 위치 자르기

```powershell
$s = "PowerShell"

$s.Substring(0, 5)  # 앞에서 5글자: Power
$s.Substring(5)     # 5번 인덱스부터 끝까지: Shell

$s[0..4] -join ""   # 앞 5글자: Power
$s[-5..-1] -join "" # 뒤 5글자: Shell
```

- `Substring(start, length)`는 범위를 벗어나면 예외 발생
- 인덱싱(`[]`)은 문자 배열처럼 다룰 수 있어 짧은 처리에 편함

### 3) 구분자 기준으로 앞/뒤 자르기

```powershell
$f = "app-prod-01.log"

($f -split "-", 2)[0]   # 첫 '-' 앞: app
($f -split "-", 2)[1]   # 첫 '-' 뒤: prod-01.log

$f -replace "\.log$", ""  # 뒤 확장자 제거: app-prod-01
```

### 4) 대소문자 변환

```powershell
$name = "DevLog"

$name.ToUpper()  # DEVLOG
$name.ToLower()  # devlog
```

### 5) 미선언/Null/빈 문자열 기본값 설정

```powershell
# 5-1. Null일 때 기본값 (PowerShell 7+)
$port = $null
$port = $port ?? 8080

# 5-2. 미선언/Null/빈 문자열까지 함께 처리
if ([string]::IsNullOrWhiteSpace($env:APP_MODE)) {
    $mode = "dev"
} else {
    $mode = $env:APP_MODE
}
```

- `??` 는 **왼쪽이 null일 때만** 기본값 사용
- 빈 문자열(`""`)까지 기본 처리하려면 `IsNullOrEmpty`/`IsNullOrWhiteSpace` 사용

### 6) 실무 자주 쓰는 조합 예제

```powershell
# APP_NAME이 없으면 기본값 사용 후, 소문자 + 접미사
$app = ([string]::IsNullOrWhiteSpace($env:APP_NAME)) ? "myapp" : $env:APP_NAME
$logFile = "${($app.ToLower())}.log"
```