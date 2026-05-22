# 정규표현식(regular expression) 정리

## 참고

### Standard / Integration
- https://www.regular-expressions.info/

### POSIX BRE/ERE standard
- https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap09.html

### POSIX BRE
- https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap09.html#tag_09_03

### POSIX ERE
- https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap09.html#tag_09_04

### PCRE2 standard
- https://www.pcre.org/

### PCRE2 man page
- https://perldoc.perl.org/perlre

### PCRE2 syntax
- https://www.pcre.org/current/doc/html/pcre2syntax.html

### Java Regular Expressions
- http://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html
- https://docs.oracle.com/javase/tutorial/essential/regex/index.html

### Google RE2 Syntax
- https://github.com/google/re2/wiki/Syntax


## Test

### Online regex tester and debugger
- https://regex101.com — 복잡한 정규 표현식은 이 사이트를 통하여 테스트와 분석을 실행한다.
- http://www.regexr.com/

### POSIX ERE
- https://www.boost.org/doc/libs/1_38_0/libs/regex/doc/html/boost_regex/syntax/basic_extended.html

### Ruby 기반
- http://rubular.com/

### Java 기반
- http://www.regexplanet.com/advanced/java/index.html

### 정규식의 시각화
- http://www.regexper.com/

### 정규식 추천해주는 사이트
- http://txt2re.com/index.php3


---

## 정규표현식?

정규 표현식은 텍스트 중 원하는 패턴(문자 혹은 문자열의 집합 형식)에 일치하는 부분을 검색, 치환, 추출 하기 위하여 사용하는 형식 언어이다.  
정규식은 POSIX 표준이 있고 이를 베이스로 하지만 각 언어별로 조금씩 차이가 있으므로 각 언어별 표준문서를 보는 것이 올바르다.  
다시 말해 프로그래밍 언어별로 기본 문법은 유사하나 확장된 문법(수량자 및 predefined character)에 차이가 있다.  
그러므로 정규 표현식 사용 시 프로그래밍 언어별로 문법의 유효성에 대해 반드시 검증을 요한다.  
정규표현식은 기본적으로 **case sensitive** 이다.

다음과 같은 specification hierarchy를 가짐:

```
POSIX Basic Regular Expression (BRE)
  -> POSIX Extended Regular Expression (ERE)
    -> Perl compatible regular expression (PCRE2), 각 프로그래밍 언어
```

기본적으로 PCRE, ERE 를 사용하고 특수 문자에 대해서 escape 없이 사용한다.  
일반 텍스트로서의 특수 문자가 많이 사용될 경우에만 BRE 사용 고려만 해본다.


### BRE, ERE 차이

BRE와 ERE는 특수문자에 대한 `\` escape 대상에서 차이가 있음

- **ERE**: 특수문자를 literal로 인식하고자 할 때만 `\` escape 사용
- **BRE**: 문자별로 특수문자 또는 literal로 인식하기 위해 `\` escape를 혼용함으로써 혼란의 여지가 많음

#### BRE, ERE escape 차이 있는 특수 문자

```
? + | ( ) { }
```

위 문자들이 regex 특수 문자로 인식되기 위해서:
- **BRE**: escape 필요
- **ERE**: escape 불필요

#### BRE, ERE escape 불필요한 공통 특수 문자

```
^ $ * . [ \
```

다음 문자들은 공통적으로 escape 하지 않고 사용하는 regexp 특수 문자이며 일반 텍스트 문자로 사용하려면 BRE, ERE 모두에서 `\` 를 이용해 escape 해야 한다.  
`[` 와 `]` 를 일반 문자로 사용하려면 모두 escape 해야 한다 (`\[`, `\]`). 단, `]` 가 문자 클래스의 첫 번째 문자인 경우 escape 하지 않아도 된다 (예: `[]abc]` = `]`, `a`, `b`, `c` 문자).


---

## 용어

### backoff (=giving back, backtracking)

수량자(quantifier)의 타입 중 탐욕적(Greedy) 수량자의 탐색 방법을 설명할 때 사용되는 용어  
`abcde`라는 문자열에서 매칭을 시도할 때 한 문자씩 뒤에서 앞으로 기준을 옮기며 체크하는 것을 **backoff**라고 한다.

```
try1 : abcde
try2 : abcd
try3 : abc
try4 : ab
try5 : a
```

`abcde`라는 문자열에 탐색을 할 때 문자 `a`에서 문자 `e`로 진행하는 방향을 **forward** 라고 한다면  
이와 반대로 문자 `e`에서 문자 `a`로 진행하는 방향을 **backward**라고 하며  
문자열의 각 문자를 탐색할 때 커서를 backward로 한 문자 이동하는 것을 **backoff** 라고 한다.


### Greedy, Reluctant, Possessive

수량자(`Quantifier`, `?`/`+`/`*`/`{}`)의 매칭 속성을 표현하는 용어  
수량자의 뒤에 붙여 쓰는 수량자 속성 표현

#### 탐욕적(Greedy) 수량자 : 수량자에 매칭 속성 표현이 아무것도 붙지 않은 경우 (default)

> !! wide-range + back off (last index backward move)  
> try matching as possible as wide range with back off (giving back)  
> 매칭 시도 시 최대 범위부터 시작하여 backoff하며 재 매칭 시도

wide-range란 첫 매칭 시도 범위가 최대라는 의미로써 Greedy 수량자 속성은 매칭 범위를 정할 때 최초 시도 범위의 마지막 기준 문자를 탐색 대상 문자열의 마지막 문자로 하여 최대범위로 시도하고 이후 범위의 마지막 기준 문자를 첫문자 방향으로 좁히면서 매칭을 시도하는 것을 말한다.

#### 게으른/나태한(Reluctant) 수량자 : `?` 수량자 속성을 붙이는 경우 (`??`/`+?`/`*?`/`{}?`)

> !! narrow-range + last index forward move  
> try matching as possible as narrow range  
> 매칭 시도 시 최소 범위부터 시작하여 forward로 기준을 변경하며 재 매칭 시도

narrow-range란 첫 매칭 시도 범위가 최소라는 의미로써 Reluctant 수량자는 Greedy와는 반대로  
첫 매칭 시도의 범위의 마지막 문자를 첫 문자의 다음 문자로 최소 범위 매칭 시도하고  
이후에 매칭 기준 마지막 문자를 forward 방향으로 변경하며 재매칭을 시도하는 수량자를 말함

Reluctant 수량자에서 매칭 retry는 예를 들어 아래와 같이 진행한다:

```
try1 : a
try2 : ab
try3 : abc
try4 : abcd
try5 : abcde
```

#### 독점적(Possessive) 수량자 : `+` 수량자 속성을 붙이는 경우 (`?+`/`++`/`*+`/`{}+`)

> !! wide-range + no retry (1 time matching try)  
> try just one(1) time wide-range matching without back off (giving back)  
> 매칭 시도 시 최대 범위로 1회만 하고 다시 try하지 않는 수량자


### capturing group

문자 그룹 지정

| 문법 | 설명 |
|------|------|
| `(regex)` | numbered capturing group. 역참조 시 grouping 순서에 따라 지정된 번호로 참조 가능 |
| `(?<group name>regex)` | named capturing group. 역참조 시 지정된 group 이름으로 참조 가능 |
| `(?'group name'regex)` | named capturing group (대안 문법) |
| `(?:regex)` | non-capturing group. 역참조 불가능하며 문자 그룹에 수량자나 or 연산자를 붙이기 위해 사용 |


### 역참조

numbered/named capturing group을 통해서 매칭된 실제 값을 regex에서 다시 참조하는 것

역참조는 `\` 문자를 prefix로 하여 표현한다.
- numbered capturing group: `\<number>` 로 역참조
- named capturing group: `\<name>` 로 역참조

역참조 시 `\` 문자는 환경에 따라 `\\` 형태로 escaping 해야 할 수도 있다.  
예를 들어 shell이나 java에서는 `\` 문자가 특수한 의미를 갖기 때문에 이를 escaping하기 위해 역참조 표현 시 `\\` 를 사용해야 한다.

#### search regex에서 역참조

search expression에서 역참조를 하는 것은 단일 regex 표현에서 역참조 하는 것을 말한다.

```
탐색 대상 문자열: "abcda"
Regex Pattern: "(.).+\\1"

설명:
(.) - 첫번째 문자 'a'를 그룹핑한다. 이때 (를 처음 사용했기 때문에 1번 그룹이된다.
      참고로 0번 그룹은 항상 대상 문자열 전체가 된다.
.+  - 어떤문자가 1개 이상, 'bcd'가 대응된다.
\\1 - 1번 capturing group의 역참조. 1번 그룹이 해당 위치에 다시 나타난다는 것을 의미.
      이 예에서는 1번 capturing group 'a'가 다시 나타남을 의미
```

#### replace regex에서 역참조

replace에서 regex는 탐색 expression과 변경 expression 모두에서 사용될 수 있다.  
예를 들어 `1. 2. 3. 4.` 의 문자열을 `1, 2, 3, 4,` 로 dot(.)을 comma(,)로 바꿔야 하는 경우:

- 탐색 expression: `([0-9])\.`
- 변경 expression: `\1,`


### boundary

word와 매칭을 할 때 word의 시작 또는 끝이 어떤 문자가 아닌 것일 때, 이를 구분하기 위한 표현

| 기호 | 설명 |
|------|------|
| `^` | 라인의 시작 |
| `$` | 라인의 끝 |
| `\b` | word boundary, `\w`(word)와 `\W`(non-word) 사이를 의미. word의 시작과 끝으로 모두 사용 가능 |
| `\<` | word boundary, start of word |
| `\>` | word boundary, end of word |


---

## Syntax

```
<start-anchor>"<character-pattern><quantifier>"...<end-anchor>
```

**구성요소:**

- **문자패턴**: 수량자의 적용 대상이 되는 문자 또는 문자열  
  `.`, 문자, 문자클래스`[]`, 문자그룹`()`

- **수량자**: 문자패턴의 반복 횟수를 명시하기 위한 패턴 문자  
  `+`, `*`, `?`, `{n}`, `{min,}`, `{min, max}`

- **Anchor**: 라인 안에서 찾는 문자열의 위치를 지정하기 위한 기준 패턴들  
  Boundary matchers(`^`, `$`, `\b`, `\B`, `\A`, `\G`, `\z`, `\Z`), 전/후방탐색의 `<criterion pattern>`


---

## 정규표현식 쉬운 작성 방법

> 우선 매치할 입력값의 예문을 만들고 해당 예문에서  
> 원하는 위치의 문자만 연산의 목적에 맞춰서 메타문자로 변경해보자.


---

## 정규표현식 패턴의 작성 기본

기본적으로 PCRE나 ERE를 사용하고 특수문자에 대해서 escape 없이 사용한다.  
일반 텍스트로 사용해야 할 특수문자가 문자열의 경우에만 BRE 사용 고려한다.

정규 표현식 문법의 모든 시작은 매칭 단위(문자, 문자클래스`[]`, 문자그룹`()`)에 수량자를 붙이는 것이다.  
이러한 패턴에 특정 위치를 지정하기 위해 Boundary Constructor(anchor)를 사용하고  
소비(Consume - 패턴에 매치된 결과를 패턴으로 재사용)하기 위해 Named Grouping을 사용한다.

**작성순서:**
1. 찾고자 하는 최소단위 부분열과 매치되는 작은 단위의 패턴(small-pattern)들을 작성
2. small-pattern에 수량자(quantifier)를 적용
3. 필요 시 Grouping 지정
4. Boundary 지정 - Boundary-matcher 또는 전후방 탐색(lookaround)을 적용

**매칭 단위:**

수량자를 붙일 수 있는 매칭 단위:
- 문자 (단일 문자)
- 문자 클래스 `[]` — 문자들 사이에 or 연산 수행
- Non-Capturing 문자그룹 `(?:)` — 문자열 지정, 문자열 사이에 or 연산 수행
- Capturing 문자그룹 `()` — 문자열 지정, 추후 역참조에 사용

**regex에서 수량자를 붙일 수 있는 문자 단위:**

**1. 문자 (문자 1개)**

| 표현 | 설명 |
|------|------|
| `x` | 문자 x |
| `\x` | predefined special character x |
| `\t` | TAB character |
| `\n` | NewLine character |

**2. 문자 클래스 (문자 1개)**

| 표현 | 설명 |
|------|------|
| `[abc]` | a, b, c 문자 중 하나 |
| `[^abc]` | a, b, c 문자가 아닌 문자 하나 |
| `.` | 임의의 문자 1개 |
| `\d` | 숫자 1개 |
| `\w` | word문자 `[a-zA-Z_0-9]` 1개 |
| `\s` | whitespace 문자 `[ \t\n\x0B\f\r]` 1개 |

**3. 문자 그룹 (문자열 패턴 - 2자 이상의 문자)**

```
(hello)    문자열 hello

grouping은 "(" 순서에 따라서 번호를 할당 받음.

(hello (new))(world)
  group0: hello new world
  group1: hello new
  group2: new
  group3: world
```

역참조는 `\<group-number>` 사용


---

## Summary of regular-expression constructs

> 한글 내용은 표 이하 설명 아래 참조

### Characters

| Construct | Matches |
|-----------|---------|
| `x` | The character x |
| `\\` | The backslash character |
| `\0n` | The character with octal value 0n (0 <= n <= 7) |
| `\0nn` | The character with octal value 0nn (0 <= n <= 7) |
| `\0mnn` | The character with octal value 0mnn (0 <= m <= 3, 0 <= n <= 7) |
| `\xhh` | The character with hexadecimal value 0xhh |
| `\uhhhh` | The character with hexadecimal value 0xhhhh |
| `\x{h...h}` | The character with hexadecimal value 0xh...h |
| `\t` | The tab character (`\u0009`) |
| `\n` | The newline (line feed) character (`\u000A`) |
| `\r` | The carriage-return character (`\u000D`) |
| `\f` | The form-feed character (`\u000C`) |
| `\a` | The alert (bell) character (`\u0007`) |
| `\e` | The escape character (`\u001B`) |
| `\cx` | The control character corresponding to x |

### Character Classes

| Construct | Matches |
|-----------|---------|
| `[abc]` | a, b or c (simple class) |
| `[^abc]` | Any character except a, b or c (negation) |
| `[a-zA-Z]` | a through z or A through Z, inclusive (range) |
| `[a-d[m-p]]` *(Java only)* | a through d, or m through p: `[a-dm-p]` (union) |
| `[a-z&&[def]]` *(Java only)* | d, e, or f (intersection) |
| `[a-z&&[^bc]]` *(Java only)* | a through z, except for b and c: `[ad-z]` (subtraction) |
| `[a-z&&[^m-p]]` *(Java only)* | a through z, and not m through p: `[a-lq-z]` (subtraction) |

### Predefined Character Classes

| Construct | Matches |
|-----------|---------|
| `.` | Any character (`\.`: line terminators) |
| `\d` | A digit: `[0-9]` |
| `\D` | A non-digit: `[^0-9]` |
| `\s` | A whitespace character: `[ \t\n\x0B\f\r]` |
| `\S` | A non-whitespace character: `[^\s]` |
| `\w` | A word character: `[a-zA-Z_0-9]` |
| `\W` | A non-word character: `[^\w]` |

### POSIX Character Classes (US-ASCII only) *(Java only)*

| Construct | Matches |
|-----------|---------|
| `\p{Lower}` | A lower-case alphabetic character: `[a-z]` |
| `\p{Upper}` | An upper-case alphabetic character: `[A-Z]` |
| `\p{ASCII}` | All ASCII: `[\x00-\x7F]` |
| `\p{Alpha}` | An alphabetic character: `[\p{Lower}\p{Upper}]` |
| `\p{Digit}` | A decimal digit: `[0-9]` |
| `\p{Alnum}` | An alphanumeric character: `[\p{Alpha}\p{Digit}]` |
| `\p{Punct}` | Punctuation: One of `!"#$%&'()*+,-./:;<=>?@[\]^_{|}~` |
| `\p{Graph}` | A visible character: `[\p{Alnum}\p{Punct}]` |
| `\p{Print}` | A printable character: `[\p{Graph}\x20]` |
| `\p{Blank}` | A space or a tab: `[ \t]` |
| `\p{Cntrl}` | A control character: `[\x00-\x1F\x7F]` |
| `\p{XDigit}` | A hexadecimal digit: `[0-9a-fA-F]` |
| `\p{Space}` | A whitespace character: `[ \t\n\x0B\f\r]` |

### java.lang.Character Classes *(Java only)*

| Construct | Matches |
|-----------|---------|
| `\p{javaLowerCase}` | Equivalent to `java.lang.Character.isLowerCase()` |
| `\p{javaUpperCase}` | Equivalent to `java.lang.Character.isUpperCase()` |
| `\p{javaWhitespace}` | Equivalent to `java.lang.Character.isWhitespace()` |
| `\p{javaMirrored}` | Equivalent to `java.lang.Character.isMirrored()` |

### Classes for Unicode scripts, blocks, categories and binary properties *(Java only)*

| Construct | Matches |
|-----------|---------|
| `\p{IsLatin}` | A Latin script character (script) |
| `\p{InGreek}` | A character in the Greek block (block) |
| `\p{Lu}` | An uppercase letter (category) |
| `\p{IsAlphabetic}` | An alphabetic character (binary property) |
| `\p{Sc}` | A currency symbol |
| `\P{InGreek}` | Any character except one in the Greek block (negation) |
| `[\p{L}&&[^\p{Lu}]]` | Any letter except an uppercase letter (subtraction) |

### Boundary Matchers

| Construct | Matches |
|-----------|---------|
| `^` | The beginning of a line after newline |
| `$` | The end of a line before newline |
| `\<` | Start of word (`\w = [a-zA-Z_0-9]`) |
| `\>` | End of word (`\w = [a-zA-Z_0-9]`) |
| `\b` | A word boundary (between `\w` and `\W`) |
| `\B` | A non-word boundary |
| `\A` | The beginning of the input |
| `\G` | The end of the previous match |
| `\Z` | The end of the input but for the final terminator, if any |
| `\z` | The end of the input |

> regex로 검색 패턴 입력 시 하나의 문장은 `^`와 `$` 사이에 위치하지만  
> 줄바꿈 문자(`\r`, `\n`)은 `$` 뒤나 `^` 앞에 위치한다.  
> `^<line>$` / `$\r\n^`

**`\<` 예:**
```
\<test        test로 시작하는 단어
\<WO.*RD\>    WO로 시작하고 RD로 끝나는 단어
```

**`\b` 예:**
```
\bWORD\b       [공백|특수문자]로 둘러싸인 WORD 단어
\bWO.*RD\b     [공백|특수문자]로 둘러싸인 WO로 시작하고 RD로 끝나는 단어
```

lookbehind와 lookahead로 word-boundary 대체 표현:  
`(?<=\W)(?=\w) WORD (?<=\w)(?=\W)`

### Greedy Quantifiers

> try matching as possible as wide range with back off (giving back)  
> !! wide-range + back off (last index backward move)

| Construct | Matches |
|-----------|---------|
| `X?` | X, once or not at all (0\|1) |
| `X*` | X, zero or more times (0+) |
| `X+` | X, one or more times (1+) |
| `X{n}` | X, exactly n times |
| `X{n,}` | X, at least n times |
| `X{n,m}` | X, at least n but not more than m times |

### Reluctant Quantifiers

> try matching as possible as narrow range  
> !! narrow-range + last index forward move  
> 수량자(quantifiers) 뒤에 `?` 추가

| Construct | Matches |
|-----------|---------|
| `X??` | X, once or not at all |
| `X*?` | X, zero or more times |
| `X+?` | X, one or more times |
| `X{n}?` | X, exactly n times |
| `X{n,}?` | X, at least n times |
| `X{n,m}?` | X, at least n but not more than m times |

### Possessive Quantifiers

> try just one(1) time matching without back off (giving back)  
> !! wide-range + no retry (1 time matching try)  
> 수량자(quantifiers) 뒤에 `+` 추가

| Construct | Matches |
|-----------|---------|
| `X?+` | X, once or not at all |
| `X*+` | X, zero or more times |
| `X++` | X, one or more times |
| `X{n}+` | X, exactly n times |
| `X{n,}+` | X, at least n times |
| `X{n,m}+` | X, at least n but not more than m times |

### Logical Operators

| Construct | Matches |
|-----------|---------|
| `XY` | X followed by Y |
| `X\|Y` | Either X or Y |
| `(X)` | X, as a capturing group |
| `(XX\|YY\|ZZ)` | OR group |

### Back References

| Construct | Matches |
|-----------|---------|
| `\n` | Whatever the nth capturing group matched |
| `\k<name>` | Whatever the named-capturing group "name" matched |

### Quotation

| Construct | Matches |
|-----------|---------|
| `\` | Nothing, but quotes the following character |
| `\Q` | Nothing, but quotes all characters until `\E` |
| `\E` | Nothing, but ends quoting started by `\Q` |

### Special Constructs (named-capturing and non-capturing)

| Construct | Matches |
|-----------|---------|
| `(?<name>X)` | X, as a named-capturing group |
| `(?:X)` | X, as a non-capturing group |
| `(?>X)` | X, as an independent, non-capturing group |
| `(?=X)` | X, via zero-width positive lookahead |
| `(?!X)` | X, via zero-width negative lookahead |
| `(?<=X)` | X, via zero-width positive lookbehind |
| `(?<!X)` | X, via zero-width negative lookbehind |
| `(?idmsuxU-idmsuxU)` *(Java only)* | Nothing, but turns match option flags on/off |
| `(?idmsux-idmsux:X)` *(Java only)* | X, as a non-capturing group with the given option flags |

> `(?i)<title>(?-i)foo(?i)</title>(?-i)` → `<TITLE>foo</TITLE>` 매칭  
> `(?i:<title>)foo(?i:</title>)` → `<TITLE>foo</TITLE>` 매칭


---

## Java Option Flags

| Flag | 설명 |
|------|------|
| `(?d)` UNIX_LINES | `\n`만 line terminator로 인식 (`.`, `^`, `$` 동작에 영향) |
| `(?i)` CASE_INSENSITIVE | 대소문자 구분 없는 매칭 |
| `(?x)` COMMENTS | whitespace와 `#` 주석 허용 |
| `(?m)` MULTILINE | `^`, `$`가 각 줄의 시작/끝에 매칭 |
| `(?s)` SINGLELINE (DOTALL) | `.`이 줄바꿈 문자까지 포함 |
| `(?u)` UNICODE_CASE | Unicode 기반 대소문자 구분 없는 매칭 |
| `(?U)` UNICODE_CHARACTER_CLASS | Predefined/POSIX character class를 Unicode 기준으로 동작 |


## PCRE2 Option Setting

> Changes of these options within a group are automatically cancelled at the end of the group.

| Flag | 설명 |
|------|------|
| `(?i)` | caseless |
| `(?J)` | allow duplicate named groups |
| `(?m)` | multiline |
| `(?n)` | no auto capture |
| `(?s)` | single line (dotall) |
| `(?U)` | default ungreedy (lazy) |
| `(?x)` | extended: ignore white space except in classes |
| `(?xx)` | as `(?x)` but also ignore space and tab in classes |
| `(?-...)` | unset option(s) |
| `(?^)` | unset imnsx options |


---

## 메타문자

정규 표현식의 문자 패턴을 정의하기 위해 사전에 약속된 의미를 가지며 일반 문자로 취급되지 않는다.  
정규 표현식에서 메타문자와 동일한 문자를 일반 문자로서 명시하고자 할 경우 `\` 문자를 앞에 붙이거나 `[]` 안에 담아서 이스케이프 해주어야 한다.

**메타 문자 이스케이프하기 예:**
```
\$ 또는 [$]
\^ 또는 [^]
```

> `\` 기호를 이용하는 escaping의 경우 거의 모든 언어에서 지원하지만(권장)  
> `[]` character class를 이용하여 escaping하는 방법은 언어마다 가능한 문자의 차이가 있으므로 사용을 지양한다.

| 메타문자 | 기능 | 설명 |
|----------|------|------|
| `.` | 문자 (character) | 1개의 문자(single character)와 일치. 단일행 모드에서는 새줄 문자 제외 |
| `[]` | 문자 클래스 (character class) | `[` 와 `]` 사이의 문자 중 하나 선택. `-`와 함께 쓰면 범위 지정 가능 |
| `[^ ]` | 부정 (역집합) | 문자 클래스 안의 문자를 제외한 나머지 선택 |
| `+` | 탐욕적 수량자 (Greedy) | 1회 이상, `{1,}` |
| `*` | 탐욕적 수량자 (Greedy) | 0회 이상, `{0,}` |
| `?` | 탐욕적 수량자 (Greedy) | 0 또는 1회, `{0,1}` |
| `{n}` | 탐욕적 수량자 (Greedy) | 정확히 n개 |
| `{min,}` | 탐욕적 수량자 (Greedy) | min개 이상. **주의: `{,max}` 표현은 없다!** |
| `{min,max}` | 탐욕적 수량자 (Greedy) | 최소 min개, 최대 max개 |
| `+?`, `*?`, `??`, `{}?` | 게으른 수량자 (Reluctant = Lazy) | 최소 범위에서 최대 범위로 확장하며 매칭 |
| `++`, `*+`, `?+`, `{}+` | 독점적 수량자 (Possessive) | 완전체 입력 문자열 대상으로 1회만 매칭 시도 |
| `\|` | OR (선택) | 문자열 사이에 or 연산. 주로 `()` 로 감싸 사용 |
| `&&` | AND (선택) | **Java only**. 문자 클래스에서 and 연산. `[a-e&&[def]]` = d, e |
| `(regex)` | Capturing group | `()`안의 패턴을 하나의 유닛으로 지정. 역참조 가능 |
| `(?<name>regex)` | Named-capturing group | 번호 대신 이름으로 역참조 |
| `(?'name'regex)` | Named-capturing group (대안 문법) | |
| `(?:regex)` | Non-capturing group | 역참조 불가. 수량자/or 연산 적용을 위한 그룹화 |
| `(?>regex)` | Atomic group | non-capturing + Possessive 속성 (backoff 없음) |
| `<fp>(?=<cp>)` | 긍정형 전방탐색 (positive lookahead) | `<fp>`가 `<cp>`에 의해 뒤따를 때 `<fp>`만 소비 |
| `<fp>(?!<cp>)` | 부정형 전방탐색 (negative lookahead) | `<fp>` 뒤에 `<cp>`가 없을 때 `<fp>`만 소비 |
| `(?<=<cp>)<fp>` | 긍정형 후방탐색 (positive lookbehind) | `<fp>` 앞에 `<cp>`가 있을 때 `<fp>`만 소비 |
| `(?<!<cp>)<fp>` | 부정형 후방탐색 (negative lookbehind) | `<fp>` 앞에 `<cp>`가 없을 때 `<fp>`만 소비 |
| `(?idmsuxU-idmsuxU)` | Option Flags | 매치 옵션 on/off (언어별 차이 주의) |
| `(?idmsux-idmsux:X)` | Option Flags | 옵션 적용 non-capturing group |
| `\v` | ASCII character | ASCII(0x0B) / VT (Vertical Tab) / 수직 탭 |
| `\n` | ASCII character | NL (New Line). OS별로 실제 키값 다를 수 있음 |
| `\f` | ASCII character | ASCII(0x0C) / FF (Form Feed) / 폼피드 |
| `\r` | ASCII character | ASCII(0x0D) / CR (Carriage Return) / 캐리지 리턴 |
| `\t` | ASCII character | ASCII(0x09) / HT (Horizon Tab) / 탭 |
| `.` | Predefined Character Class | Any Character |
| `\d` | Predefined Character Class | A digit: `[0-9]` |
| `\D` | Predefined Character Class | A non-digit: `[^0-9]` |
| `\s` | Predefined Character Class | Whitespace: `[ \t\n\v\f\r]` |
| `\S` | Predefined Character Class | Non-whitespace: `[^\s]` |
| `\w` | Predefined Character Class | Word character: `[a-zA-Z_0-9]` (*주의: `_` 포함*) |
| `\W` | Predefined Character Class | Non-word character: `[^\w]` |
| `^` | Boundary Construct | 한 줄의 시작. *단, `[]`안에서 사용되면 부정(역집합)의 의미* |
| `$` | Boundary Construct | 한 줄의 끝 |
| `\b` | Boundary Construct | Word boundary (`\w`와 `\W` 사이) |
| `\B` | Boundary Construct | Non-word boundary |
| `\A` | Boundary Construct | The beginning of the input (입력값 전체의 시작) |
| `\G` | Boundary Construct | The end of the previous match |
| `\Z` | Boundary Construct | The end of the input but for the final terminator, if any |
| `\z` | Boundary Construct | The end of the input |

> **주의: EOL(End-of-line) 표현은 OS 마다 다름**
> - Windows: `\r\n` (ASCII 0x0D 0x0A)
> - Linux: `\n` (ASCII 0x0A)
> - Mac: `\n` (실제 키값은 CR 0x0D)

**괄호 기호 메타문자의 종류와 사용 용도:**

| 기호 | 이름 | 용도 |
|------|------|------|
| `()` | parenthesis | character group 지정 `(abc)` |
| `[]` | (square)brackets | character class 지정 `[a-z]`, POSIX pre-defined character `[:alpha:]` |
| `{}` | braces | quantifier 명시 `{min,max}`, Java pre-defined character `{Alpha}` |

**메타문자 이용의 예:**
```
.*           임의의 문자구간
^.*$         임의의 문자열
abc\s+.+[^\r\n]    abc 이후 공백으로 시작하는 문자열에서 줄바꿈 부분을 제외한 나머지
```


---

## 수량자 (Quantifier)

정규 표현식의 패턴-유닛(문자, 문자클래스, 그룹) 뒤에  
`+`, `*`, `{}`를 사용하여 앞에 나온 패턴-유닛이 반복되는 수를 지정하는 메타 문자를 수량자라고 한다.

```
a+       'a' 문자가 1회 또는 그 이상 반복. 단일 문자 대상
[a-z]+   'a'부터 'z'까지 중 단일 문자가 1회 또는 그 이상 반복. 문자 클래스 대상
(abc)+   'abc' 문자그룹이 1회 또는 그 이상 반복. 문자그룹 대상
```

수량자에 다시 `?`를 덧붙인 `??`, `*?`, `+?`, `{}?` → **Reluctant (= Lazy) Quantifier**  
수량자에 다시 `+`를 덧붙인 `?+`, `*+`, `++`, `{}+` → **Possessive Quantifier**

**자바에서의 수량자 표현:**

| Greedy | Reluctant | Possessive | Meaning |
|--------|-----------|------------|---------|
| `X?` | `X??` | `X?+` | X, once or not at all |
| `X*` | `X*?` | `X*+` | X, zero or more times |
| `X+` | `X+?` | `X++` | X, one or more times |
| `X{n}` | `X{n}?` | `X{n}+` | X, exactly n times |
| `X{n,}` | `X{n,}?` | `X{n,}+` | X, at least n times |
| `X{n,m}` | `X{n,m}?` | `X{n,m}+` | X, at least n but not more than m times |

**Greedy vs Reluctant 비교:**

- `Greedy`: backtracking 알고리즘(back-off)으로 최대 범위 → 최소 범위로 매칭 시도
- `Reluctant`: 최소 범위 → 최대 범위로 확장하며 매칭 시도

예) 매칭 대상 `"abccc"` 일 때:
- Greedy `abc+` → `"abccc"`
- Reluctant `abc+?` → `"abc"`


### Greedy Quantifier 동작 예제

```
Input: AABABB
Pattern (Greedy): A.*B
Match result: 1 match → AABABB

1st try range against '.': A|ABABB|  → 실패 (B 매칭 불가)
2nd try range against '.': A|ABAB|B  → 성공
```

### Reluctant Quantifier 동작 예제

```
Input: AABABB
Pattern (Reluctant): A.*?B
Match result: 2 match → AAB, ABB

1st try: A||BABB  → 실패 (B 불일치)
2nd try: A|A|BABB → 성공 → 매칭 AAB
3rd try: AABA||BB → 성공 → 매칭 ABB
```

### Greedy Quantifier 매칭 방식 (코드)

```
for ( i = 0; i < input_str.length; ++i) {
    for ( j = input_str.length; j > i; --j ) {
        substr = input_str.subString(i, j);
        if ( substr matches with PATTERN) {
            return substr;
        }
    }
}
```

### Reluctant Quantifier 매칭 방식 (코드)

```
for ( i = 0; i < input_str.length; ++i) {
    for ( j = i + 1; j <= input_str.length; ++j ) {
        substr = input_str.subString(i, j);
        if ( substr matches with PATTERN) {
            return substr;
        }
    }
}
```

### Possessive Quantifier

항상 완전체 입력 문자열을 대상으로 단 한 번만 매칭 시도를 하는 수량자.  
Greedy를 이용해 설명하자면, Greedy 탐색을 단 1회만 시도하고 매칭이 되지 않아도 종료.  
**절대로 back off (= giving back)하며 시도하지 않는다.**

> ⚠️ 주의: 1회의 매칭 시도 실패 시 다음 시도는 없다.


---

## Capturing Group

`()` 안의 문자들을 하나의 유닛으로 지정(grouping)한다.

**주요 용도:**
1. 수량자를 문자 그룹에 적용할 때
2. 넘버링과 역참조를 이용하여 매칭 대상에 그룹으로 지정된 문자구획이 반복 사용되는 것을 명시할 때

**넘버링(Numbering):**  
중첩된 그룹은 왼쪽에서 오른쪽으로 `(` 를 세어서 numbering 된다.

```
((A)(B(C)))
  1. ((A)(B(C)))
  2. (A)
  3. (B(C))
  4. (C)
```

> Note 1. group 0는 패턴에 매칭된 전체 expression  
> Note 2. `(?:` 로 시작하는 non-capturing group은 그룹 번호에 집계되지 않음

**역참조(Backreferences):**

예1) 2자리 숫자 뒤에 동일한 2자리 숫자가 오는 것: `(\d\d)\1`
- `"1212"` → 매칭 (group 1 = `12`, `\1` = `12`)
- `"1234"` → 불일치 (group 1 = `12`, `\1 ≠ 34`)


### non-capturing group 예제

```
target:
https://stackoverflow.com/
https://stackoverflow.com/questions/tagged/regex

regex (capturing group):
(https?|ftp)://([^/\r\n]+)(/[^\r\n]*)?

결과:
Match "https://stackoverflow.com/"
  Group 1: "https"
  Group 2: "stackoverflow.com"
  Group 3: "/"

Match "https://stackoverflow.com/questions/tagged/regex"
  Group 1: "https"
  Group 2: "stackoverflow.com"
  Group 3: "/questions/tagged/regex"

regex (non-capturing group):
(?:https?|ftp)://([^/\r\n]+)(/[^\r\n]*)?

결과:
Match "https://stackoverflow.com/"
  Group 1: "stackoverflow.com"
  Group 2: "/"

Match "https://stackoverflow.com/questions/tagged/regex"
  Group 1: "stackoverflow.com"
  Group 2: "/questions/tagged/regex"
```

### atomic group: `(?> expression)`

non-capturing group + possessive matching


---

## 전후방탐색 (lookaround)

> PCRE에서만 유효한지 확인 필요.  
> grep에서 PCRE 옵션(`-Po`)에서만 작동하고 ECRE 옵션(`-Eo`)에서는 적용이 안된다.

획득 대상의 앞뒤에 위치한 prefix와 suffix를 명시하는데 사용하는 표현.  
획득 대상 패턴의 앞뒤에 anchor 패턴을 지정할 때 사용한다.

- **후방탐색(lookbehind)**: prefix를 정의하는 패턴
- **전방탐색(lookahead)**: suffix를 정의하는 패턴

**전방탐색(lookahead)**: `<finding pattern><criterion pattern>` 형태에 매치되는 영역을 탐색하고 `<finding pattern>`에 해당하는 영역만 소비(consume)한다.

**후방탐색(lookbehind)**: `<criterion pattern><finding pattern>` 형태에 매치되는 영역을 탐색하고 `<finding pattern>`에 해당하는 영역만 소비(consume)한다.

> ⚠️ 후방탐색(`(?<[=|!])`) 표현식에서는 변동 길이 수량자(`?`, `*`, `+` 등)을 사용할 수 없고  
> 고정 길이 수량자(`{num}`)만 사용 가능하다.  
> 전방탐색(`(?[=|!])`)에서는 변동 길이 수량자를 사용할 수 있다.

### Pattern 표

| Pattern | Type | Matches |
|---------|------|---------|
| `X(?=Y)` | Positive lookahead | X if followed by Y |
| `X(?!Y)` | Negative lookahead | X if not followed by Y |
| `(?<=Y)X` | Positive lookbehind | X if after Y |
| `(?<!Y)X` | Negative lookbehind | X if not after Y |

### 상세 설명

| Syntax | 탐색 타입 | 설명 |
|--------|-----------|------|
| `<fp>(?=<cp>)` | 긍정형 전방탐색 (positive lookahead) | `<cp>`를 포함하면서 앞쪽의 `<fp>`와 일치하는 텍스트. 매칭 결과에 `<cp>` 미포함 |
| `<fp>(?!<cp>)` | 부정형 전방탐색 (negative lookahead) | `<cp>`를 포함하지 않으면서 `<fp>`와 일치하는 텍스트. 매칭 결과에 `<cp>` 미포함 |
| `(?<=<cp>)<fp>` | 긍정형 후방탐색 (positive lookbehind) | `<cp>` 조건을 충족하면서 뒤쪽의 `<fp>`와 일치하는 텍스트. 매칭 결과에 `<cp>` 미포함 |
| `(?<!<cp>)<fp>` | 부정형 후방탐색 (negative lookbehind) | `<cp>` 조건을 충족하지 않으면서 `<fp>`와 일치하는 텍스트. 매칭 결과에 `<cp>` 미포함 |

### 예제

**전방탐색(?) 긍정형(=)**
```
Regex: \d+(?=\$)
Input: I paid 30$ for 100 apples and paid 10$ for 30 pears.
Match: 30, 10
```

**전방탐색(?) 부정형(!)**
```
Regex: \d+(?!\$)
Input: I paid 30$ for 100 apples and paid 10$ for 30 pears.
Match: 100, 30
```

**후방탐색(?<) 긍정형(=)**
```
Regex: (?<=\$)\d+
Input: I paid $30 for 100 apples and paid $10 for 30 pears.
Match: 30, 10
```

**후방탐색(?<) 부정형(!)**
```
Regex: (?<!\$)\d+
Input: I paid $30 for 100 apples and paid $10 for 30 pears.
Match: 100, 30
```

### 전후방탐색 함께 사용

```
regex syntax:
(?<={prefix-anchor-pattern}){finding pattern}(?={suffix-anchor-pattern})

예) 파일에서 전후방 anchor 사이의 부분만 추출하는 grep 명령어
$ grep -Po '(?<={prefix-anchor-pattern}).+?(?={suffix-anchor-pattern})' <path/to/file>
(-P: --perl-regexp, 주의: -E (--extended-regexp)에서는 전후방탐색 적용 안됨)
(-o: --only-matching)

예) json에서 mykey에 대한 string value를 획득하는 명령어
$ grep -Po '(?<="mykey":").+?(?=")' my.json
```

### lookaround 중복 사용 = AND 연산

```
\d+(?=a.)(?=.b)  ==  \d+(?=a.) && \d+(?=.b)

다음 문자열에 대하여: 100ab

\d+(?=ab)    → Match: 100
\d+(?=a.)    → Match: 100
\d+(?=.b)    → Match: 100
\d+(?=a.)(?=.b)  → Match: 100
\d+(?=b)     → no match
\d+(?=a.)(?=b)   → no match
```

> ⚠️ 주의: 일부 정규 표현식에서 소비한다(consume)는 용어는 "매칭 결과에 포함한다"라는 의미이다.  
> 전후방 탐색에서 `<criterion pattern>`은 매칭 결과에 소비(포함)하지 않는다.
