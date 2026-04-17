# gnu regex


## regex 규칙

gnu 유틸들에는 ERE를 기본으로 사용하고 다음 특수 문자들을 escape(`\`) 하자

- egrep
- sed -r
- awk(gawk)

Escape 대상 특수 문자

- ERE regex 특수문자: `^` `$` `.` `?` `+` `*` `[` `]` `{` `}` `(` `)` `|`
- gnu 유틸 특수 문자: `!` `\` `/`

## 자주 쓰는 표현

### 표준입력으로 부터 <word>을 포함하는 라인 출력

```text
$ <command> | grep -inI <word>
```

###  regex 패턴과 매칭된 라인과 그 파일 이름을 출력

```bash
grep -rinI '<regex>'
```

###  perl-exp 패턴과 매칭된 라인과 그 파일 이름을 출력

```bash
grep -rinIP '<perl-regex>'
```

###  검색 대상에 지정된 파일만 포함

```bash
grep -rnI --include=<glob-expr file> '<regex>'
grep -rnI --include=\*.java --include=\*.sql 'GET_JOB_SCHEDULE'
```

###  검색 대상에서 파일 제외

```bash
grep -rnI --exclude=<glob-expr file> '<regex>'
```

###  검색 대상에 지정된 디렉터리만 포함

```bash
grep -rnI --include-dir=<glob-expr dir> '<regex>'
```

###  검색 대상에서 디렉토리 제외

```bash
grep -rnI --exclude-dir=<glob-expr dir> '<regex>'
grep -rnI --exclude=\*.js --exclude-dir=.svn --exclude-dir=target 'INTO tb_hello'
```

###  regex 패턴과 매칭된 라인을 갖는 파일 이름만 출력

```bash
grep -rilI '<regex>'
```

### `<perl-regex>` 패턴과 매칭된 부분만 출력

```bash
$ grep -oP '<regex>'
$ grep -oP '(?<={prefix-anchor-pattern}).+?(?={suffix-anchor-pattern})' <path/to/file>

$ echo 'port: 1234' | grep -oP '(?<=^port:\s)\d+(?=\s*$)'
1234

$ echo 'VERSION=3.9.0' | grep -oP '(?<=^VERSION=)[.\d]+(?=\s*$)'
3.9.0
```

###  host ip address만 출력

#### ipv4 주소

```bash
hostname -i | grep -oP '\b\d+(\.\d+){3}\b'
ip -4 a | grep -oP '(?<=inet\s)\d+(\.\d+){3}'
ip -4 a show ens33 | grep -oP '(?<=inet\s)\d+(\.\d+){3}'
```

#### ipv6 주소

```bash
ip -6 a | grep -oP '(?<=inet6\s)[\da-f:]+'
ip -6 a show ens33 | grep -oP '(?<=inet6\s)[\da-f:]+'
```

### whole word 매칭 시도후 매칭 존재 여부만 status code에 남김

```bash
$ grep -qw
$ echo 1q2w3e | grep -wq 1q2w3e; echo $?
0
$ echo 1q2w3e | grep -wq 1q2w2e; echo $?
1
```

## [grep]
grep은 파일 내에서 찾고자 하는 부분 문자열을 검색해 주는 명령어이다.
파일 내부 검색이라는 점에서 파일명을 검색하는 find 명령어와는 다른 차이가 있다.
더불어 grep는 파라미터로 Regex 패턴을 사용한다. 
파일명을 검색하는 find에서 Glob 패턴을 사용하는 것과는 차이가 있으므로 주의한다.

grep은 기본적으로 검색범위로서 표준입력이나 [FILE] 파리미터를 받는다.
만약 파라미터로 파일을 지정하지 않으면 실행시 표준입력을 요구한다.
-R 옵션을 사용하면 검색범위로 [FILE] 파라미터 대신 검색의 시작점이 되는 디렉토리를 지정해야하며 지정하지않으면 현재 디렉토리를 기준으로 삼아 그 하위디렉토리 전부를 검색한다..

```text
grep [OPTION...] PATTERN [FILE...]
        -E        : PATTERN을 확장 정규 표현식(Extended RegEx)으로 평가.
        -F        : PATTERN을 정규 표현식(RegEx)이 아닌 일반 문자열 평가.
        -G        : PATTERN을 기본 정규 표현식(Basic RegEx) 평가.
        -P        : PATTERN을 Perl 정규 표현식(Perl RegEx) 평가.
        -e        : 매칭을 위한 PATTERN 전달.
        -f        : 파일에 기록된 내용을 PATTERN으로 사용.
        -i        : 대/소문자 무시.
        -v        : 매칭되는 PATTERN이 존재하지 않는 라인 선택.
        -w        : 단어(word) 단위로 매칭.
        -x        : 라인(line) 단위로 매칭.
        -z        : 라인을 newline(\n)이 아닌 NULL(\0)로 구분.
        -m        : 최대 검색 결과 갯수 제한.
        -b        : 패턴이 매치된 각 라인(-o 사용 시 문자열)의 바이트 옵셋 출력.
        -n        : 검색 결과 출력 라인 앞에 라인 번호 출력.
        -H        : 검색 결과 출력 라인 앞에 파일 이름 표시. (defalult)
        -h        : 검색 결과 출력 시, 파일 이름 무시.
        -o        : 매치되는 문자열만 표시.
        -q        : 검색 결과 출력하지 않음.
        -a        : 바이너리 파일을 텍스트 파일처럼 처리.
        -I        : 바이너리 파일은 검사하지 않음.
        -d        : 디렉토리 처리 방식 지정. (read, recurse, skip)
        -D        : 장치 파일 처리 방식 지정. (read, skip)
        -r        : 하위 디렉토리 탐색. 심볼릭 링크 제외.
        -R        : 하위 디렉토리 탐색. 심볼릭 링크 포함.
        -L        : PATTERN이 존재하지 않는 파일 이름만 표시.
        -l        : 패턴이 존재하는 파일 이름만 표시.
        -c        : 파일 당 패턴이 일치하는 라인의 갯수 출력.

HELP page)
Usage: grep [OPTION]... PATTERN [FILE]…
Search for PATTERN in each FILE or standard input.
PATTERN is, by default, a basic regular expression (BRE).
Example: grep -i 'hello world' menu.h main.c

Regexp selection and interpretation:
  -E, --extended-regexp          PATTERN is an extended regular expression (ERE)
  -F, --fixed-strings            PATTERN is a set of newline-separated fixed strings
  -G, --basic-regexp             PATTERN is a basic regular expression (BRE)
  -P, --perl-regexp              PATTERN is a Perl regular expression
  -e, --regexp=PATTERN           use PATTERN for matching
  -f, --file=FILE                obtain PATTERN from FILE
  -i, --ignore-case              ignore case distinctions
  -w, --word-regexp              force PATTERN to match only whole words
  -x, --line-regexp              force PATTERN to match only whole lines
  -z, --null-data                a data line ends in 0 byte, not newline

Miscellaneous:
  -s, --no-messages              suppress error messages
  -v, --invert-match             select non-matching lines
  -V, --version                  print version information and exit
      --help                     display this help and exit
      --mmap                     deprecated no-op; evokes a warning

Output control:
  -m, --max-count=NUM            stop after NUM matches
  -b, --byte-offset              print the byte offset with output lines
  -n, --line-number              print line number with output lines
      --line-buffered            flush output on every line
  -H, --with-filename            print the file name for each match (default)
  -h, --no-filename              suppress the file name prefix on output
      --label=LABEL              use LABEL as the standard input file name prefix
  -o, --only-matching            show only the part of a line matching PATTERN
  -q, --quiet, --silent          suppress all normal output
      --binary-files=TYPE        assume that binary files are TYPE;
                                 TYPE is 'binary', 'text', or 'without-match'
  -a, --text                     equivalent to --binary-files=text
  -I                             equivalent to --binary-files=without-match
  -d, --directories=ACTION       how to handle directories;
                                 ACTION is 'read', 'recurse', or 'skip'
  -D, --devices=ACTION           how to handle devices, FIFOs and sockets;
                                 ACTION is 'read' or 'skip'
  -r, --recursive                like --directories=recurse
                                 하위 디렉토리까지 모두 검색
  -R, --dereference-recursive         likewise, but follow all symlinks
                                      -r과  유사하지만 심볼릭 링크까지 전부 따라간다
      --include=FILE_PATTERN        search only files that match FILE_PATTERN
      --exclude=FILE_PATTERN        skip files and directories matching FILE_PATTERN
      --exclude-from=FILE           skip files matching any file pattern from FILE
      --exclude-dir=PATTERN         directories that match PATTERN will be skipped.
  -L, --files-without-match         print only names of FILEs containing no match
  -l, --files-with-matches          print only names of FILEs containing matches
  -c, --count                       print only a count of matching lines per FILE
  -T, --initial-tab                 make tabs line up (if needed)
  -Z, --null                        print 0 byte after FILE name

Context control:
  -B, --before-context=NUM          print NUM lines of leading context
  -A, --after-context=NUM           print NUM lines of trailing context
  -C, --context=NUM                 print NUM lines of output context
  -NUM                              same as --context=NUM
      --color[=WHEN],
      --colour[=WHEN]               use markers to highlight the matching strings;
                                    WHEN is 'always', 'never', or 'auto'
  -U, --binary                      do not strip CR characters at EOL (MSDOS/Windows)
  -u, --unix-byte-offsets           report offsets as if CRs were not there (MSDOS/Windows)
```

### 메타문자 기능

| 메타문자 |  기능 | 사용 예 | 설명 |
|---|---|---|---|
| ^ | 행의 시작 지시자 | '^test' | test로 시작하는 모든 행과 대응함. |
| $ | 행의 끝 지시자 | 'test$' | test로 끝나는 모든 행과 대응함. |
| . | 하나의 문자와 대응 | 't.s.' | 총 4개의 문자로 이루어진 문자열을 검색하는 데, 첫 번째는 't' 세 번째는 's'인 문자열을 모두 대응. |
| * | 선행 문자와 같은 문자 0개 또는 임의 개수와 대응 | 'test*' | 'test'를 기본으로 뒤에 아무 문자나 포함하여도 모두 대응. |
| [] | [] 사이의 문자 집합 중, 문자 하나와 대응    | '[Tt]est'    | 'test' 나 'Test'를 대응함. |
| [^ ] | 문자집합에 속하지 않는 한 문자와 대응 | '[^A-T]est'  | A와 T 사이 범위에 포함되지 않는 한 문자와 est가 붙은 문자열만 검색한다. |
| \< | 단어의 시작 지시자 | '\<test'     | test로 시작하는 단어를 포함하는 행과 대응 |
| \> | 단어의 끝 지시자 | 'test\>'     | test로 끝나는 단어를 포함하는 행과 대응 |
| \(..\) | 다음 사용을 위해서 태그를 붙임.           | '\(tes\)ing' | 지정된 부분을 태그 1에 저장한다. 나중에 태그 값을 참고하려면 \1을 쓴다. 맨 왼쪽부터 시작하여 태그를 9개까지 쓸 수 있다. 왼쪽 예에서는 tes가 레지스터1에 저장되고, 나중에 \1로 참고할 수 있다.  |
| x\{m\} | 문자 x를 m번 반복한다. | 't\{5\}' | 문자 t가 5회 연속으로 나오는 모든 행을 대응함. |
| x\{m,\} | 적어도 m번 반복한다. | 't\{5,\}' | 문자 t가 최소한 5회 반복되는 모든 행과 대응함. |
| x\{m,n\} | m회 이상 n회 이하 반복한다. | t\{5,10\}' | 문자 t가 5회에서 10회 사이의 횟수로 연속으로 나오는 문자열과 대응함. |

### grep 예제

#### 정확히 매칭되는 부분만 출력해야 할 때 : PCRE(-Po)와 Reluctant quantifier(+?) 사용

예) 파일에서 전후방 anchor 사이의 부분만 패턴으로 추출하는 grep 명령어

```bash
 grep -Po '(?<={prefix-anchor-pattern}).+?(?={suffix-anchor-pattern})' <path/to/file>
```

#### 가장 일반적으로 사용해야할 옵션은

표준입력이나 특정 파일 하나를 검색 대상으로 할 때: -inH
하위경로의 파일을 전부 검색 대상으로 할 때: -rinHI / -RinHI

```text
# 주요 옵션
-r, --recursive : 하위 디렉토리 모두 검색, symlink 제외
-R, --dereference-recursive : 하위 디렉토리 모두 검색, symlink 포함
-i, --ignore-case : 대소문자 구문 안함
-l, --file-with-matches : 표준형식(매칭 라인) 출력 하지 않고 파일 이름만 출력
-n, --line number : linenumber 출력
-H, --with-filename : filename 출력
-I, --binary-files=without-match : binary-file에서 검색 안함
--exclude=<glob-pat file-name> : glob 패턴에 매칭 되는 파일을 검색 대상에서 제외
--exclude-dir=<glob-pat dir-name> : glob 패턴에 매칭 되는 디렉토리를 검색 대상에서 제외
    # 예: vcs 경로 제외  --exclude-dir='.svn .git'
```

#### grep multiple patterns, 여러 개의 패턴을 잡을 때

```bash
grep 'fatal\|error\|critical' /var/log/nginx/error.log
```

```bash
# -E, --extended-regexp : to use extended regular expression
grep -E 'fatal|error|critical' /var/log/nginx/error.log

# -P, --perl-regexp

# -i, --ignore-case : to use ignore case 
grep -i 'fatal\|error\|critical' /var/log/nginx/error.log

# -w, --word-regex : to use whole word
grep -w 'fatal\|error\|critical' /var/log/nginx/error.log
```

#### hello.txt 파일내에 hi가 포함된 곳을 검색

```bash
grep "hi" hello.txt
```

#### hello.txt 파일내에 hi를 대소문자 구분없이 검색하고 파일명 라인번호와 함께 출력

```bash
grep -inH "hi" hello.txt
```

#### 현재 디렉토리로 부터 하위 디렉토리로 “uuid”단어를 포함하는 파일명만 찾는다
이때 출력 결과에 라인 넘버와  매칭된 파일 이름도 출력한다..

```bash
grep -RinHI "uuid"
```

#### /home 디렉토리로 부터 하위 디렉토리로 “hi”단어를 포함하는 파일명만 출력

```bash
grep -lR "hi" /home
```

#### 현재 디렉토리 부터 하위 디렉토리 중 "sw"을 포함 하는 라인을 전부 찾는다 
이때 Binary파일도 text type으로 가정한다.

```bash
grep -aiR "sw" ./
```

#### 많은 수의 혹은 대용량인 파일을 대상으로 grep을 사용할 때 명령어 맨 앞에 "LC_ALL=C" 접두어를 붙이면 성능이 6배 정도 향상 된다. grep은 디폴트로 utf-8코드를 모두 검색하는데, 이 접두어를 붙이면 ASCII 코드만 검색한다. Binary file들을 skip하고 싶으면 -I 옵션 사용을 고려하라.

```bash
LC_ALL=C grep -rin “orange” ./pantry
```

#### 위 명령어보다 더 빠를수 있는것은 grep 대신 fgrep을 사용하는 것이다.

```bash
LC_ALL=C fgrep -rni “orange” ./pantry
```

#### 특정 단어를 제외하는 옵션 -v, -Ev

여러 단어를 제외해야할 때 -v 옵션을 반복하여 지정하는 방법

```bash
grep Hello a | grep -v apple|grep -v orange| grep -v banana
```

여러 단어를 제외해야할 때 -Ev 옵션으로 제외할 단어들을 한번에 지정하는 방법

```bash
grep Hello a | grep -Ev 'apple|orange|banana'
```

#### word 바운더리를 지정하여 검색하는 방법 (문서 편집기의 Word 지정 옵션과 같은 효과)

```bash
grep -i '\<block\>' test.txt
```

위 명령어는 test.txt 파일 내에서 block이라는 단어를 검색하는데
`-i` 옵션으로 case-insensitive를 지정하고
block 앞뒤에 \< \> 기호(word boundary)를 명시 함으로써 block이라는 단어만을 검색하도록 한다.
'\<' 는 start of word, '\>' 는 end of word를 의미하는 anchor이다.
regex의 word는 \w = [a-zA-Z_0-9] 문자로만 이루어지며 이외 문자는 delimeter가 된다.

#### 라인이 아닌 특정 패턴으로 시작하는 단어만 출력

```bash
egrep -o '\<word[^ ]*\>' test.txt
```

주의) '[^ ]*' 대신 '.*' 을 사용하면 매칭 문자열은 공백문자를 포함하게 되어 라인의 끝까지 매치하게 된다.

#### 전후방탐색 - 탐색 대상의 prefix/suffix anchor를 지정할 때 사용

```text
# syntax
(?<={prefix}){search target}(?={suffix})
```

예) 파일에서 전후방 anchor 사이의 부분만 패턴으로 추출하는 grep 명령어

```bash
grep -Po '(?<={prefix-anchor-pattern}).+?(?={suffix-anchor-pattern})' <path/to/file>
```

-P, --perl-regexp, 주의 -E, --extended-regexp 옵션에서는 전후방탐색이 정상 적용되지 않는다.
-o, --only-matching

예) json에서 mykey에서 대한 string value를 획득하는 명령어

```bash
grep -Po '(?<="mykey":").+?(?=")' my.json
```

예) 라인에서 숫자만 추출하기
패턴 대상라인: "  port: 8823  ", "  port: 8823"

```bash
$ echo "  port: 8823" | grep -P '^\s+port:\s\d+$'
  port: 8823
$ echo "  port: 8823" | grep -Po '(?<=port:\s)\d+(?=\s*$)'
8823
```

주의) lookbehind(?<=) 표현식에서는 변경 길이 수량자(?,*,+)를 사용할 수 없고 고정길이 수량자만 사용 가능

```text
-P : perl expression 사용 (\s, \d 등 사용 가능)
-o : 매칭 결과 문자열만 출력
```

## grep에 사용되는 glob syntax

grep은 표준입력의 각 라인에 대해 'PATTERN'을 포함하는 라인을 찾는 역할을 한다.

### Pattern syntax 선택 옵션

```plain
-G, default, BREs(basic regular expressions)
```

'PATTERN'에 기본 regex 사용,
    regex 메타문자에 대해 escape(preceded by \) 필요

```plain
-E    EREs(extended regular expressions)
```

'PATTERN'에 확장 regex 사용(--extened-regexp),
    regex 메타문자에 대해 escape(preceded by \) 불필요

```plain
# egrep
-F    'PATTERN'에 regex 사용 안함(--fixed-strings), 'PATTERN'의 문자열 literal 그대로 매칭 시도
```

```plain
# fgrep
-P    PCREs(Perl-compatible regular expressions)
```

복잡한 패턴을 작성 할 때 권장
가장 효율적인 표현과 모든 기능을 갖추고 있다.

### egrep / grep -E / grep --extened-regexp

- egrep은 grep 명령어에 -E 또는 --extened-regexp 옵션을 준 것과 같다.
- 옵션 풀네임을 보면 알 수 있듯이 확장 정규표현식을 사용하기 위한 명령이다.
- 확장 정규표현식에 쓰이는 메타문자를 사용할 때 각각의 문자에 escape 처리를 해주지 않아도 된다.

아래 코드를 보자.

```bash
egrep 'no(escape|character)' someFile 
grep 'no\(escape\|character\)' someFile
```

someFile 이라는 파일에서 noescape 또는 nocharacter 라는 
문자열 둘 중 하나를 검색한다는 정규표현식을 사용할 때, 
확장 정규표현식(egrep)에서는 그냥 사용하면 되지만 
기본 정규표현식(grep)에서는 메타문자 각각에 대하여 \ 문자를 통해
escape 처리를 해주어야 동작하게 된다.

#### egrep에서 추가된 메타문자들

```plain
+ : (수량자) 바로 앞의 문자가 1 이상
? : (수량자) 바로 앞의 문자가 0 or 1
a|b : a 또는 b와 매칭되는 문자(or)
() : 문자 그룹
```

### fgrep / grep -F / grep --fixed-strings

egrep이 기본 grep의 정규표현식을 확장하는 개념이라면 
fgrep은 정규표현식을 사용하지 않는다고 선언해주는 것과 같다.
따라서 정규표현식에서 메타문자로 사용되는 문자들이 그 자체로서 패턴 비교에 활용된다.

```bash
fgrep 'self.' * 
grep 'self.' *
```

위 코드를 보면 fgrep과 grep 명령어 모두 self. 라는 문자열을 검색하고 있다.
fgrep에서는 순수하게 'self.' 이라는, self 뒤에 . 문자를 포함한 문자열만을 검색하는데 반해
grep에서는 self. 뿐만이 아니라 selfi, selfy, selfa 등 .이 문자 1개를 의미하는 메타문자로서 동작하여 패턴 비교에 활용된다.

### 샘플 -  nic의 IP address 획득

`<devname>` : nic 이름. eth0, ens192 등

```bash
ip -4 addr show <devname> | grep -Po '(?<=inet\s)\d+(\.\d+){3}'
echo $(hostname)@$(ip -4 addr show <devname> | grep -Po '(?<=inet\s)\d+(\.\d+){3}')
```
