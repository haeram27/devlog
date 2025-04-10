
# 공백/ANSI Control Sequence 포함 확인

---

- [공백/ANSI Control Sequence 포함 확인](#공백ansi-control-sequence-포함-확인)
  - [공백 포함 여부 확인](#공백-포함-여부-확인)
  - [터미널 제어 코드 포함 여부 확인](#터미널-제어-코드-포함-여부-확인)
  - [예](#예)
- [공백 제거를 위한 sanitize 함수](#공백-제거를-위한-sanitize-함수)
  - [sanitize 함수](#sanitize-함수)
  - [참고](#참고)
    - [character class:  \[\[:print:\]\]](#character-class--print)
- [터미널 제어 시퀀스 제거를 위한 sanitize 함수](#터미널-제어-시퀀스-제거를-위한-sanitize-함수)
  - [터미널 제어 시퀀스](#터미널-제어-시퀀스)
    - [대표적인 터미널 제어 시퀀스 종류](#대표적인-터미널-제어-시퀀스-종류)
    - [예: 대표적인 OSC 시퀀스](#예-대표적인-osc-시퀀스)
  - [sanitize 함수](#sanitize-함수-1)
- [터미널 제어 시퀀스 (TODO)](#터미널-제어-시퀀스-todo)
  - [표준 문서](#표준-문서)
  - [대표적인 터미널 제어 시퀀스 종류](#대표적인-터미널-제어-시퀀스-종류-1)
  - [대표적인 OSC 시퀀스](#대표적인-osc-시퀀스)
  - [참고](#참고-1)
    - [ST (String Terminator)](#st-string-terminator)

---
shell의 어떤 데이터에 공백 또는 터미널 제어 시퀀스 포함 여부를 확인하려면 다음과 같은 방식을 사용한다.

- ***공백 포함 여부 확인***은 `[]`로 감싸서 echo로 출력
- ***터미널 제어 시퀀스 포함 여부 확인***은 `hd(hexdump -C)` 또는 `hexdump -c` 명령으로 출력

```bash
cat [${VAR}] | hexdump -c
```

## 공백 포함 여부 확인

```bash
echo ${VAR}
echo "[${VAR}]"
```

## 터미널 제어 코드 포함 여부 확인

터미널 제어 코드

```bash
echo ${VAR}
echo -n "${VAR}" | hd
echo -n "${VAR}" | hexdump -C
echo -n "${VAR}" | hexdump -c
```

## 예

```bash
$ temp="hello  "
$ echo -n "[$temp]" | hexdump -c
0000000   [   h   e   l   l   o           ]  \n
000000a
```

다음은 공백(\t, \r, \n) 및 텍스트 컬러를 위한 SGR 제어코드를 포함하는 데이터 예제이다.
여기서 temp 변수는 echo로 단순 출력시 red 컬러로 출력되며, 터미널 제어 코드는 보이지 않는다.
`hexdump -c` 명령을 사용하면 SGR 제어 코드도 확인이 가능하다.

```bash
$ temp=$'\033[0;31mhello\t\r\n\033[0m'
$ temp="$(printf '\033[0;31mhello\t\r\n\033[0m')"

$ echo "[$temp]"
[hello
]

$ echo -n "[$temp]" | hexdump -c
0000000   [ 033   [   0   ;   3   1   m   h   e   l   l   o  \t  \r  \n
0000010 033   [   0   m   ]
0000015
```

다음 예제는 터미널 제목을 설정하는 OSC 제어 시퀀스를 포함하는 temp 데이터이다.
이 temp 데이터를 단순히 echo로 출력했을 때에는 "6.0" 이라는 값만 보이게 된다.
"033]0;" 부터 "\a"에 이르는 제어 시퀀스 바이너리는 터미널에 출력되지 않는다.

```bash
$ temp="$(printf '\x1b\x5d\x30\x3b\x6d\x6f\x6e\x67\x6f\x73\x68\x20\x6d\x6f\x6e\x67\x6f\x64\x62\x3a\x2f\x2f\x3c\x63\x72\x65\x64\x65\x6e\x74\x69\x61\x6c\x73\x3e\x40\x31\x32\x37\x2e\x30\x2e\x30\x2e\x31\x3a\x32\x37\x30\x31\x37\x2f\x61\x64\x6d\x69\x6e\x3f\x64\x69\x72\x65\x63\x74\x43\x6f\x6e\x6e\x65\x63\x74\x69\x6f\x6e\x3d\x74\x72\x75\x65\x26\x73\x65\x72\x76\x65\x72\x53\x65\x6c\x65\x63\x74\x69\x6f\x6e\x54\x69\x6d\x65\x6f\x75\x74\x4d\x53\x3d\x32\x30\x30\x30\x07\x36\x2e\x30\x0d\x0a')"

$echo $temp
6.0

$ echo -n "$temp" | hexdump -c
0000000 033   ]   0   ;   m   o   n   g   o   s   h       m   o   n   g
0000010   o   d   b   :   /   /   <   c   r   e   d   e   n   t   i   a
0000020   l   s   >   @   1   2   7   .   0   .   0   .   1   :   2   7
0000030   0   1   7   /   a   d   m   i   n   ?   d   i   r   e   c   t
0000040   C   o   n   n   e   c   t   i   o   n   =   t   r   u   e   &
0000050   s   e   r   v   e   r   S   e   l   e   c   t   i   o   n   T
0000060   i   m   e   o   u   t   M   S   =   2   0   0   0  \a   6   .
0000070   0  \r
0000072
```

# 공백 제거를 위한 sanitize 함수

## sanitize 함수

다음은 터미널 제어 시퀀스 제거를 위한 sanitize 함수 이다.

```bash
#!/bin/bash

sanitize_space() {
  echo "$1"
    | tr -d '\r\n'
    | tr -cd '[:print:]'
    | xargs
}

sanitize_space_c() {
  echo "$1" |
    tr -d '\r\n' |          # Remove CR, LF
    tr -cd '[:print:]' |    # Remove non-printable chars
    xargs                   # Trim and flatten
}

var="$(printf '\x1b\x5d\x30\x3b\x6d\x6f\x6e\x67\x6f\x73\x68\x20\x6d\x6f\x6e\x67\x6f\x64\x62\x3a\x2f\x2f\x3c\x63\x72\x65\x64\x65\x6e\x74\x69\x61\x6c\x73\x3e\x40\x31\x32\x37\x2e\x30\x2e\x30\x2e\x31\x3a\x32\x37\x30\x31\x37\x2f\x61\x64\x6d\x69\x6e\x3f\x64\x69\x72\x65\x63\x74\x43\x6f\x6e\x6e\x65\x63\x74\x69\x6f\x6e\x3d\x74\x72\x75\x65\x26\x73\x65\x72\x76\x65\x72\x53\x65\x6c\x65\x63\x74\x69\x6f\x6e\x54\x69\x6d\x65\x6f\x75\x74\x4d\x53\x3d\x32\x30\x30\x30\x07\x36\x2e\x30\x0d\x0a')"

cleaned=$(sanitize_space "$var")
echo "$cleaned"
```

## 참고

| 표준 | 설명 | 링크 |
|------|------|------|
| **ECMA-48** | ANSI 제어 시퀀스 정의 및 의미 | [ecma-international.org](https://ecma-international.org/publications-and-standards/standards/ecma-48/) |
| **XTerm ctlseqs** | 터미널에서의 ANSI 제어 시퀀스 실제 구현 내용 (시퀀스별 ASCII값 확인) | [ctlseqs.html](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html) |

### character class:  \[\[:print:\]\]
<https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap07.html#tag_07_03_01>
터미널에 출력 가능한 문자 클래스(눈으로 볼 수 있는 문자만 포함하는 클래스)
upper, lower, alpha, digit, xdigit, punct, graph 클래스와 \<space\>(0x20) 포함
cntrl (제어문자 클래스) 제외, \<space\>이외 tab,CR,LF 공백 문자도 제외

# 터미널 제어 시퀀스 제거를 위한 sanitize 함수

## 터미널 제어 시퀀스

터미널을 제어(화면 설정/하이퍼 링크 등)하기 위하여 사용되는 이진 시퀀스 데이터
터미널에 제어 시퀀스 데이터가 전달되면 터미널 환경을 변경 할 수 있다.
일부 어플리케이션에서 stdout이 터미널 장치(tty or Pseudo-TTY) 인 경우 터미널 제어 시퀀스 데이터를 응답에 한다.
터미널 제어 시퀀스는 주로 터미널의 appearance(윈도우 타이틀, 텍스트 컬러)를 변경하여 사람에 대한 가독성/편의성을 추가 하거나 디바이스 제어를 하여 알림 소리 등을 발생시키는데 사용된다.

### 대표적인 터미널 제어 시퀀스 종류

| 시퀀스 종류 | 예시 | 설명 |
|-------------|------|------|
| **OSC** | `ESC ] 0;title BEL` | 윈도우 제목 설정 |
| **CSI/SGR** | `ESC [31m` | 글자 색, 스타일 등 |
| **ESC 단독** | `ESC ( B` | 문자 세트 전환 |
| **DCS/APC/PM/SOS** | `ESC P...ESC \` | 디바이스 제어 시퀀스 등 |
| **기타** | `ESC ^...ESC \` | 다양한 확장 시퀀스 |

### 예: 대표적인 OSC 시퀀스

| 시퀀스 코드 | 의미                         |
|-------------|------------------------------|
| `ESC ] 0;<TITLE-VALUE> BEL` | 윈도우/탭 타이틀 설정        |
| `ESC ] 1;<NAME-VALUE> BEL` | 아이콘 이름 설정 (GUI용)    |
| `ESC ] 8;;<URI-VALUE> ST` | 하이퍼링크 (iTerm2, Kitty 등) |

## sanitize 함수

다음은 터미널 제어 시퀀스 제거를 위한 sanitize 함수 이다.

```bash
#!/bin/bash

sanitize_term_ctl_seq() {
  echo "$1"
    | sed -E $'s/\x1b\\]([^\x1b\x07]*)(\x07|\x1b\\\\)//g'
    | sed -E $'s/\x1b\\[[0-9;?]*m//g'
    | sed -E $'s/\x1b[()][A-Za-z0-9]//g'
    | sed -E $'s/\x1bP.*?\x1b\\\\//g'
    | tr -d '\r\n'
    | tr -cd '[:print:]'
    | xargs
}

sanitize_term_ctl_seq_c() {
  echo "$1" |
    sed -E $'s/\x1b\\]([^\x1b\x07]*)(\x07|\x1b\\\\)//g' |  # Remove OSC
    sed -E $'s/\x1b\\[[0-9;?]*m//g' |                      # Remove CSI/SGR
    sed -E $'s/\x1b[()][A-Za-z0-9]//g' |                   # Remove ESC () type
    sed -E $'s/\x1bP.*?\x1b\\\\//g' |                      # Remove DCS
    tr -d '\r\n' |                                         # Remove CR, LF
    tr -cd '[:print:]' |                                   # Remove non-printable chars
    xargs                                                  # Trim and flatten
}

osc_str="$(printf '\x1b\x5d\x30\x3b\x6d\x6f\x6e\x67\x6f\x73\x68\x20\x6d\x6f\x6e\x67\x6f\x64\x62\x3a\x2f\x2f\x3c\x63\x72\x65\x64\x65\x6e\x74\x69\x61\x6c\x73\x3e\x40\x31\x32\x37\x2e\x30\x2e\x30\x2e\x31\x3a\x32\x37\x30\x31\x37\x2f\x61\x64\x6d\x69\x6e\x3f\x64\x69\x72\x65\x63\x74\x43\x6f\x6e\x6e\x65\x63\x74\x69\x6f\x6e\x3d\x74\x72\x75\x65\x26\x73\x65\x72\x76\x65\x72\x53\x65\x6c\x65\x63\x74\x69\x6f\x6e\x54\x69\x6d\x65\x6f\x75\x74\x4d\x53\x3d\x32\x30\x30\x30\x07\x36\x2e\x30\x0d\x0a')"

sgr_str=$'\x1b\x5b\x30\x3b\x33\x31\x6d\x68\x65\x6c\x6c\x6f\x1b\x5b\x30\x6d'

cleaned=$(sanitize_term_ctl_seq "${ocs_str}")
echo "$cleaned"

cleaned=$(sanitize_term_ctl_seq "${sgr_str}")
echo "$cleaned"
```

sed 명령으로 시작하는 라인은 각 제어 시퀀스 구간을 제거 하도록 시퀀스의 시작/종료 코드 구간을 regex 패턴으로 매칭한다.

``` sed 's/[^[:print:]]//g' ```
표준입력으로 주어진 문자열에서 printable 문자클래스를 제외한 모든 문자를 제거

# 터미널 제어 시퀀스 (TODO)

터미널을 제어(화면 설정/하이퍼 링크 등)하기 위하여 사용되는  7-bit(C0)) 또는 8-bit(C1)로 표현되는 특정 이진 시퀀스 값.
터미널 제어 시퀀스는 ANSI 제어 시퀀스에 포함되며 ASCII 코드의 \0x00 부터 \x1F 까지의 일부 제어 문자들도 ANSI 제어 시퀀스에서 정의된 내용이다. ANSI 제어 시퀀스는 printable하지 않으므로 실제로 터미널이 출력물에서 문자로 보여질 수는 없다.

터미널 장치(TTY or Pseudo-TTY)가 표준 입력으로 제어 코드를 받게 되면 터미널 별로  미리 구현된 동작을 수행하게 된다.
터미널 제어 시퀀스는 주로 터미널의 appearance(윈도우 타이틀, 텍스트 컬러)를 변경하여 사람에 대한 가독성/편의성을 추가 하거나 디바이스 제어를 하는데 사용된다.

일부 어플리케이션에서 stdout이 터미널 장치(tty or Pseudo-TTY) 인 경우 터미널 제어 시퀀스 데이터를 응답에 포함시킨다. 이 경우 만약 어플리케이션의 응답을 수신처에서 재가공 하여 사용해야할 경우 제어 시퀀스가 문제를 일으키기도 한다. (이러한 상황에서 전문 용어로 "쓰레기 값"이라고 표현하기도 함)
그러므로 쉘에서 어플리케이션의 응답 데이터를 재사용해야하는 경우에는 사실 어플리케이션을 실행할 때 tty 환경을 사용하지 않는 것이 최선이다.

## 표준 문서

| 표준 | 설명 | 링크 |
|------|------|------|
| **ECMA-48** | ANSI 제어 시퀀스 정의 및 의미 | [ecma-international.org](https://ecma-international.org/publications-and-standards/standards/ecma-48/) |
| **ISO/IEC 6429** | ECMA-48의 국제판 | 상업용 구매 또는 일부 기술문서 사이트에서 가능 |
| **XTerm ctlseqs** | 터미널에서의 ANSI 제어 시퀀스 실제 구현 내용 (시퀀스별 ASCII값 확인) | [ctlseqs.html](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html) |
| **VT100/DEC docs** | 전통적 터미널 시퀀스 | 여러 오픈소스 아카이브에서 제공 |

## 대표적인 터미널 제어 시퀀스 종류

* OSC (OPERATING SYSTEM COMMAND)
- DCS (DEVICE CONTROL STRING)
- APC (APPLICATION PROGRAM COMMAND)
- PM (PRIVACY MESSAGE)
- SGR (SELECT GRAPHIC RENDITION)
- CSI (CONTROL SEQUENCE INTRODUCER)
- SOS (START OF STRING)
- ST (STRING TERMINATOR)

| Parameter | 예시 | 설명 |
|-------------|------|------|
| C | A single (required) character. |
| Ps | A single (usually optional) numeric parameter, composed of one or more digits. |
| Pm | Any number of single numeric parameters, separated by ; character(s).  Individual values for the parameters are listed with Ps |
| Pt | A text parameter composed of printable characters. |

| 시퀀스 종류 | 예시 | 설명 |
|-------------|------|------|
| OSC | `ESC ] 0;title BEL` or "OSC *Ps* ; *Pt* ST" | 윈도우 제목 설정 |
| DCS | `ESC [31m` | 글자 색, 스타일 등 |
| APC | `ESC ( B` | 문자 세트 전환 |
| **DCS/APC/PM/SOS** | `ESC P...ESC \` | 디바이스 제어 시퀀스 등 |
| **기타** | `ESC ^...ESC \` | 다양한 확장 시퀀스 |

| 시퀀스 종류 | 예시 | 설명 |
|-------------|------|------|
| **OSC** | "OSC *Ps* ; *Pt* BEL" or "OSC *Ps* ; *Pt* ST" | 윈도우 제목 설정 |
| **CSI/SGR** | `ESC [31m` | 글자 색, 스타일 등 |
| **ESC 단독** | `ESC ( B` | 문자 세트 전환 |
| **DCS/APC/PM/SOS** | `ESC P...ESC \` | 디바이스 제어 시퀀스 등 |
| **기타** | `ESC ^...ESC \` | 다양한 확장 시퀀스 |

## 대표적인 OSC 시퀀스

| 시퀀스 코드 | 의미                         |
|-------------|------------------------------|
| `ESC ] 0;<TITLE-VALUE> BEL` | 윈도우/탭 타이틀 설정        |
| `ESC ] 1;<NAME-VALUE> BEL` | 아이콘 이름 설정 (GUI용)    |
| `ESC ] 8;;<URI-VALUE> ST` | 하이퍼링크 (iTerm2, Kitty 등) |

## 참고

### ST (String Terminator)
