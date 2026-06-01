# shell parameter expansion

- [bash - Shell Parameter Expansion](https://www.gnu.org/software/bash/manual/bash.html#Shell-Parameter-Expansion)
- [zsh - Parameter Expansion](http://zsh.sourceforge.net/Doc/Release/Expansion.html#Parameter-Expansion)

## substitution

| parameter status   | Set and Not Null     | Set But Null    | Unset           |
|--------------------|----------------------|-----------------|-----------------|
| ${parameter:-word} | substitute parameter | substitute word | substitute word |
| ${parameter-word}  | substitute parameter | substitute null | substitute word |
| ${parameter:=word} | substitute parameter | assign word     | assign word     |
| ${parameter=word}  | substitute parameter | substitute null | assign word     |
| ${parameter:?word} | substitute parameter | error, exit     | error, exit     |
| ${parameter?word}  | substitute parameter | **substitute null** | **error, exit** |
| ${parameter:+word} | substitute word      | substitute null | substitute null |
| ${parameter+word}  | substitute word      | **substitute word** | **substitute null** |

- `${...}`
  - `{...}`에 주어진 연산식을 수행하여 그 결과를 값으로 반환($의 의미)해준다.
- `:-`
  - substitute 치환
  - 변수 미선언 혹은 NULL인 경우 param의 값을 변경하지 않고(영향을 주지 않음), 지정한 값으로 확장(${})
- `:=`
  - assign 할당
  - 변수 미선언 혹은 NULL인 경우 param의 값을 지정한 값으로 할당(변경)하고, 지정한 값으로 확장(${})

## [parameter manipulation](https://wiki.kldp.org/HOWTO/html/Adv-Bash-Scr-HOWTO/string-manipulation.html)

- 예제를 테스트하기 위한 변수

```bash
string="abc-efg-123-abc"
```

- scpript에서 unset or null 변수 초기화

```bash
VAR=${VAR:-default}
```

- if로 unset or null 변수 초기화

```bash
if [[ !(-v VAR && -n ${VAR}) ]]; then
  VAR=default
  echo "VAR is set as \"${VAR}\" automatically."
fi
```

| 문자 | 설명 |
|---|---|
| ${변수} | $변수와 동일하지만 {} 사용해야만 동작하는 것들이 있음 |
| | 변수를 사용할 때는 항상 {}를 사용하는 것이 좋다 |
| | (예: echo ${string}) |
| ${변수:시작위치} | 지정 인덱스 부터 문자열 추출, 첫문자는 인덱스는 0번 |
| ${string:position} | (예: echo ${string:4}) |
| | positions:: |
| | abcABC123ABCabc |
| | 0123456789..... |
| ${변수:시작위치:길이} | 지정 인덱스 부터 지정한 길이 만큼의 문자열 추출, 첫문자는 인덱스는 0번 |
| ${string:position:length} | (예: echo ${string:4:3} # BC1) |
| | positions:: |
| | abcABC123ABCabc |
| | 0123456789..... |
| ${변수:시작위치:-끝위치} | 세 번째 parameter가 음수인 경우 해당 값은 끝위치를 나타내며 |
| ${string:offset-from-front:offset-from-end} | 끝위치 값 = offset from end(음수)이다 |
| | 시작위치 값 = offset from front(양수)이다 |
| | 시작위치 값이 음수일 수는 없으며 음수를 입력하는 경우 expansion은 정상동작하지 않는다 |
| | (예: echo ${string:3:-6} # ABC123) |
| | positions:: |
| | +++ 0123456789..... :시작 위치 값=offset from front |
| | abcABC123ABCabc |
| | --- .....9876543210 :끝 위치 값=offset from end |
| ${변수:-단어} | |
| ${parameter:-word} | 변수 참조시 default 값으로 확장에 사용 |
| | 참조(확장) 대상인 미선언 변수를 신규 정의 하지 않음 |
| | 변수의 확장 시점에 변수가 미선언 혹은 NULL인 경우 default 값으로 확장하여 반환, 그러나 변수에 default값을 할당하지 않음 |
| | 확장 결과 값 지정(변수에 값 할당 x) |
| | 변수 미선언 혹은 NULL일 때 기본값 지정(substitute) |
| | substitute) |
| 변수 미선언 혹은 NULL인 경우 | |
| param의 값을 변경하지 않고(영향을 주지 않음), 지정한 값으로 확장(${}) | |
| | 변수 참조시 default 값 지정에 사용 |
| | ex) script 내에서 positional param 참조시 사용 |
| | local fruit=${1:-apple} |
| local second=${2:-val2} | |
| local third=${3:-val3} | |
| | ex) script 내에서 변수 사용처에서 default값 지정 |
| | echo ${ENVAR:-val} |
| ${변수-단어} | 변수 미선언시만 기본값 지정 |
| | 예) |
| | ${parameter-HELLO} |
| ${변수:=단어} | 변수 선언(정의)시 default 값 지정에 사용 |
| 참조(확장) 대상인 미선언 변수를 신규 정의 함 | |
| ${parameter:=word} | |
| | 변수의 확장 시점에 변수가 미선언 혹은 NULL인 경우 default 값으로 확장하여 반환, 그리고 변수에 default값을 할당함 |
| | : ${VAR=varl} == VAR=${VAR:-val} |
| | assignment) |
| | 변수 미선언 혹은 NULL인 경우 |
| param의 값을 지정한 값으로 변경(할당)하고, 지정한 값으로 확장(${}) | |
| | 변수 선언시 default 값 지정에 사용 |
| | ex) script 내에서 미선언 parameter를 쉘변수로 정의 |
| | : ${SHELLVAR:=val} |
| 이 명령라인은 다음과 동일 | |
| SHELLVAR=${SHELLVAR:-val} | |
| | ':'는 true, no-op(operation) 또는 'no effect'라고 불리는 builtin 명령어로써 뒤에 명시되는 문자열에 대해서 명령라인으로 간주하고 확장($ 처리) 까지만 진행 후 처리를 중단한다. 항상 Zero(0) exitcode를 반환하며 표준 입력만 허용하고 어떤 표준 출력 처리도 하지 않는다. |
| |주로 다음의 용도로 사용|
| | 1, 쉘변수 확장만 수행 |
| 위의 예제문은 변수 선언 위치에서 SHELLVAR 변수에 값을 할당 하도록 쉘 확장(${})까지만 처리하고, 확장된 word가 명령어로써 실행되지는 않도록 한다. | |
| | 2, 스크립트에서 명령라인 주석 처리(# 대신) |
| | 3, 쉘 논리 연산에서 true 명령어 대신 사용, true 명령은 외부 프로그램이지만 :는 shell builtin으로써 true 명령보다 실행속도가 빠르다. |
| ${변수=단어} | 변수 미선언시만 쉘변수 선언 |
| | 예) |
| | : ${parameter=HELLO} |
| ${변수:?단어} | 변수 미선언 혹은 NULL일 때 단어 출력 후 스크립트 종료 |
| | (예: echo ${string:?HELLO}) |
| ${변수?단어} | 변수 미선언시만 단어 출력 후 스크립트 종료 |
| | (예: echo ${string?HELLO}) |
| ${변수:+단어} | 변수 선언시만 단어 사용 |
| | (예: echo ${string:+HELLO}) |
| ${변수+단어} | 변수 선언 혹은 NULL일 때 단어 사용 |
| | (예: echo ${string+HELLO}) |
| ${#변수} | 문자열 길이 |
| | (예: echo ${#string}) |
| ${변수#단어} | 변수 값의 앞에서 부터 짧게(narrow) 매칭된 부분 삭제 |
| | 주의) 패턴의 첫 문자는 변수 값의 첫 문자 또는 어야만 한다. |
| | *는 wild-card로 0개 이상의 모든 문자를 의미 |
| | $ s=a1b2b3c; echo ${s#*b} |
| | 2b3c |
| | $ s=a1b2b3c; echo ${s#a*b} |
| | 2b3c |
| ${변수##단어} | 변수 값의 앞에서 부터 길게(wide) 매칭된 부분 삭제 |
| | 주의) 패턴의 첫 문자는 변수 값의 첫 문자 또는 *이어야만 한다. |
