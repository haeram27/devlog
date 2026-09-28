# powershell 주의 문법


|문법|PowerShell 의미|
|---|---|
|(Get-Date)|grouping expression, 식을 먼저 평가|
|$(Get-Date)|subexpression, 결과를 하나의 값으로 반환|
|& { ... }|스크립트 블록 실행|
|pwsh -Command ...|새 PowerShell 프로세스|


## `{}`

| 문법                    | 의미             |
| ----------------------- | -------------- |
| `{ ... }`               | ScriptBlock 생성 |
| `& { ... }`             | ScriptBlock 실행 |
| `if (...) { ... }`      | 조건문 본문         |
| `foreach (...) { ... }` | 반복문 본문         |
| `Where-Object { ... }`  | 필터 조건          |
| `function f { ... }`    | 함수 본문          |
| `@{ ... }`              | Hashtable 리터럴  |
| `${var}`                | 변수명 경계 지정      |


## `().subCommand()`

## `& $cmd`

### psh

```psh
$h = @{
    Name = "foo"
    Age  = 30
}

$h.GetType().Name
// Hashtable

$h["Name"]
// foo

$h.Name
// foo
```