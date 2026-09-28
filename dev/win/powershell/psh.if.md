# PowerShell `if` 문이 `True`/`False`를 판단하는 기준

PowerShell의 `if`는 **표현식 결과를 Boolean으로 변환**해서 판단한다.

```powershell
if (<expression>) {
    # True
} else {
    # False
}
```

즉, 핵심은 "종료 코드"가 아니라 **`<expression>`의 값**이다.

## 기본 원리

```powershell
if (<expression>) { ... }
```

내부적으로는 `<expression>` 결과를 `[bool]`로 해석한 뒤 분기한다고 이해하면 된다.

> **문법 주의:** PowerShell은 `if` 뒤에 괄호가 **필수**다.  
> `if command` 형태는 허용되지 않으며 파서 오류가 발생한다.
>
> ```powershell
> if (Get-Process) { "OK" }   # 올바름
> if Get-Process { "OK" }     # 문법 오류
> ```

### `[bool]`은 무엇인가?

`[bool]`은 **명령어(cmdlet)가 아니라 타입 리터럴(type literal)** 이며, 값 앞에 붙여 **Boolean으로 형 변환(cast)** 할 때 쓰는 표현식이다.

```powershell
[bool]0         # False
[bool]1         # True
[bool]""        # False
[bool]"False"   # True (문자열이 비어있지 않기 때문)
```

따라서 `if` 문맥에서 보이는 `[bool]`은 "설정"이 아니라, 값을 `True/False`로 바꿔 평가하기 위한 **형 변환 용도**다.

## 값 타입별 평가 기준

| 구분 | `False`로 평가 | `True`로 평가 |
|---|---|---|
| Boolean | `$false` | `$true` |
| 숫자 | `0` | `1`, `-1`, `10` 등 0이 아닌 값 |
| 문자열 | `""` (빈 문자열) | `"False"`, `" "`(공백), `"Hello"` 등 길이 1 이상 |
| 배열 | `@()` (빈 배열) | `@(1)` 등 원소 1개 이상 |
| 객체 | `$null` | 생성된 객체(예: `[PSCustomObject]@{}`) |

## 자주 헷갈리는 포인트

### 1) `$false`와 `False`는 다르다

```powershell
if ($false) { "참" }  # 올바른 Boolean 리터럴 사용
if (False)  { "참" }  # False라는 명령/식별자를 찾으려다 오류 가능
```

### 2) `"False"`는 문자열이라 `True`

```powershell
[bool]"False"   # True
```

문자열 `"False"`는 Boolean `False`가 아니다.

## 명령 결과를 `if`에 넣는 경우

PowerShell은 명령의 **출력 객체**를 평가한다.

```powershell
if (Get-Item file.txt) {
    "파일 존재"
}
```

평가 흐름:

1. 명령 실행  
2. 출력(객체) 반환  
3. 반환값을 Boolean으로 변환  
4. `True`/`False` 결정

## 외부 프로그램(EXE)과 종료 코드

```powershell
if (git status) {
    "참"
}
```

위 코드는 `git`의 종료 코드가 아니라, `git status`의 **출력값**을 기준으로 판단한다.

종료 코드 기준으로 판단하려면 `$LASTEXITCODE`를 사용해야 한다.

```powershell
git status > $null
if ($LASTEXITCODE -eq 0) {
    "성공"
}
```

또는 마지막 명령 성공 여부 변수 `$?`를 사용할 수 있다.

```powershell
git status > $null
if ($?) {
    "성공"
} else {
    "실패"
}
```

## Bash와 비교

| 쉘 | 예시 | 판단 기준 |
|---|---|---|
| Bash | `if command; then ...; fi` | 종료 코드(`0`이면 성공) |
| PowerShell | `if (command) { ... }` | 명령 출력/표현식 결과를 `[bool]` 변환 |

## 핵심 요약

PowerShell의 `if`는 **"표현식 결과가 Boolean으로 변환됐을 때 참인가?"**를 판단한다.  
따라서 `if (command)`는 기본적으로 종료 코드 체크가 아니며, 종료 코드가 필요하면 `$LASTEXITCODE` 또는 `$?`를 명시적으로 확인해야 한다.