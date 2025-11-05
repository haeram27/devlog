# shell에서 conditional expr 사용(TODO)

## if 와 test의 차이

`if` 는 command의 exitcode를 기준으로 참(zero)/거짓(non-zero)을 판단한다.  
`test`(`[]` or `[[ ]]`) 명령어는 논리연산 결과를 기준으로 참/거짓을 판단한다.  

## if

- `if` 는 command의 exitcode를 기준으로 참(zero)/거짓(non-zero)을 판단한다.  
exitcode가 0이면 참 그외 값이면 거짓으로 판단한다. 

- `if`의 인자로는 command을 사용해야한다. 그래서 `test` 명령이 if의 인자로 사용될 수 있다.  
command가 여러개일 경우(`;`으로 구분된) 마지막 command의 결과를 기준으로 판단한다.

```bash
if [!] <command>; ... then
    <command;>...;
elif [!] <command>; ... then
    <command;>...
else
    <command;>...
fi
```

## test와 keyword

`test`(`[]` or `[[ ]]`) 명령어는 논리연산 결과를 기준으로 참/거짓을 판단한다.  
`test`의 인자로는 `논리연산 expr`이 사용된다.  

`if`와 `test`에 사용되는 부정연산자 `!`는 직후 오는 단일 연산의 결과에 대해서만 적용된다.

### `[]`와 `[[]]` 차이점

(중요) bash, zsh를 사용한다면 **`[[ ]]`를 사용할 것

| 특징 | `[ ]` (test) | `[[ ]]` (keyword) |
|------|--------------|-------------------|
| 타입 | POSIX 표준 명령어 | Bash 확장 키워드 |
| 이식성 | 모든 POSIX 쉘에서 동작 | Bash, Zsh, Ksh만 지원 |
| 단어 분리 | 발생함 (변수 명에 인용 필수) | 발생하지 않음 |
| 패턴 매칭 | 제한적 | 지원 (`==`, `!=`로 glob 패턴) |
| 정규식 | 미지원 | 지원 (`=~` 연산자) |
| 논리 연산자 | `-a`, `-o` (권장 안 함) | `&&`, `||` |
| 괄호 그룹화 | `\( \)` (이스케이프 필요) | `( )` (직접 사용) |

### 변수에 quoting("") - 가장 중요한 차이

#### `[ ]` - 반드시 인용 필요

```bash
var="hello world"

# ? 에러 발생 - 위험한 코드
[ $var = "hello world" ]
# bash: [: too many arguments (단어 분리로 인해)

# ? 인용 필수
[ "$var" = "hello world" ]  # 정상 동작

# 빈 변수 처리
unset var
[ $var = "" ]   # ? 에러: [: =: unary operator expected
[ "$var" = "" ] # ? 정상
```

#### `[[ ]]` - 인용 불필요 (자동 보호)

```bash
var="hello world"

# ? 인용 없이도 동작
[[ $var = "hello world" ]]  # 정상

# ? 빈 변수도 안전하게 처리
unset var
[[ $var = "" ]]  # 정상 동작
```

### 언제 무엇을 사용할까?

#### `[ ]` 사용

- POSIX 호환이 필요한 경우 (sh, dash 등)
- 간단한 조건식
- 이식성이 중요한 스크립트

#### `[[ ]]` 사용 (대부분의 경우 권장)

- **Bash 스크립트 작성 시** (기본 권장)
- 패턴 매칭이 필요한 경우
- 정규식이 필요한 경우
- 복잡한 조건식
- 안전한 변수 처리 (인용 생략 가능)
